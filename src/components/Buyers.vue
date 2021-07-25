<template>
  <div>
  </div>
</template>
<script>
const axios = require('axios')

export default {
  name: "Buyers",
  data (){
    return {
      access: null,
      refresh: null,
      buyers: []
    }
  },
  computed: {
    endpoint () {
      return `${process.env.VUE_APP_API_URI}/buyers/`
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
        const headers = {
          'Authorization': `Bearer ${this.access}`
        }
        switch(method) {
          case 'get':
            return axios[method](this.endpoint, {headers: headers})
          default:
            return axios[method](this.endpoint, payload || null, {headers: headers})
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
        this.buyers = res.data.results
        this.$emit('buyers:loaded', this.buyers)
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
  }
}
</script>