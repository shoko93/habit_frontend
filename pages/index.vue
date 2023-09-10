<template>
  <div>
    <v-row v-for="(item, index) in data" :key="item.id">
      <v-col col="12">
        <v-card
          elevation="2"
        >
          <v-card-title>{{ item.title }}</v-card-title>
          <v-card-text>
            {{ item.created_at}}<br>
            {{ item.text_body }}
            <v-row class="mt-2">
              <v-col col="12">
                <v-btn text v-for="tag in item.tags" :key="tag.id">{{ tag.tag }}</v-btn>
              </v-col>
            </v-row>
          </v-card-text>
          <v-card-actions>
            <v-btn icon @click="like(index)"><v-icon :color="iconColor(item.like)">mdi-thumb-up</v-icon></v-btn>
            <v-btn icon :to="'/posts/' + item.id + '/comment'"><v-icon>mdi-comment-text</v-icon></v-btn>
            <v-btn icon @click="bookmark(index)"><v-icon :color="iconColor(item.bookmark)">mdi-heart</v-icon></v-btn>
          </v-card-actions>
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
      data: [],
      message: "",
      showMessage: false,
    };
  },
  async mounted() {
    this.$liffInit
      .then(() => {
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
  },
  methods: {
    iconColor: function(like) {
      return like ? "blue" : ""
    },
    like: function(index) {
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
    },
    bookmark: function(index) {
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
  }
};
</script>