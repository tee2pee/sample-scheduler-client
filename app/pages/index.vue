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
              )
              v-text-field(
                type="password"
                v-model="password"
                label="パスワード"
                :rules="[v => !!v || 'パスワードを入力してください。']"
                required
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
    if (store.state.auth.loggedIn) {
      return redirect('/mypage')
    }
  },
  data () {
    return {
      valid: true,
      email: '',
      password: '',
      snackbar: {
        show: false,
        text: ''
      }
    }
  },
  methods: {
    async login () {
      if (!this.$refs.form.validate()) {
        return false
      }
      try {
        await this.$auth.loginWith('local', {
          data: {
            auth: {
              email: this.email,
              password: this.password
            }
          }
        })
        await this.$apolloHelpers.onLogin(this.$auth.getToken('local').match(/^Bearer\s+(.+)$/i)[1])
        this.$router.push('/mypage')
      } catch (e) {
        this.snackbar.text = 'ログインに失敗しました。'
        this.snackbar.show = true
        // window.console.log(e)
      }
    }
  }
}
</script>
