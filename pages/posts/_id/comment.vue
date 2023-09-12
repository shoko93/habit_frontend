<template>
    <div>
      <v-row>
        <v-col col="12">
          <PostItem
            :post="item"
            :currentUserId="currentUserId"
            @deleteConfirm="deleteConfirm"
          ></PostItem>
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
      <DeleteDialog
        :showDialog="showDialog"
        @closeDialog="showDialog=false"
        @deletePost="deletePost"
      ></DeleteDialog>
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
          bookmark: false,
          image: {
            url: ""
          }
        },
        currentUserId: "",
        comments: [],
        comment: "",
        message: "",
        showMessage: false,
        showDialog: false,
      };
    },
    mounted() {
      this.$liffInit
        .then(() => {
          liff
            .getProfile()
            .then((profile) => {
              this.currentUserId = profile.userId
            })
            .catch((err) => {
              console.log("error", err)
            });
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
      deleteConfirm: function() {
        this.showDialog = true
      },
      deletePost: function() {
        axios.delete(`/posts/${this.item.id}`)
        .then((response) => {
          this.$router.push({path: '/'})
        })
        .catch((error) => {
          console.log(error)
        })
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
    }
  };
  </script>