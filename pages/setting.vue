<template>
  <div>
    <div v-show="showMessage">
      <p>{{ message }}</p>
    </div>
    <v-row>
      <v-col col="12">
        <v-text-field label="ユーザ名" v-model="userName"></v-text-field>
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
      userName: "",
      sdkVersion: "",
      liffError: "",
      message: "",
      showMessage: false,
    };
  },
  async mounted() {
    this.$liffInit
      .then(() => {
        this.sdkVersion = liff.getVersion();
        axios.post('/users', {line_id_token: this.$liff.getIDToken()})
          .then((response) => {
            if (response.data.user) {
              this.userName = response.data.user.name
            } else {
              this.$liff
                .getProfile()
                .then((profile) => {
                  this.userName = profile.displayName
                })
                .catch((err) => {
                  console.log("error", err)
                });
            }
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
    submit: function() {
      let data = {
        name: this.userName,
        line_id_token: this.$liff.getIDToken()
      }
      axios.put('/users', {user: data})
        .then((response) => {
          this.message = "設定完了しました"
          this.showMessage = true
        })
        .catch((error) => {
          this.message = "設定に失敗しました"
          this.showMessage = true
        })
    }
  }
};
</script>