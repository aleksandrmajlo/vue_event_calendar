<template>
  <div>

    <div class="s-calendar__head">
         <a class="s-calendar__arrow s-calendar__arrow--left" @click.prevent="preMonth" href="#">
              <svg class="icon icon-chevron-left ">
                  <use xlink:href="/img/svg/sprite.svg#chevron-left"></use>
              </svg>
          </a>
          <div class="s-calendar__month">{{curYearMonth}}</div>
          <a @click.prevent="nextMonth" class="s-calendar__arrow s-calendar__arrow--right" href="#">
                  <svg class="icon icon-chevron-right ">
                    <use xlink:href="/img/svg/sprite.svg#chevron-right"></use>
                  </svg>
          </a>
    </div>
    <div class="cal-body s-calendar__body s-calendar__body--main">
      <div class="weeks">
        <span
          v-for="(dayName, dayIndex) in i18n[calendar.options.locale].dayNames"
          class="item"
          :key="dayIndex"
          >
          {{i18n[calendar.options.locale].dayNames[(dayIndex + calendar.options.weekStartOn) % 7]}}
        </span>
      </div>

      <div class="dates" >
        <div v-for="date in dayList" class="item"
          :class="[{
            today: date.status ? (today == date.date) : false,
            event: date.status ? (date.title != undefined) : false,
            [calendar.options.className] : (date.date == selectedDay)
          }, ...date.customClass]"
          :style={backgroundColor:date.color}
          @click="handleChangeCurday(date)"
          :key="date.date"
          >
          <span class="date-num " >
            {{date.status ? date.date.split('/')[2] : '&nbsp;'}}
          </span>
          <!--
          <span v-if="date.status ? (today == date.date) : false" class="is-today"  >
          </span>
          <span v-if="date.status ? (date.title != undefined) : false" class="is-event"
            :style="{borderColor: customColor, backgroundColor: (date.date == selectedDay) ? customColor : 'inherit'}">
          </span>
          -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import i18n from '../i18n.js'
import { dateTimeFormatter, isEqualDateStr} from '../tools.js'

const inBrowser = typeof window !== 'undefined'
export default {
  name: 'cal-panel',
  data () {
    return {
      i18n
    }
  },
  props: {
    events: {
      type: Array,
      required: true
    },
    calendar: {
      type: Object,
      required: true
    },
    selectedDay: {
      type: String,
      required: false
    }
  },
  computed: {
    dayList () {
      let firstDay = new Date(this.calendar.params.curYear, this.calendar.params.curMonth, 1)
      let dayOfWeek = firstDay.getDay()
      if (this.calendar.options.weekStartOn > dayOfWeek) {
        dayOfWeek = dayOfWeek - this.calendar.options.weekStartOn + 7
      } else {
        dayOfWeek = dayOfWeek - this.calendar.options.weekStartOn
      }

      let startDate = new Date(firstDay)
      startDate.setDate(firstDay.getDate() - dayOfWeek)

      let item, status, tempArr = [], tempItem
      for (let i = 0 ; i < 42 ; i++) {
          item = new Date(startDate);
          item.setDate(startDate.getDate() + i);

          if (this.calendar.params.curMonth === item.getMonth()) {
            status = 1
          } else {
            status = 0
          }
          tempItem = {
            date: `${item.getFullYear()}/${item.getMonth()+1}/${item.getDate()}`,
            status: status,
            customClass: [],
            color:'#7c7c7c'
          }
          this.events.forEach((event) => {
            if (isEqualDateStr(event.date, tempItem.date)) {
              tempItem.title = event.title
              tempItem.desc = event.desc || ''
              if (event.color&&tempItem.status==1) tempItem.color=event.color;
              if (event.customClass) tempItem.customClass.push(event.customClass);

            }
          })
          tempArr.push(tempItem)
      }
      return tempArr
    },
    today () {
      let dateObj = new Date()
      return `${dateObj.getFullYear()}/${dateObj.getMonth()+1}/${dateObj.getDate()}`
    },
    curYearMonth () {
      var arr=[
         'Январь',
         'Февраль',
         'Март',
         'Апрель',
          'Май',
         'Июнь',
         'Июль',
         'Август',
         'Сентябрь',
         'Октябрь',
         'Ноябрь',
         'Декабрь',
      ];
      let tempDate = Date.parse(new Date(`${this.calendar.params.curYear}/${this.calendar.params.curMonth+1}/01`));
      let date = new Date(tempDate ),
          month=date.getMonth(),
          year=date.getFullYear();
      return arr[month]+' '+  year;
      // return dateTimeFormatter(tempDate, this.i18n[this.calendar.options.locale].format)
    },
    customColor () {
      return this.calendar.options.color
    }
  },
  methods: {
    nextMonth () {
      this.$EventCalendar.nextMonth()
      this.$emit('month-changed', this.curYearMonth)
    },
    preMonth () {
      this.$EventCalendar.preMonth()
      this.$emit('month-changed', this.curYearMonth)
    },
    handleChangeCurday (date) {
      if (date.status) {
        this.$emit('cur-day-changed', date.date)
      }
    }
  }
}
</script>
