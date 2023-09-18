<template>
    <div>
      <v-snackbar
        v-model="showMessage"
        :top="true"
      >
      {{ message }}
      <template v-slot:action="{ attrs }">
        <v-btn
          color="blue"
          text
          v-bind="attrs"
          @click="showMessage = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
      <v-row>
        <v-col cols="12">
          <v-file-input
            label="image"
            accept="image/*"
            v-model="addImage"
            prepend-icon="mdi-camera"
          />
        </v-col>
      </v-row>
      <v-row v-if="item.image.url">
        <v-col col="12">
          <v-img
            max-width="100%"
            :src="item.image.url"
          ></v-img>
        </v-col>
      </v-row>
      <v-row>
        <v-col col="12">
          <v-text-field label="タイトル" v-model="item.title"></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col col="12">
          <v-textarea label="本文" v-model="item.text_body"></v-textarea>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <v-combobox
            v-model="select"
            :items="items"
            label="タグ"
            multiple
          ></v-combobox>
        </v-col>
      </v-row>
      <v-row>
        <v-col col="12" class="text-right">
          <v-btn
            depressed
            color="primary"
            @click="submit"
          >
            送信
          </v-btn>
        </v-col>
      </v-row>
    </div>
  </template>
  <script>
  import axios from '~/plugins/axios'
  export default {
    data: function() {
      return {
        item: {
          title: "",
          created_at: "",
          text_body: "",
          like: false,
          bookmark: false,
          image: {
            url: ""
          }
        },
        sdkVersion: "",
        liffError: "",
        message: "",
        showMessage: false,
        select: [],
        items: [],
        addImage: null,
      };
    },
    async mounted() {
      this.$liffInit
        .then(() => {
            axios.post(`/posts/${this.$route.query.id}`, {line_id_token: this.$liff.getIDToken()})
              .then((response) => {
                this.item = response.data
                response.data.tags.forEach((element) => {
                  this.select.push(element.tag)
                })
              })
              .catch((error) => {
                console.log(error)
              })
        })
        .catch((error) => {
          this.liffError = error;
        });
      axios.get('/tags')
        .then((response) => {
          response.data.forEach((element) => {
            this.items.push(element.tag)
          })
        })
        .catch((error) => {
          console.log(error)
        })
    },
    methods: {
      submit: function() {
        let data = {
          title: this.item.title,
          text_body: this.item.text_body,
          tags: this.select,
          line_id_token: this.$liff.getIDToken()
        }
        axios.patch(`/posts/${this.$route.query.id}`, {post: data})
          .then((response) => {
            if (this.addImage) {
              const formData = new FormData()
              formData.append('id', this.$route.query.id)
              formData.append('image', this.addImage)
              const config = {
                headders: {
                  'content-type': 'multipart/form-data'
                }
              }
              axios.post('/posts/image', formData, config)
                .then(() => {
                  this.message = "更新完了しました"
                  this.showMessage = true
                })
                .catch(() => {
                  this.message = "更新に失敗しました"
                  this.showMessage = true
                })
            } else {
              this.message = "更新完了しました"
              this.showMessage = true
            }
          })
          .catch((error) => {
            this.message = "更新に失敗しました"
            this.showMessage = true
          })
      }
    }
  };
  </script>