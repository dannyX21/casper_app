<template>
  <div>
    <b-notification v-if="this.feed !== null" type="is-info is-light" aria-close-label="Close notification" :duration="duration" auto-close>
      {{message}}
    </b-notification>
    <b-notification v-if="this.feeds.length === 0" type="is-danger is-light" aria-close-label="Close notification">
      Feeds not found! Please import orders file.
    </b-notification>
  </div>
</template>
<script>
const axios = require('axios')
const dayjs = require('dayjs')
import advancedFormat from 'dayjs/plugin/advancedFormat'
import dayjsPluginUTC from 'dayjs-plugin-utc'
import relativeTime from 'dayjs/plugin/relativeTime'
dayjs.extend(advancedFormat)
dayjs.extend(dayjsPluginUTC)
dayjs.extend(relativeTime)

export default {
  name: "Feed",
  props: {
    updateFeeds: {
      type: Boolean,
      default: false
    }
  },
  data (){
    return {
      access: null,
      refresh: null,
      feed: null,
      feeds: [],
      duration: 5000
    }
  },
  computed: {
    endpoint () {
      return `${process.env.VUE_APP_API_URI}/feeds/`
    },
    message () {
        return this.feed !== null ?`Updated by ${this.feed.uploaded_by.first_name} ${this.feed.uploaded_by.last_name} on ${dayjs.utc(this.feed.updated_at).local().format('MMM D, YYYY @ h:mm A')}` : null
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
        this.feeds = res.data.results
        console.log(this.feeds)
        if(this.feeds.length > 0) {
          this.feed = this.feeds[0]
          this.$emit('feed:selected', this.feed)
        }
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
  watch: {
    updateFeeds (value) {
      if (value) {
        this.load()
      }
    }
  },
  mounted (){
    this.load()
  }
}
</script>