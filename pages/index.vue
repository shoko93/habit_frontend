<template>
  <div>
    <v-row>
      <v-col col="12">
        <v-card
          elevation="2"
        >
          <v-card-title>検索</v-card-title>
          <v-card-text>
            <v-row>
              <v-col col="12">
                <v-text-field label="キーワード" v-model="keyword"></v-text-field>
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
          </v-card-text>
          <v-card-actions>
            <v-row>
              <v-col col="12" class="text-right">
                <v-btn
                  depressed
                  color="primary"
                  @click="search"
                >
                  検索
                </v-btn>
              </v-col>
            </v-row>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
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
import moment from 'moment'
export default {
  data: function() {
    return {
      data: [],
      message: "",
      showMessage: false,
      currentUserId: "",
      showDialog: false,
      deleteIndex: 0,
      select: [],
      items: [],
      keyword: ""
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
        axios.post('/posts', {line_id_token: this.$liff.getIDToken()})
          .then((response) => {
            this.data = response.data.map((item) => {
              item.created_at = moment(item.created_at).format('YYYY-MM-DD H:m')
              return item
            });
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
    search: function() {
      let data = {
        keyword: this.keyword,
        tags: this.select,
        line_id_token: this.$liff.getIDToken()
      }
      axios.post('/posts/search', data)
        .then((response) => {
          this.data = response.data.map((item) => {
            item.created_at = moment(item.created_at).format('YYYY-MM-DD H:m')
            return item
          });
        })
        .catch((error) => {
          console.log(error)
        })
    },
    tagSearch: function(tag) {
      this.keyword = ""
      this.select = []
      this.select.push(tag)
      this.search()
    }
  }
};
</script>