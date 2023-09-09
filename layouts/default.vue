<template>
  <v-app>
    <v-navigation-drawer app v-model="drawer">
      <!-- -->
      <v-list
        dense
        nav
      >
        <v-list-item to='/'>
          <v-list-item-content>
            <v-list-item-title>一覧</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item to='/posts/create'>
          <v-list-item-content>
            <v-list-item-title>新規登録</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item to='/posts/bookmark'>
          <v-list-item-content>
            <v-list-item-title>お気に入り</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar app>
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
    </v-footer>
  </v-app>
</template>
<script>
  export default {
    data: () => ({
      drawer: false,
    }),
  }
</script>