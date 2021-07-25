<template>
  <div class="section">
    <div class="columns">
      <div class="column">
        <b-collapse class="card" animation="slide" aria-id="tableSettings" :open.sync="filtersOpen">
          <template #trigger="props">
            <div class="card-header" role="button" aria-controls="tableSettings">
              <p class="card-header-title">Filters</p>
              <a class="card-header-icon">
                <b-icon
                  :icon="props.open ? 'chevron-up' : 'chevron-down'">
                </b-icon>
              </a>
            </div>
          </template>
          <div class="card-content">
            <div class="content">
              <div class="columns">
                <div class="column is-2">
                  <b-field label="SO#">
                    <b-input v-model="salesOrderNumber" size="is-small" @input="load"></b-input>
                  </b-field>
                </div>
                <div class="column is-2">
                  <b-field label="PO#">
                    <b-input v-model="purchaseOrderNumber" size="is-small" @input="load"></b-input>
                  </b-field>
                </div>
                <div class="column is-2">
                  <b-field label="P/N">
                    <b-input v-model="itemNumber" size="is-small" @input="load"></b-input>
                  </b-field>
                </div>
                <div class="column is-2">
                  <b-field label="Ship date">
                    <b-datepicker placeholder="Click to select..." v-model="confirmedShipping" range size="is-small" @input="load"></b-datepicker>
                  </b-field>
                </div>
                <div class="column is-2">
                  <b-field label="Buyer">
                    <b-input v-model="buyer" size="is-small" @input="load"></b-input>
                  </b-field>
                </div>
                <div class="column is-2">
                  <b-field label="Planner">
                    <b-input v-model="planner" size="is-small" @input="load"></b-input>
                  </b-field>
                </div>
              </div>
              <div class="columns">
                <div class="column is-2">
                  <b-switch v-model="tableProps.paginated" size="is-small">Paginated</b-switch>
                </div>
                <div class="column is-2">
                  <b-select v-model="tableProps.perPage" :disabled="!tableProps.paginated" size="is-small">
                    <option value="25">25 per page</option>
                    <option value="50">50 per page</option>
                    <option value="100">100 per page</option>
                    <option value="250">250 per page</option>
                  </b-select>
                </div>
              </div>
            </div>
          </div>
        </b-collapse>
      </div>
    </div>
    <div class="columns">
      <div class="column">
        <b-table
          v-if="orders !== null"
          :data="orders"
          :columns="columns"
          :striped="tableProps.striped"
          :narrowed="tableProps.narrowed"
          :hoverable="tableProps.hoverable"
          :sticky-header="tableProps.stickyHeader"
          :height="tableProps.height"
          :sort-icon="tableProps.sortIcon"
          :default-sort="tableProps.defaultSort"
          :default-sort-direction="tableProps.defaultSortDirection"
          :paginated="tableProps.paginated"
          :per-page="tableProps.perPage"
          :current-page.sync="tableProps.currentPage"
          aria-next-label="Next page"
          aria-previous-label="Previous page"
          aria-page-label="Page"
          aria-current-label="Current page"
        ></b-table>
      </div>
    </div>
  </div>
</template>
<script>
const axios = require('axios')
const dayjs = require('dayjs')
import advancedFormat from 'dayjs/plugin/advancedFormat'
import dayjsPluginUTC from 'dayjs-plugin-utc'
import relativeTime from 'dayjs/plugin/relativeTime'
import debounce from 'lodash.debounce'
dayjs.extend(advancedFormat)
dayjs.extend(dayjsPluginUTC)
dayjs.extend(relativeTime)
export default {
  name: "Orders",
  props: {
    feed: {
      type: Object,
      default: null
    }
  },
  data () {
    return {
      access: null,
      refresh: null,
      orders: null,
      columns: [
        {field: 'sales_order_number', label: 'SO#', centered: true, sortable: true},
        {field: 'item_number', label: 'P/N', centered: true, sortable: true},
        {field: 'revision', label: 'Revision', centered: true},
        {field: 'quantity', label: 'Qty', numeric: true},
        {field: 'extended_quantity', label: 'Ext. Qty', numeric: true},
        {field: 'unit', label: 'UM', centered: true},
        {field: 'confirmed_shipping', label: 'Ship Date', sortable: true},
        {field: 'purchase_order_number', label: 'PO#', centered: true},
        {field: 'buyer.code', label: 'Buyer', centered: true, sortable: true},
        {field: 'planner.code', label: 'Planner', centered: true, sortable: true},
        {field: 'note', label: 'Note'}
      ],
      tableProps: {
        bordered: true,
        striped: true,
        narrowed: true,
        hoverable: true,
        stickyHeader: true,
        sortIcon: 'chevron-up',
        height: '800px',
        defaultSort: 'confirmed_shipping',
        defaultSortDirection: 'asc',
        paginated: false,
        perPage: 25,
        currentPage: 1
      },
      salesOrderNumber: null,
      purchaseOrderNumber: null,
      itemNumber: null,
      buyer: null,
      planner: null,
      confirmedShipping: [],
      filtersOpen: false
    }
  },
  computed: {
    endpoint () {
      return `${process.env.VUE_APP_API_URI}/feeds/${this.feed.id}/orders/`
    },
    headers () {
      return {
          'Authorization': `Bearer ${this.access}`
        }
    },
    query () {
      const f = { pagination: 0 }
      if (this.salesOrderNumber !== null) {
        f.sales_order_number = this.salesOrderNumber
      }
      if (this.purchaseOrderNumber !== null) {
        f.purchase_order_number = this.purchaseOrderNumber
      }
      if (this.itemNumber !== null) {
        f.item_number = this.itemNumber
      }
      if (this.confirmedShipping.length > 0) {
        f.confirmed_shipping_gte = dayjs(this.confirmedShipping[0]).unix()
        f.confirmed_shipping_lt = dayjs(this.confirmedShipping[1]).unix()
      }
      if (this.buyer !== null) {
        f.buyer = this.buyer
      }
      if (this.planner !== null) {
        f.planner = this.planner
      }
      return f
    },
    config () {
      return {params: this.query, headers: this.headers}
    }
  },
  methods: {
    request (method, payload) {
      this.access = window.$cookies.get('casperAccess')
      this.refresh = window.$cookies.get('casperRefresh')
      if (this.access === null && this.refresh !== null) {
        this.$emit('needRefresh', this.request, method, payload)
      }
      else if (this.refresh === null) {
        this.$emit('needLogin')
      }
      else {
        switch(method) {
          case 'get':
            return axios[method](this.endpoint, this.config)
          default:
            return axios[method](this.endpoint, payload || null, this.config)
        }
      }
      return null
    },
    load () {
      const prom = this.request('get')
      if (prom === null) {
        return
      }
      prom.then(res => {
        console.log(`count: ${res.data.results.length}`)
        this.orders = res.data.results
      }).catch(err => {
        if (err.response && err.response.data.detail) {
          this.$buefy.toast.open({
            duration: 5000,
            message: err.response.data.detail,
            position: 'is-bottom',
            type: 'is-danger'
          })
        }
      })
    }
  },
  mounted (){
    this.load()
  },
  created () {
    this.load = debounce(this.load, 1000)
  }
}
</script>