<template>
    <div>
      <v-row>
        <v-col col="12">
          <v-card
            elevation="2"
          >
            <v-card-title>{{ item.title }}</v-card-title>
            <v-card-text>
              {{ item.created_at}}<br>
              {{ item.text_body }}
            </v-card-text>
            <v-card-actions>
              <v-btn icon @click="like(index)"><v-icon :color="iconColor(item.like)">mdi-thumb-up</v-icon></v-btn>
              <v-btn icon @click="bookmark(index)"><v-icon :color="iconColor(item.bookmark)">mdi-heart</v-icon></v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
      <v-row>
        <v-col col="12">
          <v-card
            elevation="2"
          >
            <v-card-title>コメント</v-card-title>
            <v-card-text>
              <v-row v-for="comment in comments" :key="comment.id">
                <v-col col="12">
                  <v-card
                    elevation="2"
                    >
                    <v-card-title>{{ comment.user.name }}</v-card-title>
                    <v-card-text>
                      {{ comment.comment }} 
                    </v-card-text>
                  </v-card>
                </v-col>
              </v-row>
              <v-row>
                <v-col col="12">
                  <v-card
                    elevation="2"
                    >
                    <v-card-text>
                      <v-row>
                        <v-col col="12">
                          <v-textarea label="コメント" v-model="comment"></v-textarea>
                        </v-col>
                      </v-row>
                    </v-card-text>
                    <v-card-actions>
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
                    </v-card-actions>
                  </v-card>
                </v-col>
              </v-row>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </div>
  </template>
  <script>
  import axios from '~/plugins/axios'
  import moment from 'moment'
  export default {
    data: function() {
      return {
        item: {
          title: "",
          created_at: "",
          text_body: "",
          like: false,
          bookmark: false
        },
        comments: [],
        comment: "",
        message: "",
        showMessage: false,
      };
    },
    async mounted() {
      this.$liffInit
        .then(() => {
          axios.post(`/posts/${this.$route.params.id}`, {line_id_token: this.$liff.getIDToken()})
            .then((response) => {
              this.item = response.data
              this.$set(this.item, 'created_at', moment(this.item.created_at).format('YYYY-MM-DD H:m'))
            })
            .catch((error) => {
              console.log(error)
            })
        })
        .catch((error) => {
          this.liffError = error;
        });
      this.getComments();
    },
    methods: {
      getComments: function() {
        axios.get(`/posts/${this.$route.params.id}/comments`)
        .then((response) => {
          this.comments = response.data
        })
        .catch((error) => {
          console.log(error)
        })
      },
      iconColor: function(like) {
        return like ? "blue" : ""
      },
      submit: function() {
        let data = {
          post_id: this.item.id,
          comment: this.comment,
          line_id_token: this.$liff.getIDToken()
        }
        axios.post('/posts/comment', {comment: data})
          .then((response) => {
            // this.message = "登録完了しました"
            // this.showMessage = true
            this.getComments()
          })
          .catch((error) => {
            // this.message = "登録に失敗しました"
            // this.showMessage = true
          })
      },
      like: function(index) {
        let data = {
          post_id: this.item.id,
          line_id_token: this.$liff.getIDToken()
        }
        axios.post('/posts/like', {like: data})
          .then((response) => {
            let item = this.data[index]
            item.like = true
            this.$set(this.data, index, item)
          })
          .catch((error) => {
            console.log(error)
          })
      },
      bookmark: function(index) {
        let data = {
          post_id: this.item.id,
          line_id_token: this.$liff.getIDToken()
        }
        axios.post('/posts/bookmark', {bookmark: data})
          .then((response) => {
            let item = this.data[index]
            item.bookmark = true
            this.$set(this.data, index, item)
          })
          .catch((error) => {
            console.log(error)
          })
      }
    }
  };
  </script>