<template>
    <div>
        <div class="page-heading">
            <div class="page-title">
                <div class="row">
                    <div class="col-12 col-md-6 order-md-1 order-last">
                        <h3>Order List</h3>
                    </div>
                    <div class="col-12 col-md-6 order-md-2 order-first">
                        <nav aria-label="breadcrumb" class="breadcrumb-header float-start float-lg-end">
                            <ol class="breadcrumb">
                                <li class="breadcrumb-item"><router-link to="/seller/dashboard">{{ __('dashboard') }}</router-link></li>
                                <li class="breadcrumb-item active" aria-current="page">Order List</li>
                            </ol>
                        </nav>
                    </div>
                </div>
            </div>
            <section class="section">
                <div class="card">
                    <div class="card-header">
                        <h4 class="card-title">Latest Orders</h4>
                    </div>
                    <div class="card-body">
                        <div class="row">
                            <b-col md="3">
                                <h6 class="box-title">From & To Date</h6>
                                <div class="d-flex justify-content-center align-items-center">
                                    <date-range-picker
                                        :autoApply=false
                                        :showDropdowns = true
                                        v-model="dateRange"
                                        :maxDate="maxDate"
                                        @update="getOrders"
                                    ></date-range-picker>
                                    <button class="btn btn-sm btn-danger ml-1" @click="dateRange.startDate = null, dateRange.endDate = null, getOrders()">
                                        {{ __('clear') }}
                                    </button>
                                </div>
                            </b-col>

                            <b-col md="3">
                                <h6 class="box-title" for="status">Status </h6>
                                <select id="status" name="status" v-model="status" @change="getOrders()" class="form-control form-select">
                                    <option value="">All Orders</option>
                                    <option v-for="status in statuses" :value='status.id'>{{ status.status }}</option>
                                </select>
                            </b-col>
                            <b-col md="3" class="offset-md-2">
                                <h6 class="box-title">{{ __('search') }}</h6>
                                <b-form-input
                                    id="filter-input"
                                    v-model="filter"
                                    type="search"
                                    :placeholder="__('search')"
                                ></b-form-input>
                            </b-col>
                            <b-col  md="1" class="text-center">
                                <button class="btn btn-primary btn_refresh" v-b-tooltip.hover :title="__('refresh')" @click="getOrders()">
                                    <i class="fa fa-refresh" aria-hidden="true"></i>
                                </button>
                            </b-col>

                        </div>
                        <b-tabs pills active-nav-item-class="font-weight-bold text-uppercase">
                            <b-tab title="Orders" active @click="getOrders">
                                <div class="table-responsive mt-3">
                                    <b-table
                                        :items="orders"
                                        :fields="orderFields"
                                        :current-page="currentPage"
                                        :per-page="perPage"
                                        :filter="filter"
                                        :filter-included-fields="filterOn"
                                        :sort-by.sync="sortBy"
                                        :sort-desc.sync="sortDesc"
                                        :sort-direction="sortDirection"
                                        :bordered="true"
                                        :busy="isLoading"
                                        stacked="md"
                                        show-empty
                                        small>

                                        <template #table-busy>
                                            <div class="text-center text-black my-2">
                                                <b-spinner class="align-middle"></b-spinner>
                                                <strong>{{ __('loading') }}...</strong>
                                            </div>
                                        </template>

                                        <template #cell(mobile)="row">
                                            {{ row.item.mobile | mobileMask }}
                                        </template>

                                        <template #cell(actions)="row">
                                            <router-link :to="{ name: 'SellerViewOrder',params: { id: row.item.id, record : row.item }}" v-b-tooltip.hover title="View" class="btn btn-primary btn-sm"><i class="fa fa-eye"></i></router-link>
                                        </template>
                                    </b-table>
                                </div>
                                <b-row>
                                    <div class="col-md-4 text-success h6">Total Amount :- {{ $currency }} {{ total_amount }}</div>
                                    <div class="col-md-4 text-success h6">Total D.Chrg:- {{ $currency }} {{ delivery_charge }}</div>
                                    <div class="col-md-4 text-success h6">Total Final Amount:- {{ $currency }} {{ final_total }}</div>
                                </b-row>
                                <b-row>
                                    <b-col  md="2" class="my-1">
                                        <b-form-group
                                            :label="__('per_page')"
                                            label-for="per-page-select"
                                            label-align-sm="right"
                                            label-size="sm"
                                            class="mb-0">
                                            <b-form-select
                                                id="per-page-select"
                                                v-model="perPage"
                                                :options="pageOptions"
                                                size="sm"
                                                class="form-control form-select"
                                            ></b-form-select>
                                        </b-form-group>
                                    </b-col>
                                    <b-col  md="4" class="my-1" offset-md="6">
                                        <b-pagination
                                            v-model="currentPage"
                                            :total-rows="totalOrderRows"
                                            :per-page="perPage"
                                            align="fill"
                                            size="sm"
                                            class="my-0"
                                        ></b-pagination>
                                    </b-col>
                                </b-row>
                            </b-tab>
                            <b-tab title="Order Items" @click="getOrders">
                                <div class="table-responsive mt-3">
                                    <b-table
                                        :items="order_items"
                                        :fields="orderItemFields"
                                        :current-page="itemCurrentPage"
                                        :per-page="itemPerPage"
                                        :filter="filter"
                                        :filter-included-fields="filterOn"
                                        :sort-by.sync="sortBy"
                                        :sort-desc.sync="sortDesc"
                                        :sort-direction="sortDirection"
                                        :bordered="true"
                                        :busy="isLoading"
                                        stacked="md"
                                        show-empty
                                        small>
                                        <template #table-busy>
                                            <div class="text-center text-black my-2">
                                                <b-spinner class="align-middle"></b-spinner>
                                                <strong>{{ __('loading') }}...</strong>
                                            </div>
                                        </template>

                                        <template #cell(mobile)="row">
                                            {{ row.item.mobile | mobileMask }}
                                        </template>

                                        <template #head(tax)="row">
                                            {{'Tax ('+$currency+') (%)' }}
                                        </template>

                                        <template #cell(is_credited)="row">
                                            <span v-if="row.item.is_credited" class="badge bg-success">Credited</span>
                                            <span v-else class="badge bg-danger"> Not Credited</span>
                                        </template>

                                        <template #cell(active_status)="row">
                                            <span v-if="row.item.active_status == $received" class="badge bg-primary">{{ row.item.active_status }}</span>
                                            <span v-else-if="row.item.active_status == $pending" class="badge bg-secondary">Awaiting Payment</span>
                                            <span v-else-if="row.item.active_status == $processed" class="badge bg-info">{{ row.item.active_status }}</span>
                                            <span v-else-if="row.item.active_status == $shipped" class="badge bg-warning">{{ row.item.active_status }}</span>
                                            <span v-else-if="row.item.active_status == $delivered" class="badge bg-success">{{ row.item.active_status }}</span>
                                            <span v-else-if="row.item.active_status == $returned || row.item.active_status == $cancelled" class="badge bg-danger">{{ row.item.active_status }}</span>
                                            <span v-else class="badge bg-danger">Not Found</span>
                                        </template>
                                        <template #cell(delivery_time)="row">
                                            {{ new Date(row.item.delivery_time).toLocaleString()  }}
                                        </template>
                                        <template #cell(actions)="row">
                                            <router-link :to="{ name: 'SellerViewOrder',params: { id: row.item.order_id, record : row.item }}" v-b-tooltip.hover title="View" class="btn btn-primary btn-sm"><i class="fa fa-eye"></i></router-link>
                                        </template>
                                    </b-table>
                                </div>
                                <b-row>
                                    <div class="col-md-12 text-success text-center h6">Total:- {{ $currency }} {{ order_items_total_sum }}</div>
                                </b-row>
                                <b-row>
                                    <b-col  md="2" class="my-1">
                                        <b-form-group
                                            :label="__('per_page')"
                                            label-for="per-page-select"
                                            label-align-sm="right"
                                            label-size="sm"
                                            class="mb-0">
                                            <b-form-select
                                                id="per-page-select"
                                                v-model="perPage"
                                                :options="pageOptions"
                                                size="sm"
                                                class="form-control form-select"
                                            ></b-form-select>
                                        </b-form-group>
                                    </b-col>
                                    <b-col  md="4" class="my-1" offset-md="6">
                                        <b-pagination
                                            v-model="itemCurrentPage"
                                            :total-rows="totalOrderItemRows"
                                            :per-page="itemPerPage"
                                            align="fill"
                                            size="sm"
                                            class="my-0"
                                        ></b-pagination>
                                    </b-col>
                                </b-row>
                            </b-tab>
                        </b-tabs>
                    </div>
                </div>
            </section>
        </div>
    </div>
</template>
<script>
import DateRangePicker from 'vue2-daterange-picker'
import moment from "moment";
import axios from "axios";
export default {
    name: "range_dates",
    components: {DateRangePicker},
    data: function() {
        return {
            dateRange: {startDate:null, endDate:null},
            maxDate : new Date(),
            status:"",
            orderFields: [
                { key: 'id', label: 'O.ID', sortable: true, sortDirection: 'desc' },
                { key: 'user_name', label: 'User', sortable: true, class: 'text-center' },
                { key: 'mobile', label: 'Mob.', sortable: true, class: 'text-center' },
                { key: 'total', label: 'Total('+ this.$currency +')', sortable: true, class: 'text-center' },
                { key: 'delivery_charge', label: 'D.Chrg', sortable: true, class: 'text-center' },

                /*{ key: 'tax', label: 'Tax '+ this.$currency +'(%)', sortable: true, class: 'text-center' },
                { key: 'discount', label: 'Disc.'+ this.$currency +'(%)', sortable: true, class: 'text-center' },
                { key: 'promo_discount', label: 'Promo Disc.('+ this.$currency +')', sortable: true, class: 'text-center' },
                { key: 'wallet_balance', label: 'Wallet Used('+ this.$currency +')', sortable: true, class: 'text-center' },*/

                { key: 'final_total', label: 'F.Total('+ this.$currency +')', sortable: true, class: 'text-center' },
                { key: 'payment_method', label: 'P.Method', sortable: true, class: 'text-center' },
                { key: 'delivery_time', label: 'D.Time', sortable: true, class: 'text-center' },
                { key: "actions", label: "Actions" }
            ],
            totalOrderRows:1,
            orderItemFields: [
                { key: 'order_id', label: 'O.ID', sortable: true, sortDirection: 'desc' },
                { key: 'id', label: 'O.Item ID', sortable: true, sortDirection: 'desc' },
                { key: 'is_credited', label: 'Commission', sortable: true, class: 'text-center' },
                { key: 'user_name', label: 'U.Name', sortable: true, class: 'text-center' },
                { key: 'product_name', label: 'Product', sortable: true, class: 'text-center' },
                { key: 'mobile', label: 'Mob.', sortable: true, class: 'text-center' },
                { key: 'total', label: 'Total('+ this.$currency +')', sortable: true, class: 'text-center' },

                /*{ key: 'tax', label: 'Tax '+ this.$currency +'(%)', sortable: true, class: 'text-center' },*/

                { key: 'payment_method', label: 'P.Method', sortable: true, class: 'text-center' },
                { key: 'delivery_time', label: 'D.Time', sortable: true, class: 'text-center' },
                { key: 'active_status', label: 'A.Status', sortable: true, class: 'text-center' },
                { key: "actions", label: "Actions" }
            ],
            totalOrderItemRows: 1,
            currentPage: 1,
            perPage: this.$perPage,
            pageOptions: this.$pageOptions,
            sortBy: '',
            sortDesc: false,
            sortDirection: 'asc',
            filter: null,
            filterOn: [],
            page: 1,

            isLoading: false,
            sectionStyle : 'style_1',
            max_visible_units : 12,
            max_col_in_single_row : 3,
            statuses:[],

            orders: [],
            total_amount:0,
            delivery_charge:0,
            final_total:0,

            order_items:[],
            order_items_total_sum:0,

            itemCurrentPage:1,
            itemPerPage:5,

        }
    },
    computed: {
        sortOptions() {
            // Create an options list from our fields
            return this.orderFields
                .filter(f => f.sortable)
                .map(f => {
                    return { text: f.label, value: f.key }
                })
        }
    },
    mounted() {
        // Set the initial number of items
        this.totalOrderRows = this.orders.length
        this.totalOrderItemRows = this.order_items.length
    },
    created: function() {
        this.getOrderStatus();
        this.getOrders();
    },
    methods: {
        getOrderStatus: function () {
            let vm = this;
            axios.get(this.$apiUrl + '/order_statuses').then((response) => {
                this.isLoading = false
                this.statuses = response.data.data;
            }).catch(error => {
                vm.isLoading = false;
                if (error.request.statusText) {
                    this.showError(error.request.statusText);
                }else if (error.message) {
                    this.showError(error.message);
                } else {
                    this.showError("Something went wrong!");
                }
            });
        },
        getOrders(){
            this.isLoading = true
            let param = {
                "startDate": (this.dateRange.startDate != null) ? moment(this.dateRange.startDate).format('YYYY-MM-DD'):"",
                "endDate": (this.dateRange.endDate != null) ? moment(this.dateRange.endDate).format('YYYY-MM-DD'):"",
                "status":this.status,
            }
            axios.get(this.$sellerApiUrl + '/orders',{
                params: param
            }).then((response) => {
                this.isLoading = false
                this.orders = response.data.data.orders;
                this.totalOrderRows = this.orders.length;

                this.total_amount = this.orders.map(item => item.total).reduce((prev, curr) => prev + curr, 0).toFixed(2);
                this.delivery_charge = this.orders.map(item => item.delivery_charge).reduce((prev, curr) => prev + curr, 0).toFixed(2);
                this.final_total = this.orders.map(item => item.final_total).reduce((prev, curr) => prev + curr, 0).toFixed(2);

                this.order_items = response.data.data.order_items;
                this.totalOrderItemRows = this.order_items.length;
                this.order_items_total_sum = this.order_items.map(item => item.total).reduce((prev, curr) => prev + curr, 0).toFixed(2);

            });
        }
    }
};
</script>

<style scoped>
@import "../../../../node_modules/vue2-daterange-picker/dist/vue2-daterange-picker.css";
.vue-daterange-picker[data-v-1ebd09d2] {
    min-width: 80%;
}
@media only screen and (min-width: 600px) {
    .vue-daterange-picker[data-v-1ebd09d2] {
        min-width: 90%;
    }
}
</style>
