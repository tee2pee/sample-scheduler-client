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
              ref="loginForm"
              v-model="loginForm.valid"
              lazy-validation
              @submit.prevent="login"
            )
              v-text-field(
                v-model="loginForm.email"
                label="メールアドレス"
                :rules="[v => !!v || 'メールアドレスを入力してください。']"
                required
                prepend-icon="mdi-account"
              )
              v-text-field(
                type="password"
                v-model="loginForm.password"
                label="パスワード"
                :rules="[v => !!v || 'パスワードを入力してください。']"
                required
                prepend-icon="mdi-lock"
              )
              v-card-actions
                v-spacer
                v-btn(
                  depressed
                  type="submit"
                  color="primary"
                ) ログイン
                v-spacer
              v-card-actions.pa-0
                v-spacer
                v-btn(
                  text
                  small
                  @click="signupDialog.show = true"
                ) ユーザー登録
                v-spacer
  v-dialog(
    v-model="signupDialog.show"
    max-width="450"
  )
    v-card
      v-toolbar(
        flat
        dark
        color="primary"
        class="mb-2"
      )
        v-btn(
          icon
          dark
          @click="signupDialog.show = false"
        )
          v-icon mdi-close
      v-card-text
        v-form(
          ref="signupDialog"
          v-model="signupDialog.valid"
          lazy-validation
          @submit.prevent="signup"
        )
          v-text-field(
            v-model="signupDialog.name"
            label="名前"
            :rules="[v => !!v || '名前を入力してください。']"
            required
            prepend-icon="mdi-account"
          )
          v-text-field(
            v-model="signupDialog.email"
            label="メールアドレス"
            :rules="[v => !!v || 'メールアドレスを入力してください。']"
            required
            prepend-icon="mdi-email"
          )
          v-text-field(
            type="password"
            v-model="signupDialog.password"
            label="パスワード"
            :rules="[v => !!v || 'パスワードを入力してください。']"
            required
            prepend-icon="mdi-lock"
          )
          v-text-field(
            type="password"
            v-model="signupDialog.password_confirmation"
            label="パスワード（再入力）"
            :rules="[v => !!v || 'パスワード（再入力）を入力してください。']"
            required
            prepend-icon="mdi-lock"
          )
          v-card-actions
            v-spacer
            v-btn(
              depressed
              type="submit"
              color="primary"
            ) ユーザー登録
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
      // ログインフォーム
      loginForm: {
        // 状態
        valid: true,
        // 入力項目
        email: '',
        password: ''
      },
      // ユーザー登録
      signupDialog: {
        // 状態
        show: false,
        // 入力項目
        name: '',
        email: '',
        password: '',
        password_confirmation: ''
      },
      // スナックバー
      snackbar: {
        show: false,
        text: ''
      }
    }
  },
  methods: {
    async loginCore (email, password) {
      try {
        // Knockログイン
        await this.$auth.loginWith('local', {
          data: {
            auth: {
              email,
              password
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
    },
    async login () {
      // バリデーションNGの場合、処理終了
      if (!this.$refs.loginForm.validate()) {
        return false
      }
      // ログイン
      await this.loginCore(
        this.loginForm.email,
        this.loginForm.password
      )
    },
    async signup () {
      // バリデーションNGの場合、処理終了
      if (!this.$refs.signupDialog.validate()) {
        return false
      }
      try {
        // ユーザー登録
        await this.$axios.$post('/signup', {
          user: {
            name: this.signupDialog.name,
            email: this.signupDialog.email,
            password: this.signupDialog.password,
            password_confirmation: this.signupDialog.password_confirmation
          }
        })
        // ログイン
        await this.loginCore(
          this.signupDialog.email,
          this.signupDialog.password
        )
        // モーダル終了
        this.signupDialog.show = false
      } catch (e) {
        // ユーザー登録失敗
        this.snackbar.text = 'ユーザー登録に失敗しました。'
        this.snackbar.show = true
        // window.console.log(e)
      }
    }
  }
}
</script>
