<template>
  <div>
    <v-row v-for="(item, index) in data" :key="item.id">
      <v-col col="12">
        <PostItem
          :post="item"
          :currentUserId="currentUserId"
          @deleteConfirm="deleteConfirm"
          @tagClick="tagClick"
        ></PostItem>
      </v-col>
    </v-row>
    <DeleteDialog
      :showDialog="showDialog"
      @closeDialog="showDialog=false"
      @deletePost="deletePost"
    ></DeleteDialog>
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
    deleteConfirm: function(item) {
      this.deleteIndex = this.data.findIndex(element => element.id === item.id);
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
    tagClick: function(tag) {
      this.$emit('tagClick', tag)
    }
  }
};
</script>