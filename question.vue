<template lang="pug">
div
  PageHeader(:title="title", :items="items")
  .row
    .col-12
      .card
        .card-body
          .row.mb-2
            .col-sm-6
            .col-sm-6
              .float-sm-right
                button.btn.btn-success.mb-2.mb-sm-0(type="button")
                  i.mdi.mdi-cog
          .row.mb-2
            .col-sm-12.col-md-6
              div(id="tickets-table_length", class="dataTables_length")
                label.d-inline-flex.align-items-center
                  | Display&nbsp;
                  b-form-select(
                    v-model="perPage",
                    size="sm",
                    :options="pageOptions"
                  )
                  | &nbsp;Orders : {{currentPage}}
            .col-sm-12.col-md-6
              div(id="tickets-table_filter", class="dataTables_filter text-md-right")
                label.d-inline-flex.align-items-center
                  | Search:
                  b-form-input.form-control.form-control-sm.ml-2(
                    v-model="filter",
                    type="search",
                    placeholder="請輸入電話號碼"
                  )
          .table-responsive.mb-0
            b-table(
              table-class="table table-centered w-100",
              thead-tr-class="bg-light",
              :items="orders",
              :fields="fields",
              responsive="sm",
              :per-page="perPage",
              :current-page="currentPage",
              ref="table"
            )
              template(v-slot:cell(check)="data")
                .custom-control.custom-checkbox.text-center
                  input.custom-control-input(
                    type="checkbox",
                    :id="`contacusercheck${data.item._id}`"
                  )
                  label.custom-control-label(:for="`contacusercheck${data.item._id}`")
              template(v-slot:cell(id)="data")
                nuxt-link.text-body.font-weight-medium(to="/ecommerce/order-detail")
                | {{ data.item.orderNo }}
              template(v-slot:cell(recipient)="data")
                  | {{ data.item.recipient.name }}
              template(v-slot:cell(date)="data")
                | {{ dateFormat(data.item.createdAt) }}
                small.text-muted {{data.item.time}}
              template(v-slot:cell(mobile)="data")
                | {{ data.item.recipient.mobile }}
              template(v-slot:cell(payment)="data")
                | {{ data.item.pay }}
              template(v-slot:cell(paymentstatus)="data")
                span.badge(
                  :class=`
                  {
                    'badge-danger': data.item.payStatus === '未付款',
                    'badge-success': data.item.payStatus === '已付款'
                  }`
                )
                  | {{ data.item.payStatus }}
              template(v-slot:cell(action))
                ul.list-inline.table-action.m-0
                  li.list-inline-item
                    a.action-icon(href="javascript:void(0);")
                      i.mdi.mdi-eye
                  li.list-inline-item
                    a.action-icon(href="javascript:void(0);")
                      i.mdi.mdi-square-edit-outline
                  li.list-inline-item
                    a.action-icon(href="javascript:void(0);")
                      i.mdi.mdi-delete
          .row
            .col
              div(class="dataTables_paginate paging_simple_numbers float-right")
                ul.pagination.pagination-rounded
                  b-pagination(
                    v-model="currentPage",
                    :total-rows="rows",
                    :per-page="perPage",
                  )

</template>
<script>
import to from 'await-to-js';
import moment from 'moment';

/**
 * Orders component
 */
export default {
    head() {
        return {
            title: `${this.title} | 全部訂單`,
        };
    },
    data() {
        return {
            orders: null,
            title: "訂單管理頁面",
            items: [{
                    text: "訂單管理"
                },
                {
                    text: "全部訂單"
                },
            ],
            totalRows: 1,
            currentPage: 1,
            perPage: 10,
            pageOptions: [10, 25, 50, 100],
            rows: null,
            filter: null,
            filterOn: [],
            sortBy: "age",
            sortDesc: false,
            fields: [{
                  key: "check",
                  label: ""
                },
                {
                  key: "id",
                  label: "訂單編號"
                },
                {
                  key: "date",
                  label: "購買日期"
                },
                {
                  key: "recipient",
                  label: "收件人"
                },
                {
                  key: "mobile",
                  label: "訂購人手機"
                },
                {
                  key: "payment",
                  label: "付款方式"
                },
                {
                  key: "paymentstatus",
                  label: "付款狀態"
                },
                {
                  key: "orderstatus",
                  label: "訂單狀態"
                },
                {
                  key: "orderstatus",
                  label: "訂單狀態"
                },
            ]
        };
    },
    watch: {
      currentPage (val) {
        this.fetchOrders(val, this.perPage);
        this.$refs.table.$forceUpdate()
      },
    },
    mounted() {
        this.fetchOrders(this.currentPage, this.perPage);
        // Set the initial number of items
        this.totalRows = this.items.length;
    },
    methods: {
        async fetchOrders (page, limit) {
          page = this.currentPage;
          limit = this.perPage;
          let [ordersErr, orders] = await to(this.$api.order.listOrders({page, limit}));
          if (ordersErr) {
            console.log(ordersErr);
          }
          this.orders = orders.data.docs;
          this.rows = orders.data.totalDocs;
          this.perPage = orders.data.limit;
          this.totalRows = orders.data.totalPages;
        },
        dateFormat(date) {
          return moment(`${date}`).format('YYYY-MM-DD HH:mm');
        }
    },
    middleware: 'router-auth',
};
</script>