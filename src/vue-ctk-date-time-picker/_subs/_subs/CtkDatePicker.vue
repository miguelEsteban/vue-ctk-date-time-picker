<template>
  <div id="CtkDatePicker" class="datepicker-container" :class="{'flex-1 inline': withoutInput}">
    <div class="datepicker-controls flex align-center justify-content-center">
      <div class="arrow-month h-100">
        <button type="button" tabindex="-1" class="datepicker-button datepicker-prev text-center h-100 flex align-center" @click="changeMonth('prev')">
          <svg viewBox="0 0 1000 1000"><path d="M336.2 274.5l-210.1 210h805.4c13 0 23 10 23 23s-10 23-23 23H126.1l210.1 210.1c11 11 11 21 0 32-5 5-10 7-16 7s-11-2-16-7l-249.1-249c-11-11-11-21 0-32l249.1-249.1c21-21.1 53 10.9 32 32z"></path></svg>
        </button>
      </div>
      <div class="datepicker-container-label flex-1">
        <transition-group :name="transitionLabelName" class="h-100 flex align-center justify-content-center">
          <div class="datepicker-label fs-16" v-for="month in [month]" :key="month.month" v-text="getMonthFormatted()"></div>
        </transition-group>
      </div>
      <div class="arrow-month h-100 text-right">
        <button type="button" tabindex="-1" class="datepicker-button datepicker-next text-center h-100 flex align-center justify-content-right" @click="changeMonth('next')">
          <svg viewBox="0 0 1000 1000"><path d="M694.4 242.4l249.1 249.1c11 11 11 21 0 32L694.4 772.7c-5 5-10 7-16 7s-11-2-16-7c-11-11-11-21 0-32l210.1-210.1H67.1c-13 0-23-10-23-23s10-23 23-23h805.4L662.4 274.5c-21-21.1 11-53.1 32-32.1z"></path></svg>
        </button>
      </div>
    </div>
    <div class="datepicker-week flex">
      <div v-for="(weekDay, index) in weekDays" :key="index" class="flex-1 text-muted fs-12 flex justify-content-center align-center">
        {{weekDay}}
      </div>
    </div>
    <div class="month-container" :style="{height: (monthDays.length + weekDay) > 35 ? '250px' : '210px'}">
      <transition-group :name="transitionDaysName">
        <div class="datepicker-days flex" v-for="month in [month]" :key="month.month">
          <div class="datepicker-day align-center justify-content-center"
               v-for="start in weekDay" :key="start + 'startEmptyDay'"></div>
          <button type="button" tabindex="-1"  class="datepicker-day flex align-center justify-content-center"
               v-for="day in monthDays" :key="day.format('D')"
               :class="{selected: isSelected(day) && value, disabled: (isDisabled(day) || isWeekEndDay(day)), enable: !(isDisabled(day) || isWeekEndDay(day))}"
               @click="isDisabled(day) || isWeekEndDay(day) ? '' : selectDate(day)">
            <span class="datepicker-day-effect" v-show="!isDisabled(day) || isSelected(day)" :style="bgStyle"></span>
            <span class="datepicker-day-text">{{day.format('D')}}</span>
          </button>
          <div class="datepicker-day flex align-center justify-content-center"
               v-if="endEmptyDays"
               v-for="end in endEmptyDays" :key="end  + 'endEmptyDay'"></div>
        </div>
      </transition-group>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import { getWeekDays } from './../../modules/month'
export default {
  name: 'CtkDatePicker',
  props: ['month', 'dateTime', 'color', 'minDate', 'maxDate', 'locale', 'withoutInput', 'noWeekendsDays', 'value'],
  data () {
    return {
      transitionDaysName: 'slidenext',
      transitionLabelName: 'slidevnext',
      weekDays: getWeekDays(this.locale)
    }
  },
  computed: {
    bgStyle () {
      return {
        backgroundColor: this.color
      }
    },
    endEmptyDays () {
      if ((this.monthDays.length + this.weekDay) > 35) {
        return 42 - this.monthDays.length - this.weekDay
      } else {
        return 35 - this.monthDays.length - this.weekDay
      }
    },
    monthDays () {
      return this.month.getMonthDays()
    },
    weekDay () {
      return this.month.getWeekStart()
    }
  },
  methods: {
    getMonthFormatted () {
      return this.month.getFormatted()
    },
    isDisabled (day) {
      if (this.minDate && this.maxDate) {
        return !moment(day).isBetween(this.minDate, this.maxDate)
      } else if (this.minDate) {
        return moment(day).isBefore(this.minDate)
      } else if (this.maxDate) {
        return moment(day).isAfter(this.maxDate)
      }
      return false
    },
    isSelected (day) {
      return moment(moment(this.dateTime).format('YYYY-MM-DD')).isSame(day.format('YYYY-MM-DD'))
    },
    isWeekEndDay (day) {
      const dayConst = moment(day).day()
      const weekendsDaysNumbers = [6, 0]
      return this.noWeekendsDays ? weekendsDaysNumbers.indexOf(dayConst) > -1 : false
    },
    selectDate (day) {
      this.$emit('change-date', day)
    },
    changeMonth (val) {
      this.transitionDaysName = 'slide' + val
      this.transitionLabelName = 'slidev' + val
      this.$emit('change-month', val)
    }
  }
}
</script>
<style lang="scss" scoped>
  @import "../../assets/animation.scss";
  #CtkDatePicker {
    width: 290px;
    padding: 0 5px;
    .datepicker-controls {
      height: 56px;
      .arrow-month {
        flex: 0 0 40px;
      }
      .datepicker-button {
        background: transparent;
        cursor: pointer;
        padding: 0 10px;
        border: none;
        outline: none;
        svg {
          height: 17px;
          width: 17px;
          fill: #2c3e50;
        }
        &.datepicker-prev {
          text-align: left !important;
        }
        &.datepicker-next {
          text-align: right !important;
        }
      }
      .datepicker-container-label {
        text-transform: capitalize;
        font-size: 18px;
        line-height: 18px;
        position: relative;
        height: 18px;
        overflow: hidden;
      }
      .datepicker-label {
        text-transform: capitalize;
      }
    }
    .datepicker-week {
      height: 41px;
      text-transform: capitalize;
      // border-bottom: 1px solid #eaeaea;
    }
    .month-container {
      position: relative;
      overflow: hidden;
    }
    .datepicker-days {
      display: flex;
      display: -ms-flexbox;
      overflow: hidden;
      flex-wrap: wrap;
      -ms-flex-wrap: wrap;
      .datepicker-day {
        height: 41px;
        flex-grow: 1;
        width: calc(100% / 7);
        position: relative;
        border: none;
        background: transparent;
        font-size: 13px;
        outline: none;
        &.enable {
          cursor: pointer;
        }
        .datepicker-day-effect {
          position: absolute;
          opacity: 0.6;
          background: dodgerblue;
          height: 30px;
          width: 30px;
          border-radius: 50%;
          -webkit-transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms;
          transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms;
          transform: scale(0);
        }
        .datepicker-day-text {
          position: relative;
        }
        &:hover {
          color: #FFF;
          .datepicker-day-effect {
            transform: scale(1);
            opacity: 0.6;
          }
        }

        &.selected {
          color: #FFF;
          .datepicker-day-effect {
            transform: scale(1);
            opacity: 1;
          }
        }
        &.disabled {
          color: #CCC;
          &.selected {
            color: #fff;
          }
          .datepicker-day-effect {
            transform: scale(0);
            opacity: 0;
          }
        }
      }
    }
  }
  @media screen and (max-width: 412px) {
    #CtkDatePicker {
      width: 100%;
      &:not(.inline) {
        .datepicker-controls {
          height: 36px !important;
        }
        .datepicker-week {
          height: 21px !important;
        }
      }
    }
  }
</style>
