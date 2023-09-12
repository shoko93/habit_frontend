<template>
  <v-dialog v-model="showDialog">
    <v-card
      elevation="2"
    >
      <v-card-title>確認</v-card-title>
      <v-card-text>
        削除しますか？
      </v-card-text>
      <v-card-actions>
        <v-btn text @click="$emit('deletePost')">はい</v-btn>
        <v-btn text @click="$emit('closeDialog')">いいえ</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>
<script>
export default {
  props: {
    showDialog: Boolean
  },
};
</script>