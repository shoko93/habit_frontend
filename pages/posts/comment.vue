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
              <v-row v-for="(comment, index) in comments" :key="comment.id">
                <v-col col="12">
                  <v-card
                    elevation="2"
                    >
                    <v-card-title>{{ comment.user.name }}</v-card-title>
                    <v-card-text>
                      <div v-if="editIndex == index">
                        <v-row>
                          <v-col cols="12">
                            <v-file-input
                              label="image"
                              accept="image/*"
                              v-model="newImage"
                              prepend-icon="mdi-camera"
                            />
                          </v-col>
                        </v-row>
                        <v-row v-if="comment.image.url">
                          <v-col col="12">
                            <v-img
                              max-width="100%"
                              :src="comment.image.url"
                            ></v-img>
                          </v-col>
                        </v-row>
                        <v-row>
                          <v-col col="12">
                            <v-textarea label="コメント" v-model="newComment"></v-textarea>
                          </v-col>
                        </v-row>
                      </div>
                      <div v-else>
                        <v-row v-if="comment.image.url">
                          <v-col col="12">
                            <v-img
                              max-width="100%"
                              :src="comment.image.url"
                            ></v-img>
                          </v-col>
                        </v-row>
                        <v-row>
                          <v-col col="12">
                            {{ comment.comment }}
                          </v-col>
                        </v-row>
                      </div>
                    </v-card-text>
                    <v-card-actions>
                      <v-row v-if="editIndex == index">
                        <v-col col="12" class="text-right">
                          <v-btn
                            depressed
                            color="primary"
                            @click="updateComment"
                          >
                            送信
                          </v-btn>
                          <v-btn
                            depressed
                            @click="editIndex=null"
                          >
                            キャンセル
                          </v-btn>
                        </v-col>
                      </v-row>
                      <v-row v-else>
                        <v-col col="12" class="text-right">
                          <v-btn icon @click="like(index)"><v-icon :color="iconColor(comment.like)">mdi-thumb-up</v-icon></v-btn>
                          <v-btn icon v-if="item.line_id==currentUserId" @click="editComment(index)"><v-icon>mdi-pencil</v-icon></v-btn>
                          <v-btn icon v-if="item.line_id==currentUserId" @click="deleteCommentConfirm(index)"><v-icon>mdi-delete</v-icon></v-btn>
                        </v-col>
                      </v-row>
                    </v-card-actions>
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
                        <v-col cols="12">
                          <v-file-input
                            label="image"
                            accept="image/*"
                            v-model="addImage"
                            prepend-icon="mdi-camera"
                          />
                        </v-col>
                      </v-row>
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
        @deleteItem="deletePost"
      ></DeleteDialog>
      <DeleteDialog
        :showDialog="showCommentDialog"
        @closeDialog="showCommentDialog=false"
        @deleteItem="deleteComment"
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
        showCommentDialog: false,
        editIndex: null,
        newComment: "",
        newImage: null,
        deleteIndex: "",
        addImage: null,
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
          axios.post(`/posts/${this.$route.query.id}`, {line_id_token: this.$liff.getIDToken()})
            .then((response) => {
              this.item = response.data
              this.$set(this.item, 'created_at', moment(this.item.created_at).format('YYYY-MM-DD H:m'))
            })
            .catch((error) => {
              console.log(error)
            })
          this.getComments();
        })
        .catch((error) => {
          this.liffError = error;
        });
    },
    methods: {
      getComments: function() {
        axios.post(`/posts/${this.$route.query.id}/comments`, {line_id_token: this.$liff.getIDToken()})
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
      like: function(index) {
        console.log(this.comments[index])
        if (this.comments[index].like) {
          let data = {
            line_id_token: this.$liff.getIDToken()
          }
          axios.delete(`/comments/${this.comments[index].id}/like`, {data: data})
            .then((response) => {
              let item = this.comments[index]
              item.like = false
              this.$set(this.comments, index, item)
            })
            .catch((error) => {
              console.log(error)
            })
        } else {
          let data = {
            comment_id: this.comments[index].id,
            line_id_token: this.$liff.getIDToken()
          }
          console.log(data)
          axios.post('/comments/like', {like: data})
            .then((response) => {
              let item = this.comments[index]
              item.like = true
              this.$set(this.comments, index, item)
            })
            .catch((error) => {
              console.log(error)
            })
        }
      },
      editComment: function(index) {
        this.editIndex = index
        this.newComment = this.comments[index].comment
      },
      deleteConfirm: function() {
        this.showDialog = true
      },
      deleteCommentConfirm: function(index) {
        this.showCommentDialog = true
        this.deleteIndex = index
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
      deleteComment: function() {
        axios.delete(`/comments/${this.comments[this.deleteIndex].id}`)
        .then((response) => {
          this.comments.splice(this.deleteIndex, 1)
          this.deleteIndex = ""
          this.showCommentDialog = false
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
            if (this.addImage) {
              const formData = new FormData()
              formData.append('id', response.data.id)
              formData.append('image', this.addImage)
              const config = {
                headders: {
                  'content-type': 'multipart/form-data'
                }
              }
              axios.post('/comments/image', formData, config)
                .then(() => {
                  this.comment = ""
                  this.addImage = null
                  this.getComments()
                })
                .catch((error) => {
                  console.log(error)
                })
            } else {
              this.comment = ""
              this.getComments()
            }
          })
          .catch((error) => {
            console.log(error)
          })
      },
      updateComment: function() {
        let data = {
          comment: this.newComment,
        }
        axios.patch(`/comments/${this.comments[this.editIndex].id}`, {comment: data})
          .then((response) => {
            if (this.newImage) {
              const formData = new FormData()
              formData.append('id', this.comments[this.editIndex].id)
              formData.append('image', this.newImage)
              const config = {
                headders: {
                  'content-type': 'multipart/form-data'
                }
              }
              axios.post('/comments/image', formData, config)
                .then(() => {
                  this.editIndex = null
                  this.newComment = ""
                  this.newImage = null
                  this.getComments()
                })
                .catch((error) => {
                  console.log(error)
                })
            } else {
              let commentItem = this.comments[this.editIndex]
              commentItem.comment = this.newComment
              this.$set(this.comments, this.editIndex, commentItem)
              this.editIndex = null
              this.newComment = ""
            }
          })
          .catch((error) => {
            console.log(error)
          })
      }
    }
  };
  </script>