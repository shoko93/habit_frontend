<template>
  <v-app>
    <v-navigation-drawer app v-model="drawer" :bottom="true">
      <!-- -->
      <v-list
        dense
        nav
      >
        <v-list-item to='/'>
          <v-list-item-content>
            <v-list-item-title><v-icon>mdi-format-list-bulleted</v-icon> 一覧</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item to='/posts/create'>
          <v-list-item-content>
            <v-list-item-title><v-icon>mdi-pencil</v-icon> 新規登録</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item to='/posts/bookmark'>
          <v-list-item-content>
            <v-list-item-title><v-icon>mdi-heart</v-icon> お気に入り</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item @click="logout">
          <v-list-item-content>
            <v-list-item-title><v-icon>mdi-logout</v-icon> ログアウト</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar app v-if="!$vuetify.breakpoint.mobile">
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <!-- -->
    </v-app-bar>

    <!-- Sizes your content based upon application components -->
    <v-main>

      <!-- Provides the application the proper gutter -->
      <v-container fluid>
        <Nuxt />
      </v-container>
    </v-main>

    <v-footer app>
      <!-- -->
      <v-row v-if="$vuetify.breakpoint.mobile">
        <v-col col="6">
          <v-btn
            elevation="2"
            fab
            icon
            @click="$router.go(-1)"
          ><v-icon>mdi-chevron-left</v-icon></v-btn>
        </v-col>
        <v-col col="6" class="text-right">
          <v-btn
            elevation="2"
            fab
            icon
            @click="drawer=true"
          ><v-icon>mdi-menu</v-icon></v-btn>
        </v-col>
      </v-row>
    </v-footer>
  </v-app>
</template>
<script>
  export default {
    data: () => ({
      drawer: false,
    }),
    methods: {
      logout: function() {
        this.$liff.logout()
        location.reload();
      },
    }
  }
</script>