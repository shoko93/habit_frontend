<template>
  <div>
    <div>
      タイトル<br>
      <input type="text" v-model="title">
    </div>
    <div>
      本文<br>
      <textarea v-model="text_bdoy"></textarea>
    </div>
    <div>
      <button @click="submit">送信</button>
    </div>
  </div>
</template>
<script>
import axios from '~/plugins/axios'
export default {
  data: function() {
    return {
      title: "",
      text_bdoy: "",
      sdkVersion: "",
      liffError: ""
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
    submit: async function() {
      let data = {
        title: this.title,
        text_body: this.text_bdoy,
        line_id_token: this.$liff.getIDToken()
      }
      await axios.post('/posts', {post: data})
    }
  }
};
</script>