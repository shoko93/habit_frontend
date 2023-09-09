<template>
  <div>
    <v-row v-for="item in data" :key="item.id">
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
            <v-btn text>いいね</v-btn>
            <v-btn text>コメント</v-btn>
            <v-btn text>お気に入り</v-btn>
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
        console.log(this.$liff.getIDToken())
      })
      .catch((error) => {
        this.liffError = error;
      });
    axios.get('/posts')
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
  methods: {
    submit: function() {
      let data = {
        title: this.title,
        text_body: this.textBody,
        line_id_token: this.$liff.getIDToken()
      }
      axios.post('/posts', {post: data})
        .then((response) => {
          this.message = "登録完了しました"
          this.showMessage = true
        })
        .catch((error) => {
          this.message = "登録に失敗しました"
          this.showMessage = true
        })
    }
  }
};
</script>