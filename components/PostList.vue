<template>
  <div>
    <v-row v-for="(item, index) in data" :key="item.id">
      <v-col col="12">
        <v-card
          elevation="2"
        >
          <v-card-title>{{ item.title }}</v-card-title>
          <v-card-text>
            <v-img
              v-if="item.image.url"
              max-height="150"
              max-width="250"
              :src="item.image.url"
            ></v-img>
            {{ item.created_at}}<br>
            {{ item.text_body }}
            <v-row class="mt-2">
              <v-col col="12">
                <v-btn text v-for="tag in item.tags" :key="tag.id" @click="tagSearch(tag.tag)">{{ tag.tag }}</v-btn>
              </v-col>
            </v-row>
          </v-card-text>
          <v-card-actions>
            <v-btn icon @click="like(index)"><v-icon :color="iconColor(item.like)">mdi-thumb-up</v-icon></v-btn>
            <v-btn icon :to="'/posts/' + item.id + '/comment'"><v-icon>mdi-comment-text</v-icon></v-btn>
            <v-btn icon @click="bookmark(index)"><v-icon :color="iconColor(item.bookmark)">mdi-heart</v-icon></v-btn>
            <v-btn icon v-if="item.line_id==currentUserId" :to="'/posts/' + item.id + '/edit'"><v-icon>mdi-pencil</v-icon></v-btn>
            <v-btn icon v-if="item.line_id==currentUserId" @click="deleteConfirm(index)"><v-icon>mdi-delete</v-icon></v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
    <v-dialog v-model="showDialog">
      <v-card
        elevation="2"
      >
        <v-card-title>確認</v-card-title>
        <v-card-text>
          削除しますか？
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="deletePost">はい</v-btn>
          <v-btn text @click="showDialog=false">いいえ</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
import axios from '~/plugins/axios'
export default {
  props: {
    data: Array,
    currentUserId: String
  },
  data: function() {
    return {
      showDialog: false,
      deleteIndex: 0,
    };
  },
  methods: {
    iconColor: function(like) {
      return like ? "blue" : ""
    },
    like: function(index) {
      if (this.data[index].like) {
        let data = {
          line_id_token: this.$liff.getIDToken()
        }
        axios.delete(`/posts/${this.data[index].id}/like`, {data: data})
          .then((response) => {
            let item = this.data[index]
            item.like = false
            this.$set(this.data, index, item)
          })
          .catch((error) => {
            console.log(error)
          })
      } else {
        let data = {
          post_id: this.data[index].id,
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
      }
    },
    bookmark: function(index) {
      if (this.data[index].bookmark) {
        let data = {
          line_id_token: this.$liff.getIDToken()
        }
        axios.delete(`/posts/${this.data[index].id}/bookmark`, {data: data})
          .then((response) => {
            let item = this.data[index]
            item.bookmark = false
            this.$set(this.data, index, item)
          })
          .catch((error) => {
            console.log(error)
          })
      } else {
        let data = {
          post_id: this.data[index].id,
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
    },
    deleteConfirm: function(index) {
      this.deleteIndex = index
      this.showDialog = true
    },
    deletePost: function() {
      axios.delete(`/posts/${this.data[this.deleteIndex].id}`)
        .then((response) => {
          this.data.splice(this.deleteIndex, 1)
          this.deleteIndex = 0
          this.showDialog = false
        })
        .catch((error) => {
          console.log(error)
        })
    },
    tagSearch: function(tag) {
      if (this.$route.path == '/') {
        this.$emit('tagClick', tag)
      } else {
        this.$router.push({path: '/', query :{tag:tag}})
      }
    }
  }
};
</script>