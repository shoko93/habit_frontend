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
    <PostList
      :data="data"
      :currentUserId="currentUserId"
      @tagClick="tagSearch"
    ></PostList>
  </div>
</template>
<script>
import axios from '~/plugins/axios'
import moment from 'moment'
export default {
  data: function() {
    return {
      data: [],
      currentUserId: "",
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
        if (this.$route.query.tag) {
          this.tagSearch(this.$route.query.tag)
        } else {
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
          }
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