<template>
  <section>
    <div class="columns">
      <div class="column is-one-third is-offset-one-third">
        <b-field class="file">
          <b-upload v-model="file" native drag-drop @input="uploadFile">
            <section class="section">
              <div class="content has-text-centered">
                <p>
                  <b-icon
                    icon="upload"
                    size="is-large">
                  </b-icon>
                </p>
                <p>Drop your files here or click to upload</p>
              </div>
            </section>
          </b-upload>
        </b-field>
      </div>
    </div>
    <div class="tags">
      <span v-for="(file, index) in dropFiles" :key="index" class="tag is-primary">
        {{file.name}}
        <button class="delete is-small" type="button" @click="deleteDropFile(index)"></button>
      </span>
    </div>
  </section>
</template>
<script>
const axios = require('axios')
export default {
  name: 'Upload',
  props: {
    access: {
      type: String,
      default: null
    },
    refresh: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      file: {},
      dropFiles: []
    }
  },
  computed: {
    endpoint () {
      return `${process.env.VUE_APP_API_URI}/feeds/upload/`
    },
    headers () {
      return {
          'Authorization': `Bearer ${this.access}`,
          'Content-Type': 'multipart/form-data'
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
    deleteDropFile(index) {
      this.dropFiles.splice(index, 1);
    },
    uploadFile () {
      if (this.access === null && this.refresh !== null) {
        this.$emit('needRefresh')
      }
      else if (this.refresh === null) {
        this.$emit('needLogin')
      } else {
        const formData = new FormData()
        formData.append('file', this.file)
        axios.post(this.endpoint, formData, this.config)
          .then(res => {
            this.$buefy.toast.open({
              duration: 5000,
              message: `${res.data.filename} uploaded successfully!`,
              position: 'is-bottom',
              type: 'is-success'
            })
            this.$emit('file:updated')
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
    }
  }
};
</script>