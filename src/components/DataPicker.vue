<style scoped>
ul {
  list-style-type: none;
}

body {
  font-family: Verdana, sans-serif;
}

/* Month header */
.month {
  padding: 0 0;
  width: 100%;
  background: #1abc9c;
  text-align: center;
}

/* Month list */
.month ul {
  margin: 0;
  padding: 15px;
}

.month ul li {
  color: white;
  font-size: 20px;
  text-transform: uppercase;
  letter-spacing: 3px;
}

/* Previous button inside month header */
.month .prev {
  float: left;
  padding-top: 10px;
}

/* Next button */
.month .next {
  float: right;
  padding-top: 10px;
}

/* Weekdays (Mon-Sun) */
.weekdays {
  margin: 0;
  padding: 10px 0;
  background-color: #ddd;
}

.weekdays li {
  display: inline-block;
  width: 13.6%;
  color: #666;
  text-align: center;
}

/* Days (1-31) */
.days {
  padding: 10px 0;
  background: #eee;
  margin: 0;
}

.days li {
  list-style-type: none;
  display: inline-block;
  width: 13.6%;
  text-align: center;
  margin-bottom: 5px;
  font-size: 12px;
  color: #777;
}

/* Highlight the "current" day */
.days li .active {
  padding: 5px;
  background: #1abc9c;
  color: white !important
}

.days li .event {
  padding: 5px;
  background: #541ba5;
  border-radius: 30px;
  color: white !important
}

.date-picker {
  position: relative;
  width: 100%;
  max-width: 300px;
}

.date-picker__wrapper {
  position: absolute;
  width: 100%;
}

.date-picker__input {
  width: 100%;
}

.calendar-day {
  position: relative;

}

.calendar-day__wrapper {
  position: absolute;
  background: blanchedalmond;
  border-radius: 5px;
  width: 100%;
  height:100%;
  min-height: 20px;
  min-width: 100px;
  left: 30px;
  line-height: 20px;
  word-break: break-word;
  z-index: 999;
  display: none;
}

.calendar-day__show {
  display: block;
}
</style>
<script>
export default {
  props: {
    events: Array
  },
  created() {
    this.getToday();
  },
  data() {
    return {
      date: '',
      dateRegex: '^(3[01]|[12][0-9]|0[1-9]).(1[0-2]|0[1-9]).[0-9]{4}$',
      popupState: false,
      //calendar vars +
      today: null,
      currentMonthIndex: 0,
      currentYear: 2000,
      currentDay: 1,
      firstDayOfTheMonth: 1,
      dayInMonth: 28,
      months: [
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
      ],

      days: [],
      weekDaysName: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'],
      dayStruct: {},
      fullDate: '',
      //calendar vars -
      showEvent: false
    }
  },
  methods: {
    getFirstDayOfMonth(month, year) {
      return new Date(year, month, 1);
    },
    getLastDayOfMonth(month, year) {
      return new Date(year, month + 1, 0);
    },
    daysInMonth(month, year) {
      //because need non-index month
      return new Date(year, month + 1, 0).getDate();
    },
    getToday() {
      let today = new Date();
      this.currentMonthIndex = today.getMonth();
      this.currentYear = today.getFullYear();
      this.currentDay = today.getDate();
      this.fullDate = today.toLocaleDateString();
    },
    showCalendar() {
      this.popupState = !this.popupState;
    },
    fillDayArray(firstDay, monthIndex, year) {
      let arr = [];
      for (let i = firstDay; i <= this.daysInMonth(monthIndex, year); i++) {
        let id = ''.concat(i, monthIndex, year);
        let fullDate = new Date(year, monthIndex, i);
        arr.push({d: i, m: monthIndex, id: id, dateFormatted: fullDate.toLocaleDateString("ru-RU")});
      }
      return arr;
    },
    nexOrPreviousMonth(plus) {
      if (plus) {
        this.currentMonthIndex++;
      } else {
        this.currentMonthIndex--;
      }
      if (this.currentMonthIndex < 0) {
        this.currentMonthIndex = 11;
        this.currentYear = this.currentYear - 1;
      }
      if (this.currentMonthIndex > 11) {
        this.currentMonthIndex = 0;
        this.currentYear = this.currentYear + 1;
      }
    },
    chooseDate(chosenDate) {
      if (chosenDate === {}) {
        return '';
      }
      this.date = chosenDate.dateFormatted;
      this.popupState = false;
      this.$emit('getDate', this.date);
    },
    findEvents(dateArray) {
      for (let i = 0; i < this.events.length; i++) {
        let dateEventStruct = dateArray.find(dateStruct => dateStruct.dateFormatted === this.events[i].date);
        if (dateEventStruct) dateEventStruct.event = this.events[i].name;
      }
    }
  },
  computed: {
    currentFirstDayOfWeekInTheMonth() {
      return this.getFirstDayOfMonth(this.currentMonthIndex, this.currentYear).getDay();
    },
    currentLastDayOfWeekInTheMonth() {
      return this.getLastDayOfMonth(this.currentMonthIndex, this.currentYear).getDay();
    },
    showHidePopup() {
      return this.popupState ? '' : ' show'
    },
    validateDateSuccess() {
      if (this.date === '') {
        return true;
      }
      return this.date.match(this.dateRegex);
    },
    getDayArrayForCurrentMonth() {
      let dateArray = [];
      dateArray = (this.fillDayArray(1, this.currentMonthIndex, this.currentYear));
      this.findEvents(dateArray);
      for (let i = 1; i < this.currentFirstDayOfWeekInTheMonth || this.currentFirstDayOfWeekInTheMonth === 0 ? i < 7 : false; i++) { // 1 - because we start form monday
        dateArray.unshift(this.dayStruct);
      }
      for (let i = this.currentLastDayOfWeekInTheMonth; i <= 6 && i !== 0; i++) { // 6 - because we need last index of the week
        dateArray.push(this.dayStruct);
      }
      return dateArray;
    }
  }
}
</script>

<template>
  <div class="date-picker">
    <input
        class="date-picker__input"
        v-model="date"
        @click="showCalendar"
        maxlength="10"
    />
    <div v-show="!validateDateSuccess" style="color:red;">Ошибка ввода даты</div>
    <div class="date-picker__wrapper" v-show="!showHidePopup">
      <div class="month">
        <ul>
          <li @click="nexOrPreviousMonth(false)" class="prev">&#10094;</li>
          <li @click="nexOrPreviousMonth(true)" class="next">&#10095;</li>
          <li>{{ months[currentMonthIndex] }}<br><span style="font-size:18px">{{ currentYear }}</span></li>
        </ul>
      </div>

      <ul class="weekdays">
        <li v-for="weekDayName in weekDaysName" :key="weekDayName">
          {{ weekDayName }}
        </li>
      </ul>

      <ul class="days">
        <li class="calendar-day"
            v-for="(day, index) in getDayArrayForCurrentMonth"
            :key="index"
            @click="chooseDate(day)"
            @mouseenter="showEvent = day.id"
            @mouseleave="showEvent = false"
        >
          <span
              :class="{active: day.dateFormatted === fullDate, event: !!day.event}"
          >
            {{ day.d }}
          </span>
          <div
              v-if="!!day.event "
              class="calendar-day__wrapper"
              :class="{'calendar-day__show':showEvent===day.id}"
          >
            {{ day.event }}
          </div>
        </li>
      </ul>
    </div>
  </div>


</template>