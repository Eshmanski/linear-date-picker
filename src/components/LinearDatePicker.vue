<template>
  <div :class="$style['linear-date-picker']">
    <div :class="$style['header']">
      <div :class="$style['display']">
        <Transition :name="$style['fade-right']">
          <div v-if="!isClearPeriod && period[0]">
            {{ formatDate(period[0]) }}
          </div>
        </Transition>
      </div>
      <div :class="$style['center']">
        <div :class="$style['year']">{{ curMonth.year }}</div>
        <div :class="$style['month']">
          <q-select
            borderless
            :class="$style['selector']"
            :model-value="{
              value: curMonth.month,
              label: months[curMonth.month]
            }"
            :options="months.map((m, i) => ({ label: m, value: i }))"
            :option-disable="checkMonthOption"
            :popup-content-class="$style['popup-selector']"
            @update:model-value="selectMonth"
          />
        </div>
      </div>
      <div :class="$style['display']">
        <Transition :name="$style['fade-left']">
          <div v-if="!isClearPeriod && period[1]">
            {{ formatDate(period[1]) }}
          </div>
        </Transition>
      </div>
    </div>

    <div :class="$style['body']">
      <div :class="$style['btn-left']">
        <q-btn
          flat
          icon="keyboard_double_arrow_left"
          :disabled="
            isDisabled(days[0], 'day') && isDisabled(curMonth, 'month', true)
          "
          @click="() => move('back', 7)"
        ></q-btn>
        <q-btn
          flat
          icon="keyboard_arrow_left"
          :disabled="
            isDisabled(days[0], 'day') && isDisabled(curMonth, 'month', true)
          "
          @click="() => move('back', 1)"
        ></q-btn>
      </div>

      <div :style="{ overflow: 'hidden' }">
        <div ref="row" :class="$style['day-row']" @mouseleave="hover = null">
          <div
            v-for="day of days"
            :class="{
              [$style['day']]: true,
              [$style['weekend']]: day.weekDay > 4,
              [$style['hidden']]: !day.compare(curMonth, 'month'),
              [$style['hover']]: hover && hover.compare(day, 'day'),
              [$style['selected']]: isSelected(day),
              [$style['disabled']]: isDisabled(day, 'day'),
              [$style['in-range']]: isInRange(day)
            }"
            @mouseenter="hover = isDisabled(day, 'day') ? null : day"
            @click="() => (isDisabled(day, 'day') ? null : selectDay(day))"
            :key="day.date.getTime()"
          >
            <div :class="$style['week-day']">
              {{ dayOfWeek[day.weekDay] }}
            </div>

            <div :class="$style['num-day']">
              {{ day.day }}
            </div>
          </div>
        </div>
      </div>

      <div :class="$style['btn-right']">
        <q-btn
          flat
          icon="keyboard_arrow_right"
          :disabled="
            isDisabled(days[days.length - 1], 'day') &&
            isDisabled(curMonth, 'month', true)
          "
          @click="() => move('forward', 1)"
        ></q-btn>
        <q-btn
          flat
          icon="keyboard_double_arrow_right"
          :disabled="
            isDisabled(days[days.length - 1], 'day') &&
            isDisabled(curMonth, 'month', true)
          "
          @click="() => move('forward', 7)"
        ></q-btn>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {
  computed,
  defineComponent,
  onMounted,
  PropType,
  ref,
  Ref,
  watch
} from 'vue'
import { date } from 'quasar'
import { dayOfWeek, months } from 'src/models/main/date_lang.ts'

const toRange = (min: number, max: number, value: number) => {
  if (min > max) {
    const buf = min
    min = max
    max = buf
  }

  const count = max - min + 1
  const modulo = Math.abs((value - min) % count)

  if (value < min) return max + 1 - Math.abs(modulo)
  else if (value > max) return min + modulo
  else return value
}

class Day {
  date: Date

  constructor(ts?: number) {
    this.date = ts ? new Date(ts) : new Date()
  }

  get ts() {
    return this.date.getTime()
  }

  set year(y: number) {
    this.date.setFullYear(y)
  }

  get year() {
    return this.date.getFullYear()
  }

  set month(m: number) {
    this.date.setMonth(m)
  }

  get month() {
    return this.date.getMonth()
  }

  set day(d: number) {
    this.date.setDate(d)
  }

  get day() {
    return this.date.getDate()
  }

  get weekDay() {
    return toRange(0, 6, this.date.getDay() - 1)
  }

  get start() {
    return new Date(this.year, this.month, this.day, 0, 0, 0, 0).getTime()
  }

  get end() {
    return new Date(this.year, this.month, this.day, 23, 59, 59, 999).getTime()
  }

  clone() {
    return new Day(this.date.getTime())
  }

  getTS(type: 'year' | 'month' | 'day') {
    switch (type) {
      case 'year':
        return new Date(this.year, 0, 1).getTime()
      case 'month':
        return new Date(this.year, this.month, 1).getTime()
      case 'day':
        return new Date(this.year, this.month, this.day).getTime()
    }
  }

  compare(day: Day, type: 'year' | 'month' | 'day') {
    const ts_1 = this.getTS(type)
    const ts_2 = day.getTS(type)

    return ts_1 === ts_2
  }

  moveDay(count: number) {
    const day = this.date.getDate()
    this.day = day + count

    return this
  }

  moveMonth(count: number) {
    const month = this.date.getMonth()
    this.month = month + count

    return this
  }

  getFirstMonthDay() {
    return new Day(this.getTS('month'))
  }

  static getCursor(ts?: number) {
    const day = new Day(ts)

    day.moveDay(-day.weekDay)

    return day;
  }

  static getDateWithTime(dateTS: number, timeTS: number) {
    const date = new Date(dateTS);
    const time = new Date(timeTS);

    const year = date.getFullYear();
    const month = date.getMonth();
    const day = date.getDate();
    const hours = time.getHours();
    const minutes = time.getMinutes();
    const seconds = time.getSeconds();

    return new Date(year, month, day, hours, minutes, seconds);
  }
}

const week = 7

export default defineComponent({
  name: 'LinearDatePicker',
  props: {
    value: {
      type: Array as PropType<number[]>,
      required: true
    },
    weeks: {
      type: Number,
      default: () => 1
    },
    disabledMax: Date,
    disabledMin: Date,
    past: Boolean
  },
  setup(props, { emit }) {
    const cursor: Ref<Day> = ref(Day.getCursor())
    const curMonth: Ref<Day> = ref(cursor.value.getFirstMonthDay())
    const hover: Ref<Day | null> = ref(null)
    const period: Ref<number[]> = ref([])
    const isClearPeriod: Ref<boolean> = ref(false)
    const startSelecting: Ref<boolean> = ref(false)
    const showDays = computed(() => week * props.weeks)

    const days = computed(() => {
      const daysArr = [] as Day[]

      for (let i = 0; i < showDays.value; i++) {
        daysArr.push(cursor.value.clone().moveDay(i))
      }

      return daysArr
    })

    const checkMove = (type: 'forward' | 'back') => {
      const firstEl = cursor.value.clone()
      const lastEl = cursor.value.clone().moveDay(showDays.value)

      if (firstEl.compare(lastEl, 'month')) return false

      switch (type) {
        case 'forward':
          if (lastEl.compare(curMonth.value, 'month')) return false
          curMonth.value = lastEl.getFirstMonthDay()
          return true
        case 'back':
          if (firstEl.compare(curMonth.value, 'month')) return false
          curMonth.value = firstEl.getFirstMonthDay()
          return true
      }
    }

    const move = (type: 'forward' | 'back', step: number) => {
      const flag = checkMove(type)
      if (flag) return

      const cursorClone = cursor.value.clone()

      switch (type) {
        case 'forward':
          cursorClone.moveDay(step)
          break
        case 'back':
          cursorClone.moveDay(-step)
          break
      }

      if (cursor.value) cursor.value = cursorClone
    }

    const fix = (a: number, b: number) => {
      if (startSelecting.value) return
      isClearPeriod.value = true

      setTimeout(() => {
        if (startSelecting.value) return

        period.value = [b, a]
        isClearPeriod.value = false
      }, 500)
    }

    const selectDay = (day: Day) => {
      if (period.value.length === 2) period.value = []

      if (period.value.length === 1) {
        let a = period.value[0]!
        let b = day.end
        period.value = [a, b]

        if (a > b) {
          a = new Day(a).end
          b = new Day(b).start
          setTimeout(() => fix(a, b), 500)

          emit(
            'input',
            [
              Day.getDateWithTime(b, props.value[0]!),
              Day.getDateWithTime(a, props.value[1]!)
            ]
          );

          startSelecting.value = false;
        } else {
          emit(
            'input',
            [
              Day.getDateWithTime(a, props.value[0]!),
              Day.getDateWithTime(b, props.value[1]!)
            ]
          );

          startSelecting.value = false;
        }
      } else {
        period.value = [day.start]
        isClearPeriod.value = false
        startSelecting.value = true
      }
    }

    const selectMonth = (monthOption: { value: number; label: string }) => {
      const date = new Date(curMonth.value.year, monthOption.value)
      curMonth.value = new Day(date.getTime())
      cursor.value = new Day(date.getTime())
    }

    const isSelected = (day: Day) => {
      return period.value.some((ts) => {
        const periodDay = new Day(ts)
        return periodDay.compare(day, 'day')
      })
    }

    const isInRange = (day: Day) => {
      if (period.value.length === 2) {
        const time = day.date.getTime()
        const a = period.value[0]
        const b = period.value[1]

        if (a > b) {
          return a > time && b < time
        } else {
          return b > time && a < time
        }
      } else if (period.value.length === 1 && hover.value) {
        const time = day.date.getTime()
        const a = period.value[0]
        const b = hover.value.date.getTime()

        if (a > b) {
          return a > time && b < time
        } else {
          return b > time && a < time
        }
      } else {
        return false
      }
    }

    const formatDate = (ts: number) => {
      return date.formatDate(ts, 'YYYY.MM.DD')
    }

    const isDisabled = (
      day: Day | undefined,
      type: 'day' | 'month' | 'year',
      isInclude?: boolean
    ) => {
      if (day=== undefined) return false

      const point = day.getTS(type)
      const max = props.disabledMax
        ? new Day(props.disabledMax.getTime()).getTS(type)
        : Infinity
      const min = props.disabledMin
        ? new Day(props.disabledMin.getTime()).getTS(type)
        : -Infinity

      if (!isInclude) {
        return point > max || point < min
      } else {
        return point >= max || point <= min
      }
    }

    const checkMonthOption = (item: { value: number; label: string }) => {
      const day = curMonth.value.clone()
      day.month = item.value
      return isDisabled(day, 'month')
    }

    watch(
      () => props.value,
      (value) => {
        setTimeout(() => {
          if (startSelecting.value) return

          period.value = value
        }, 1000)
      }
    )

    onMounted(() => {
      const count = props.past ? week - showDays.value : 0

      cursor.value = Day.getCursor(props.value[0]).moveDay(count)
      curMonth.value = Day.getCursor(props.value[0]).getFirstMonthDay()
      period.value = props.value
    })

    return {
      days,
      hover,
      months,
      cursor,
      period,
      curMonth,
      dayOfWeek,
      isClearPeriod,
      move,
      selectDay,
      isInRange,
      isDisabled,
      formatDate,
      isSelected,
      selectMonth,
      checkMonthOption
    }
  }
})
</script>

<style module lang="scss">
.linear-date-picker {
  width: fit-content;

  // font: normal 400 12px 'UbuntuRegular'!important;
}

.popup-selector {
  max-height: 200px;

  & :global(.q-item) {
    min-height: auto;
    padding: none;
  }
}

.selector {
  width: 80px;

  & :global(.q-field__marginal) {
    height: auto;
  }

  & :global(.q-field__control),
  & :global(.q-field__native) {
    justify-content: center;
    min-height: auto;
  }
}

.header {
  display: flex;
  justify-content: center;
  align-items: center;

  & .year,
  & .month {
    font-size: 12px;
  }

  .month :global(.q-field) {
    font-size: 12px;
  }

  & .center {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-grow: 1;
    gap: 30px;
  }

  & .display {
    font-size: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100px;

    opacity: 0.5;
  }
}

.body {
  display: flex;
  align-items: center;

  & .day-row {
    display: flex;
    align-items: center;
    overflow: auto;

    & .day {
      display: flex;
      flex-direction: column;
      justify-content: center;
      cursor: pointer;

      box-sizing: border-box;
      width: 30px;
      height: 30px;
      background-color: rgb(252, 252, 252);
      border-left: 1px solid rgb(170, 170, 170);
      user-select: none;
    }

    & .day.weekend {
      background-color: rgb(211, 211, 211);
    }

    & .day.hidden {
      opacity: 0.2 !important;
    }

    & .day.disabled {
      opacity: 0.6;
      background-color: #e9b1b1c4 !important;
      cursor: not-allowed;
    }

    & .day.selected,
    & .day.hover {
      color: #ffffff !important;
      // background-color: #3863a3 !important;
      background: linear-gradient(to bottom right, #63e5c5, #14366f);
      // border: 1px solid #358b7d;
    }

    & .day.in-range {
      color: #ffffff !important;
      background: linear-gradient(to bottom right, #63e5c5, #14366f);
      // border: 1px solid #358b7d;

      // color: rgb(48, 170, 129);
      // background-color: #caf3e0;
    }

    & .num-day,
    & .week-day {
      display: flex;
      justify-content: center;
      align-items: center;
      font-weight: bold;
    }

    & .week-day {
      font-size: 8px;
      font-weight: normal;
    }

    & .num-day {
      font-size: 11px;
      font-weight: normal;
    }

    & .day:last-child {
      border-right: 1px solid rgb(121, 121, 121);
    }
  }

  & .day-row::-webkit-scrollbar {
    display: none;
  }

  & .btn-right,
  & .btn-left {
    & button {
      width: 30px;
    }
  }
}

.fade-left {
  &:global(-enter-active),
  &:global(-leave-active) {
    transition: all 0.3s ease;
  }

  &:global(-enter),
  &:global(-leave-to) {
    opacity: 0;
    transform: translateX(-20px);
  }
}

.fade-right {
  &:global(-enter-active),
  &:global(-leave-active) {
    transition: all 0.3s ease;
  }

  &:global(-enter),
  &:global(-leave-to) {
    opacity: 0;
    transform: translateX(20px);
  }
}
</style>
