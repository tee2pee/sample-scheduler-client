<template lang="pug">
.content
  v-sheet(
    tile
    height="54"
    class="d-flex"
  )
    v-btn(
      icon
      class="ma-2"
      @click="$refs.calendar.prev()"
    )
      v-icon mdi-chevron-left
    v-spacer
    .ma-2
      ButtonDatePicker(
        v-model="calendar.date"
        :type="calendar.type"
        :locale="locale"
      )
    v-spacer
    v-btn(
      icon
      class="ma-2"
      @click="$refs.calendar.next()"
    )
      v-icon mdi-chevron-right
  v-sheet(
    height="600"
  )
    v-calendar(
      ref="calendar"
      v-model="calendar.focus"
      :type="calendar.type"
      :weekdays="calendar.weekdays"
      :events="calendar.events"
      :event-overlap-mode="mode"
      :event-overlap-threshold="30"
      :event-color="getEventColor"
      @click:event="showModifyEventModal"
      @click:date="showDailySchedule"
      @click:more="showDailySchedule"
      @click:day="showCreateEventModal"
      @change="onUpdateCalendar"
    )
  v-dialog(
    v-model="scheduleDialog.show"
    max-width="400"
  )
    v-card
      v-toolbar(
        flat
        dark
        color="primary"
        class="mb-2"
      )
        v-toolbar-title スケジュール
        v-spacer
        v-btn(
          v-if="scheduleDialog.mode === 2"
          icon
          dark
          @click="deleteSchedule()"
        )
          v-icon mdi-delete
        v-btn(
          icon
          dark
          @click="scheduleDialog.show = false"
        )
          v-icon mdi-close
      v-card-text
        v-sheet(
          tile
          class="d-flex"
        )
          v-select(
            v-model="scheduleDialog.calendarId"
            :items="myInfo.calendars"
            item-text="title"
            item-value="id"
            label="カレンダー"
            single-line
          )
        v-sheet(
          tile
          class="d-flex"
        )
          v-text-field(
            label="タイトル"
            v-model="scheduleDialog.title"
            required
          )
        v-sheet(
          tile
          class="d-flex"
        )
          ButtonDatePicker(
            v-model="scheduleDialog.frDate"
            :type="'day'"
            :locale="locale"
          )
          v-spacer
          ButtonTimePicker(
            v-model="scheduleDialog.frTime"
          )
        v-sheet(
          tile
          class="d-flex"
        )
          ButtonDatePicker(
            v-model="scheduleDialog.toDate"
            :type="'day'"
            :locale="locale"
          )
          v-spacer
          ButtonTimePicker(
            v-model="scheduleDialog.toTime"
          )
      v-card-actions
        v-spacer
        v-btn(
          color="blue darken-1"
          text
          @click="saveSchedule()"
        ) 保存
  v-snackbar(
    v-model="snackbar.show"
    top
  ) {{ snackbar.text }}
</template>

<script>
import apolloMyInfo from '~/apollo/queries/myInfo'
import apolloMySchedules from '~/apollo/queries/mySchedules'
import apolloCreateSchedule from '~/apollo/mutations/createSchedule'
import apolloModifySchedule from '~/apollo/mutations/modifySchedule'
import apolloDeleteSchedule from '~/apollo/mutations/deleteSchedule'
import ButtonDatePicker from '~/components/ButtonDatePicker.vue'
import ButtonTimePicker from '~/components/ButtonTimePicker.vue'
export default {
  components: {
    ButtonDatePicker,
    ButtonTimePicker
  },
  middleware ({ store, redirect }) {
    // ログインしていない場合、ログインページへ遷移
    if (!store.state.auth.loggedIn) {
      return redirect('/')
    }
  },
  data () {
    return {
      // ログインユーザー情報
      myInfo: 'xxx',
      // ロケール
      locale: 'ja',
      // カレンダー
      // FIXME: 年月ボタン押下時の処理未実装
      calendar: {
        focus: '',
        date: this.formatDate(new Date(), 'YYYY-MM-01'),
        type: 'month',
        weekdays: [0, 1, 2, 3, 4, 5, 6],
        events: []
      },
      // スケジュールダイアログ
      scheduleDialog: {
        // 状態
        show: false,
        mode: null,
        // 入力項目
        scheduleId: -1,
        calendarId: -1,
        title: '',
        frDate: '',
        frTime: '',
        toDate: '',
        toTime: '',
        // 画面イベントオブジェクト
        calendarEvent: null
      },
      // スナックバー
      // FIXME: indexと共通化したい
      snackbar: {
        show: false,
        text: ''
      },
      events: [],
      // dummy
      type: 'month',
      types: ['month', 'week', 'day', '4day'],
      mode: 'stack',
      modes: ['stack', 'column'],
      weekday: [0, 1, 2, 3, 4, 5, 6],
      weekdays: [
        { text: 'Sun - Sat', value: [0, 1, 2, 3, 4, 5, 6] },
        { text: 'Mon - Sun', value: [1, 2, 3, 4, 5, 6, 0] },
        { text: 'Mon - Fri', value: [1, 2, 3, 4, 5] },
        { text: 'Mon, Wed, Fri', value: [1, 3, 5] }
      ],
      value: '',
      colors: ['blue', 'indigo', 'deep-purple', 'cyan', 'green', 'orange', 'grey darken-1'],
      names: ['Meeting', 'Holiday', 'PTO', 'Travel', 'Event', 'Birthday', 'Conference', 'Party']
    }
  },
  created () {
    this.$nuxt.$emit('setPageTitle', 'マイページ')
    this.mountMyInfo()
  },
  methods: {
    formatDate: (() => {
      const conv = (val) => {
        return (mat) => {
          return String(val).padStart(mat.length, '0')
        }
      }
      return (date, format) => {
        return format
          .replace(/Y{1,4}/m, conv(date.getFullYear()))
          .replace(/M{1,2}/m, conv(date.getMonth() + 1))
          .replace(/D{1,2}/m, conv(date.getDate()))
          .replace(/h{1,2}/m, conv(date.getHours()))
          .replace(/m{1,2}/m, conv(date.getMinutes()))
          .replace(/s{1,2}/m, conv(date.getSeconds()))
      }
    })(),
    convDatetime (date, time) {
      // 画面の日時から一旦Dateに変換することでタイムゾーンの影響対策
      const d = date.split('-').map(e => parseInt(e))
      d[1] -= 1 // 月補正
      return new Date(
        ...d,
        ...time.split(':')
      ).toISOString().substr(0, 19) + 'Z'
    },
    toCalendarEvent (schedule) {
      const e = {}
      e.name = schedule.title
      // FIXME: 終日未対応
      e.start = this.formatDate(new Date(schedule.frDatetime), 'YYYY-MM-DD')
      e.end = this.formatDate(new Date(schedule.toDatetime), 'YYYY-MM-DD')
      e.color = 'indigo' // FIXME: 色の設定を要検討
      e.schedule = schedule
      return e
    },
    async mountMyInfo () {
      try {
        const res = await this.$apollo.query({ query: apolloMyInfo })
        this.myInfo = res.data.myInfo
      } catch (e) {
        this.snackbar.text = 'ユーザー情報の取得に失敗しました。'
        this.snackbar.show = true
        // window.console.log(e)
      }
    },
    // FIXME: カレンダーで選択したときどうしよう
    async onUpdateCalendar ({ start, end }) {
      // FIXME: daily未対応
      this.calendar.date = start.date.substr(0, 7)
      try {
        const res = await this.$apollo.query({
          query: apolloMySchedules,
          variables: {
            // FIXME: カレンダーの先頭～末尾ではなく、月初～月末でとれてしまう
            fr: `${start.date}T00:00:00Z`,
            to: `${end.date}T00:00:00Z`
          }
        })
        const events = []
        for (const e of res.data.mySchedules) {
          events.push(this.toCalendarEvent(e))
        }
        this.calendar.events = events
      } catch (e) {
        this.snackbar.text = 'スケジュールの取得に失敗しました。'
        this.snackbar.show = true
        // window.console.log(e)
      }
    },
    showCreateEventModal ({ date }) {
      window.console.log(this.myInfo)
      // 入力項目設定
      this.scheduleDialog.scheduleId = -1
      this.scheduleDialog.calendarId = this.myInfo.calendars[0].id
      this.scheduleDialog.title = '新しい予定'
      // FIXME: 終日未対応
      this.scheduleDialog.frDate = date
      this.scheduleDialog.frTime = '12:00'
      this.scheduleDialog.toDate = date
      this.scheduleDialog.toTime = '12:00'
      // 画面イベントオブジェクト設定
      this.scheduleDialog.calendarEvent = null
      // 状態設定
      this.scheduleDialog.mode = 1
      this.scheduleDialog.show = true
    },
    showModifyEventModal ({ event, nativeEvent }) {
      // click:dayにバブリングしてしまう対応
      nativeEvent.stopPropagation()
      // 入力項目設定
      this.scheduleDialog.scheduleId = event.schedule.id
      this.scheduleDialog.calendarId = event.schedule.calendarId
      this.scheduleDialog.title = event.schedule.title
      // FIXME: 終日未対応
      const frDatetime = new Date(event.schedule.frDatetime)
      this.scheduleDialog.frDate = this.formatDate(frDatetime, 'YYYY-MM-DD')
      this.scheduleDialog.frTime = this.formatDate(frDatetime, 'hh:mm')
      const toDatetime = new Date(event.schedule.toDatetime)
      this.scheduleDialog.toDate = this.formatDate(toDatetime, 'YYYY-MM-DD')
      this.scheduleDialog.toTime = this.formatDate(toDatetime, 'hh:mm')
      // 画面イベントオブジェクト設定
      this.scheduleDialog.calendarEvent = event
      // 状態設定
      this.scheduleDialog.mode = 2
      this.scheduleDialog.show = true
    },
    async saveSchedule () {
      try {
        let mutation
        const variables = {
          calendarId: this.scheduleDialog.calendarId,
          frDatetime: this.convDatetime(
            this.scheduleDialog.frDate,
            this.scheduleDialog.frTime
          ),
          toDatetime: this.convDatetime(
            this.scheduleDialog.toDate,
            this.scheduleDialog.toTime
          ),
          title: this.scheduleDialog.title
        }
        switch (this.scheduleDialog.mode) {
          case 1:
            mutation = apolloCreateSchedule
            break
          case 2:
            mutation = apolloModifySchedule
            variables.scheduleId = this.scheduleDialog.scheduleId
            break
          default:
            return // noop
        }
        const res = await this.$apollo.mutate({
          mutation,
          variables
        })
        switch (this.scheduleDialog.mode) {
          case 1:
            this.calendar.events.push(
              this.toCalendarEvent(res.data.createSchedule))
            break
          case 2:
            Object.assign(
              this.scheduleDialog.calendarEvent,
              this.toCalendarEvent(res.data.modifySchedule)
            )
            break
          default:
            return // noop
        }
        this.scheduleDialog.mode = null
        this.scheduleDialog.show = false
      } catch (e) {
        this.snackbar.text = 'スケジュールの保存に失敗しました。'
        this.snackbar.show = true
        // window.console.log(e)
      }
    },
    async deleteSchedule () {
      try {
        // FIXME: 確認ダイアログ
        await this.$apollo.mutate({
          mutation: apolloDeleteSchedule,
          variables: {
            calendarId: this.scheduleDialog.calendarId,
            scheduleId: this.scheduleDialog.scheduleId
          }
        })
        this.calendar.events.splice(
          this.calendar.events.indexOf(this.scheduleDialog.calendarEvent), 1)
        this.scheduleDialog.mode = null
        this.scheduleDialog.show = false
      } catch (e) {
        this.snackbar.text = 'スケジュールの削除に失敗しました。'
        this.snackbar.show = true
        // window.console.log(e)
      }
    },
    showDailySchedule ({ date }) {
      // FIXME: 一旦デイリー表示は保留
      // this.calendar.focus = date
      // this.calendar.type = 'day'
    },
    // ---------------------------------------------------------------------
    getEventColor (event) {
      return event.color
    }
  }
}
</script>
