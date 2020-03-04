<template lang="pug">
.content
  v-container
    v-row
      v-col(cols=12)
        v-card.mx-auto(max-width=400)
          v-toolbar(
            color="primary"
            dark
            flat
          )
            v-toolbar-title ログイン
          v-card-text
            v-form(
              ref="form"
              v-model="valid"
              lazy-validation
              @submit.prevent="login"
            )
              v-text-field(
                v-model="email"
                label="メールアドレス"
                :rules="[v => !!v || 'メールアドレスを入力してください。']"
                required
                prepend-icon="mdi-account"
              )
              v-text-field(
                type="password"
                v-model="password"
                label="パスワード"
                :rules="[v => !!v || 'パスワードを入力してください。']"
                required
                prepend-icon="mdi-lock"
              )
              v-card-actions
                v-spacer
                v-btn(
                  type="submit"
                  color="primary"
                  depressed
                ) ログイン
                v-spacer
  v-snackbar(
    v-model="snackbar.show"
    top
  ) {{ snackbar.text }}
</template>

<script>
export default {
  middleware ({ store, redirect }) {
    // 既にログイン済みの場合、マイページへ遷移
    if (store.state.auth.loggedIn) {
      return redirect('/mypage')
    }
  },
  data () {
    return {
      // フォームバリデーション
      valid: true,
      // 入力項目
      email: '',
      password: '',
      // スナックバー
      snackbar: {
        show: false,
        text: ''
      }
    }
  },
  methods: {
    async login () {
      // バリデーションNGの場合、処理終了
      if (!this.$refs.form.validate()) {
        return false
      }
      try {
        // Knockログイン
        await this.$auth.loginWith('local', {
          data: {
            auth: {
              email: this.email,
              password: this.password
            }
          }
        })
        // ApolloにJWTトークン設定
        await this.$apolloHelpers.onLogin(this.$auth.getToken('local').match(/^Bearer\s+(.+)$/i)[1])
        // マイページへ遷移
        this.$router.push('/mypage')
      } catch (e) {
        // ログイン失敗
        this.snackbar.text = 'ログインに失敗しました。'
        this.snackbar.show = true
        // window.console.log(e)
      }
    }
  }
}
</script>
