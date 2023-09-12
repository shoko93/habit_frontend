<template>
  <PostList
    :data="data"
    :currentUserId="currentUserId"
  ></PostList>
</template>
<script>
import axios from '~/plugins/axios'
import moment from 'moment'
export default {
  data: function() {
    return {
      data: [],
      currentUserId: "",
    };
  },
  async mounted() {
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
        axios.post('/posts/bookmarks', {line_id_token: this.$liff.getIDToken()})
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
};
</script>