<template lang="pug">
.content
  v-container
    v-row
      v-col(cols=12)
        v-card.pa-4.mx-auto(max-width=400)
          h1 MyPage
          .text-center
            v-btn(
              @click="logout"
              color="primary"
              depressed
            ) ログアウト
</template>

<script>
export default {
  middleware ({ store, redirect }) {
    if (!store.state.auth.loggedIn) {
      return redirect('/')
    }
  },
  data () {
    return {

    }
  },
  methods: {
    async logout () {
      try {
        await this.$auth.logout()
        await this.$apolloHelpers.onLogout()
        this.$router.push('/')
      } catch (e) {
        // window.console.log(e)
      }
    }
  }
}
</script>
