<template>
  <v-card
    elevation="2"
  >
    <v-card-title>{{ item.title }}</v-card-title>
    <v-card-text>
      <p class="text-caption"><span v-if="item.user">{{ item.user.name }} / </span>{{ item.created_at}}</p>
      <v-img
        v-if="item.image.url"
        max-width="100%"
        :src="item.image.url"
        class="mb-4"
      ></v-img>
      <p class="text-body-1">{{ item.text_body }}</p>
      <v-row class="mt-2">
        <v-col col="12">
          <v-btn class="mr-2" outlined rounded v-for="tag in item.tags" :key="tag.id" @click="tagSearch(tag.tag)">{{ tag.tag }}</v-btn>
        </v-col>
      </v-row>
    </v-card-text>
    <v-card-actions>
      <v-row>
        <v-col col="12" class="text-right">
          <v-btn icon @click="like"><v-icon :color="iconColor(item.like)">mdi-thumb-up</v-icon></v-btn>
          <v-btn icon :to="'/posts/' + item.id + '/comment'"><v-icon>mdi-comment-text</v-icon></v-btn>
          <v-btn icon @click="bookmark"><v-icon :color="iconColor(item.bookmark)">mdi-heart</v-icon></v-btn>
          <v-btn icon v-if="isNavigatorShareButton" @click="navigatorShare"><v-icon>mdi-share-variant</v-icon></v-btn>
          <v-btn icon v-if="item.line_id==currentUserId" :to="'/posts/' + item.id + '/edit'"><v-icon>mdi-pencil</v-icon></v-btn>
          <v-btn icon v-if="item.line_id==currentUserId" @click="deleteConfirm(item)"><v-icon>mdi-delete</v-icon></v-btn>
        </v-col>
      </v-row>
    </v-card-actions>
  </v-card>
</template>
<script>
import axios from '~/plugins/axios'
export default {
  props: {
    post: Object,
    currentUserId: String
  },
  data: function() {
    return {
      item: {
        like: false,
        bookmark: false,
        image: {
          url: ""
        }
      },
      showDialog: false,
      showIcon: false,
      deleteIndex: 0,
      isNavigatorShareButton: true,
    };
  },
  mounted() {
    if (navigator.share === undefined) {
      this.isNavigatorShareButton = false
    }
    this.item = JSON.parse(JSON.stringify(this.post))
  },
  watch: { 
    post: function(newVal, oldVal) { // watch it
      this.item = JSON.parse(JSON.stringify(newVal))
    }
  },
  methods: {
    navigatorShare() {
      if (navigator.share) {
        navigator.share({
          title: this.item.title,
          text: this.item.text_body,
          url: `${window.location.origin}/posts/${this.item.id}/comment`
        })
      }
    },
    iconColor: function(like) {
      return like ? "blue" : ""
    },
    like: function() {
      if (this.item.like) {
        let data = {
          line_id_token: this.$liff.getIDToken()
        }
        axios.delete(`/posts/${this.item.id}/like`, {data: data})
          .then((response) => {
            this.item.like = false
          })
          .catch((error) => {
            console.log(error)
          })
      } else {
        let data = {
          post_id: this.item.id,
          line_id_token: this.$liff.getIDToken()
        }
        axios.post('/posts/like', {like: data})
          .then((response) => {
            this.item.like = true
          })
          .catch((error) => {
            console.log(error)
          })
      }
    },
    bookmark: function() {
      if (this.item.bookmark) {
        let data = {
          line_id_token: this.$liff.getIDToken()
        }
        axios.delete(`/posts/${this.item.id}/bookmark`, {data: data})
          .then((response) => {
            this.item.bookmark = false
          })
          .catch((error) => {
            console.log(error)
          })
      } else {
        let data = {
          post_id: this.item.id,
          line_id_token: this.$liff.getIDToken()
        }
        axios.post('/posts/bookmark', {bookmark: data})
          .then((response) => {
            this.item.bookmark = true
          })
          .catch((error) => {
            console.log(error)
          })
      }
    },
    deleteConfirm: function(item) {
      this.$emit('deleteConfirm', item)
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