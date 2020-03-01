<template lang="pug">
.content
  v-container
    v-row
      v-col(cols=12)
        v-card.pa-4.mx-auto(max-width=400)
          h1.text-center ログイン
          v-form(
            ref="form"
            v-model="valid"
            lazy-validation
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
            .text-center
              v-btn(
                @click="login"
                color="primary"
                depressed
              ) ログイン
  v-snackbar(
    v-model="snackbar.show"
    top
  ) {{ snackbar.text }}
</template>

<script>
export default {
}
</script>

<script>
export default {
  data: () => ({
    valid: true,
    email: '',
    password: '',
    snackbar: {
      show: false,
      text: '',
    }
  }),
  methods: {
    async login() {
      if (!this.$refs.form.validate()) {
        return;
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
