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
    ) {{ date }}
  v-date-picker(
    v-model="date"
    :type="type"
    :locale="locale"
    :day-format="e => new Date(e).getDate()"
    no-title
    @input="input()"
  )
</template>

<script>
export default {
  model: {
    prop: 'date',
    event: 'input'
  },
  props: ['date', 'type', 'locale'],
  data: () => ({
    menu: false
  }),
  methods: {
    input () {
      this.menu = false
      this.$emit('input', this.date)
    }
  }
}
</script>
