<template>
  <div>
    <div v-show="showMessage">
      <p>{{ message }}</p>
    </div>
    <v-row>
      <v-col col="12">
        <v-text-field label="タイトル" v-model="title"></v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col col="12">
        <v-textarea label="本文" v-model="textBody"></v-textarea>
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
      title: "",
      textBody: "",
      sdkVersion: "",
      liffError: "",
      message: "",
      showMessage: false,
      select: [],
      items: [],
    };
  },
  async mounted() {
    this.$liffInit
      .then(() => {
        this.sdkVersion = liff.getVersion();
      })
      .catch((error) => {
        this.liffError = error;
      });
  },
  methods: {
    submit: function() {
      let data = {
        title: this.title,
        text_body: this.textBody,
        tags: this.select,
        line_id_token: this.$liff.getIDToken()
      }
      axios.post('/posts/create', {post: data})
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