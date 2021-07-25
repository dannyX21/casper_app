<template>
  <div>
    <div class="container is-fluid">
      <Login v-if="showLoginDialog"></Login>
      <Buyers v-if="feed !== null" @needRefresh="needRefresh" @needLogin="needLogin" @buyers:loaded="buyersLoaded"></Buyers>
      <Feed v-if="!showLoginDialog" @needRefresh="needRefresh" @needLogin="needLogin" @feed:selected="feedSelected" :updateFeeds="updateFeeds"></Feed>
      <b-tabs position="is-centered" class="block">
        <b-tab-item label="Orders">
          <Orders v-if="feed !==null && path === 'home'" :feed="feed" @needRefresh="needRefresh" @needLogin="needLogin"></Orders>
        </b-tab-item>
        <b-tab-item label="Summary">
          <Summary 
            v-if="access !== null && feed !== null && buyers !== null && buyers.length > 0"
            :access="access"
            :refresh="refresh"
            :feed="feed"
            :buyers="buyers"
            @needRefresh="needRefresh"
            @needLogin="needLogin">
          </Summary>
        </b-tab-item>
        <b-tab-item label="Upload">
          <Upload v-if="access !== null" :access="access" :refresh="refresh" @needRefresh="needRefresh" @needLogin="needLogin" @file:updated="fileUpdated"></Upload>
        </b-tab-item>
      </b-tabs>
    </div>
  </div>
</template>

<script>
const axios = require('axios')
import Login from './Login.vue'
import Feed from './Feed.vue'
import Orders from './Orders.vue'
import Buyers from './Buyers.vue'
import Summary from './Summary.vue'
import Upload from './Upload.vue'

export default {
  name: 'Index',
  props: {
    msg: String
  },
  computed: {
    endpoint () {
      return `${process.env.VUE_APP_API_URI}/api/token/refresh/`
    },
    payload () {
      return {
        refresh: this.refresh
      }
    }
  },
  data () {
    return {
      access: null,
      refresh: null,
      showLoginDialog: false,
      feed: null,
      shadow: true,
      fixedTop: true,
      buyers: [],
      path: 'home',
      updateFeeds: false
    }
  },
  components: {
    Login,
    Feed,
    Orders,
    Buyers,
    Summary,
    Upload
  },
  methods: {
    refreshToken () {
      return axios.post(this.endpoint, this.payload).then( res => {
        this.access = res.data.access
        this.refresh= res.data.refresh
        window.$cookies.set('casperAccess', res.data.access, '1h', '/')
        window.$cookies.set('casperRefresh', res.data.refresh, '24h', '/')
        this.showLoginDialog = false
      }).catch(err => {
        if (err.response && err.response.data.detail) {
          this.$buefy.toast.open({
            duration: 5000,
            message: err.response.data.detail,
            position: 'is-bottom',
            type: 'is-warning'
          })
        }
        this.showLoginDialog = true
      })
    },
    needLogin () {
      this.showLoginDialog = true
    },
    needRefresh () {
      let method = null
      let args = []
      if (arguments.length > 0) {
        method = arguments[0]
        args = []
        for(let c=1; c<arguments.length; c++){
          args.push(arguments[c])
        }
        console.log(`method: ${method}, arguments: ${args}`)
      }
      this.refreshToken ().then(() => {
        console.log(`myRefresh: ${this.refresh}`)
      })
    },
    feedSelected (feed) {
      this.feed = feed
      this.updateFeeds = false
    },
    buyersLoaded (buyers) {
      this.buyers = buyers
    },
    fileUpdated () {
      this.updateFeeds = true
    }
  },
  mounted () {
    this.access = window.$cookies.get('casperAccess')
    this.refresh = window.$cookies.get('casperRefresh')
    console.log(`access: ${this.access}, refresh: ${this.refresh}`)
    if (this.access === null && this.refresh !== null) {
      this.refreshToken()
    }
    else if (this.refresh === null) {
      this.showLoginDialog = true
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
