<template lang="pug">
v-menu(
  ref="menu"
  v-model="menu"
  :close-on-content-click="false"
  transition="scale-transition"
  offset-y
  min-width="290px"
)
  template(
    v-slot:activator="{ on }"
  )
    v-btn(
      text
      v-on="on"
    ) {{ time }}
  v-time-picker(
    v-if="menu"
    v-model="time"
    format="24hr"
    no-title
    @click:minute="input()"
  )
</template>

<script>
export default {
  model: {
    prop: 'time',
    event: 'input'
  },
  props: ['time'],
  data: () => ({
    menu: false
  }),
  methods: {
    input () {
      this.$refs.menu.save(this.time)
      this.$emit('input', this.time)
    }
  }
}
</script>
