<template>
  <section>
    <b-modal
      has-modal-card
      scroll="keep"
      :active.sync="showModal"
      :can-cancel="false"
    >
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title">Login</p>
      </header>
      <section class="modal-card-body">
        <b-field label="Email">
          <b-input v-model="email" type="email" placeholder="Please enter your email"></b-input>
        </b-field>
        <b-field label="Password">
          <b-input v-model="password" type="password" password-reveal></b-input>
        </b-field>
      </section>
      <footer class="modal-card-foot space-between">
        <div class="buttons">
          <b-button type="is-primary" outlined @click="login">Login</b-button>
          <b-button type="is-danger" outlined @click="close">Cancel</b-button>
        </div>
      </footer>
    </div>
    </b-modal>
  </section>
</template>

<script>
  const axios = require('axios')
  export default {
    name: 'Login',
    components: {
    },
    computed: {
      endpoint () {
        return `${process.env.VUE_APP_API_URI}/api/token/`
      },
      payload () {
        return { 
          email: this.email,
          password: this.password
        }
      }
    },
    data() {
      return {
        email: null,
        password: null,
        message: null,
        showModal: true,
      }
    },
    methods: {
      close() {
        this.showModal = false
      },
      reset () {
        this.email = null
        this.password = null
        window.$cookies.remove('casperAccess')
        window.$cookies.remove('casperRefresh')
      },
      login () {
        axios.post(this.endpoint, this.payload).then(res => {
          window.$cookies.set('casperAccess', res.data.access, '1h', '/')
          window.$cookies.set('casperRefresh', res.data.refresh, '24h', '/')
          this.close()
        }).catch(err => {
          console.log(err)
          let errorMessage = 'An unexpected error occured!'
          if (err.response && err.response.data.detail) {
            errorMessage = err.response.data.detail
          }
          this.$buefy.toast.open({
            duration: 5000,
            message: errorMessage,
            position: 'is-bottom',
            type: 'is-danger'
          })
          this.reset()
        })
      }
    }
  }
</script>