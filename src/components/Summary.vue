<template>
  <div class="section">
    <div class="columns is-multiline">
      <div class="column is-3" v-for="(value, start) in weeks" :key="start">
        <div class="card">
          <header class="vard-header">
            <p class="card-header-title">
              {{ start }} - {{value.endDate}}
            </p>
          </header>
          <div class="card-content">
            <div class="content">
              <table class="table is-bordered is-striped is-narrow">
                <thead>
                  <th>Buyer</th>
                  <th>Qty</th>
                  <th>Ext. Qty</th>
                </thead>
                <tbody>
                  <tr v-for="(v, byr) in value.buyers" :key="byr">
                    <td>{{byr}}</td>
                    <td class="has-text-right">{{v.quantity}}</td>
                    <td class="has-text-right">{{v.extendedQuantity}}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
const axios = require('axios')
export default {
  name: 'Summary',
  props: {
    buyers: {
      type: Array
    },
    feed: {
      type: Object
    },
    access: {
      type: String,
      default: null
    },
    refresh: {
      type: String,
      default: null
    }
  },
  data () {
    return {
      weeks: {}
    }
  },
  computed: {
    endpoint () {
      return `${process.env.VUE_APP_API_URI}/feeds/${this.feed.id}/summary/`
    },
    headers () {
      return {
          'Authorization': `Bearer ${this.access}`
        }
    },
    query () {
      return {}
    },
    config () {
      return {params: this.query, headers: this.headers}
    }
  },
  methods: {
    request (method, payload) {
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
        const records = res.data.results
        this.weeks = {}
        records.forEach(record => {
          if(!Object.prototype.hasOwnProperty.call(this.weeks, record.start_date)) {
            this.weeks[record.start_date] = {
              endDate: record.end_date,
              buyers: {}
            }
            this.buyers.forEach(buyer => {
              this.weeks[record.start_date].buyers[buyer.code] = {
                quantity: 0,
                extendedQuantity: 0
              }
            })
          }
          this.weeks[record.start_date].buyers[record.buyer.code].quantity += record.quantity
          this.weeks[record.start_date].buyers[record.buyer.code].extendedQuantity += record.extended_quantity
        })
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
}
</script>