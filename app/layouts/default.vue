<template lang="pug">
v-app
  v-navigation-drawer(
    v-model="sidebar"
    :mobile-break-point="this.$vuetify.breakpoint.thresholds.sm"
    clipped
    fixed
    app
    v-if="!isLoginPage"
  )
    v-list
      v-list-item(
        v-for="(item, i) in items"
        :key="i"
        :to="item.to"
        router
        exact
      )
        v-list-item-action
          v-icon {{ item.icon }}
        v-list-item-content
          v-list-item-title(
            v-text="item.title"
          )
  v-app-bar(
    clipped-left
    fixed
    app
  )
    v-app-bar-nav-icon(
      @click.stop="sidebar = !sidebar"
      v-if="!isLoginPage && !isLaptop"
    )
    v-spacer(
      v-if="!isLaptop"
    )
    v-toolbar-title(
      v-text="title"
    )
    v-spacer
    v-menu(
      left
      bottom
      v-if="!isLoginPage"
    )
      template(
        v-slot:activator="{ on }"
      )
        v-btn(
          icon
          v-on="on"
        )
          v-icon mdi-dots-vertical
      v-list
        v-list-item(
          v-if="isLoggedIn"
          @click="logout()"
        )
          v-list-item-title ログアウト
  v-content
    v-container
      nuxt
  v-footer(
    fixed
    app
  )
    .text-center &copy; {{ new Date().getFullYear() }}
  v-overlay(
    :value="loading.show"
    :color="loading.color"
    :opacity="loading.opacity"
    z-index="500"
  )
    v-progress-circular(
      indeterminate
      :color="loading.progcolor"
      size="64"
    )
  v-snackbar(
    v-model="snackbar.show"
    top
  ) {{ snackbar.text }}
</template>

<script>
export default {
  data () {
    return {
      // ページタイトル
      title: '',
      // サイドバー表示ステータス
      sidebar: this.isLaptop,
      // ローディング
      loading: {
        show: true,
        color: 'white',
        opacity: 1
      },
      // スナックバー
      snackbar: {
        show: false,
        text: '',
        progcolor: 'pink accent-2'
      },
      // dummy
      items: [
        // FIXME: カレンダー
        /*
        {
          icon: 'mdi-apps',
          title: 'Welcome',
          to: '/'
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'Inspire',
          to: '/inspire'
        }
        */
      ]
    }
  },
  beforeMount () {
    this.$nuxt.$on('setPageTitle', (title) => { this.title = title })
    this.$nuxt.$on('messaging', (msg) => {
      this.snackbar.text = msg
      this.snackbar.show = true
    })
    this.$nuxt.$on('loading', () => {
      this.loading.color = 'black'
      this.loading.opacity = 0.5
      this.loading.progcolor = 'white'
      this.loading.show = true
    })
    this.$nuxt.$on('loaded', () => {
      this.loading.show = false
    })
  },
  mounted () {
    this.loading.show = false
  },
  computed: {
    isLoggedIn () {
      return this.$store.state.auth.loggedIn
    },
    isLoginPage () {
      // ログインページ判定
      return this.$route.path === '/'
    },
    isLaptop () {
      // ラップトップ画面判定
      return this.$vuetify.breakpoint.mdAndUp
    }
  },
  methods: {
    async logout () {
      try {
        // Knockログアウト
        await this.$auth.logout()
        // ApolloからJWTトークンを削除
        await this.$apolloHelpers.onLogout()
        // ログインページへ遷移
        this.$router.push('/')
      } catch (e) {
        // window.console.log(e)
      }
    }
  }
}
</script>
