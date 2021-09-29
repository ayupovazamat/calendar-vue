<template>
  <div class="calendar">
    <div class="calendar__nav">
      <span @click="switch_month('prev')" class="calendar__nav--button"></span>
      <span class="calendar__nav--month">{{month}}</span>
      <span @click="switch_month('next')" class="calendar__nav--button"></span>
    </div>
    <div class="calendar__module">
      <div class="calendar__days">
        <div>Понедельник</div>
        <div>Вторник</div>
        <div>Среда</div>
        <div>Четверг</div>
        <div>Пятница</div>
        <div>Суббота</div>
        <div>Воскресенье</div>
      </div>
      <hr>
      <div class="calendar__items">
        <div v-for="(day) in dates" :key="day.id" :class="{ today: today(day.date), past: pastDay(day.date)}"
             class="calendar__items--card">
          <span class="calendar__items--card__number"
                :class="{weekend: weekends(day.id)}">{{ day.date.format('D') }}</span>
          <div v-if="day.events" class="calendar__items--card__events">
            <div class="calendar__items--card__event" v-for="(event) in day.events" :key="event.id">
              <span :class="'calendar__items--card__event-' + event.type">{{event.eventTime}} {{event.name}}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  import moment from 'moment';

  export default {
    name: 'calendar',
    props: {
      'events': []
    },
    data() {
      return {
        days: [],
        now: '',
        selectMonthYear: [],
        types: ['green', 'red', 'orange'],
        dateEvents: [],
      }
    },
    computed: {
      month() { // вывод месяца в навигации
        if (this.now === '') this.now = moment().locale('ru')
        let str = ''
        str = moment(this.now, 'DD-MM-YYYY').format('MMMM')
        if (this.now.format('YYYY') !== '2021') {
          str += ' ' + this.now.format('YYYY')
        }
        return str
      },
      dates() {
        return this.getDates(this.now)
      }
    },
    mounted() {
      this.now = moment().locale('ru')
    },
    methods: {
      today(day) {
        return true//( moment(this.now.format('YYYY.MM.DD')).isSame(day.format('YYYY.MM.DD')) )
      },
      pastDay(day) {
        return moment(day).isSameOrBefore()
      },
      weekends(index) {
        return ((index + 1) % 7 === 0 || (index + 2) % 7 === 0)
      },
      switch_month(direction) { // переключатель месяца / года
        switch (direction) {
          case 'prev':
            this.now = moment(this.now).subtract(1, 'month');
            break
          case 'next':
            this.now = moment(this.now).add(1, 'month');
            break
          default:
            break
        }
      },
      getDates(now) {
        let monthDate = moment(now).startOf('month')
        let monthDateEnd = moment(now).endOf('month') // последний день месяца

        let days = []

        // добавляем массив дат этого месяца
        days = [...Array(monthDate.daysInMonth())].map((_, i) => {
          return monthDate.clone().add(i, 'day')
        })

        // добавляем недостающие дни предыдущего месяца в начало массива
        for (let i = 0; i > 0 - (monthDate.isoWeekday() - 1); i--) {
          days.unshift(moment(this.now).date(i))
        }

        // добавляем недостающие дни след. месяца в конец массива
        for (let i = 0; i < (7 - monthDateEnd.isoWeekday()); i++) {
          days.push(moment(this.now).add(1, 'months').date(i + 1))
        }

        return this.addEvensDate(days)
      },
      addEvensDate(days) {
        let dateEventsArr = days.map((el, i) => {
             return {'id': i, 'date': moment(el)}
           }
        )

        // проходимся по событиям
        this.events.forEach((item) => {
          let dateEvent = moment(item['date']).format('YYYY.MM.DD')

          // ищем в массиве совпадение по дате
          let indexDate = days.findIndex((element, index) => {
            if (moment(element).isSame(dateEvent)) {
              return index
            }
          })

          // если есть совпадение
          if (indexDate !== -1) {
            if (dateEventsArr[indexDate]['events'] && dateEventsArr[indexDate]['events'].length !== 0) {
              item['eventTime'] = moment(item['date']).format('HH:mm')
              dateEventsArr[indexDate]['events'].push(item)
            } else {
              item['eventTime'] = moment(item['date']).format('HH:mm')
              dateEventsArr[indexDate]['events'] = [item]
              dateEventsArr[indexDate]['date'] = days[indexDate]
            }
          }
        })
        return dateEventsArr
      },
    }
  }
</script>
<style lang="stylus">
  $bg ?= #00000021
  $green ?= green
  $red ?= red
  $orange ?= orange
  $purple ?= purple
  $light_grey ?= #f9f9f9
  $grey ?= #ccc
  body
    background $bg

    .calendar
      background-color white
      max-width 550px

      &__nav
        display flex
        align-items center
        justify-content left
        padding 10px 20px

        &--month
          width 120px
          text-align center

        &--button
          cursor pointer
          position relative
          &:before
            content " "
            z-index 99
            position absolute
            width 0
            height 0
            top -7px
            &:hover
              opacity .7

          &:first-child
            &:before
              border-top 10px solid transparent
              border-bottom 10px solid transparent
              border-right 10px solid $green
          &:last-child
            &:before
              border-top 10px solid transparent
              border-bottom 10px solid transparent
              border-left 10px solid $green


          &:hover
            font-weight 700

      &__module
        padding 10px 20px

      &__days
        display grid
        grid-template-columns repeat(7, 1fr)
        width 100%

        > *
          align-items center
          display flex
          justify-content center

      &__items
        display grid
        grid-template-columns repeat(7, 1fr)
        width 100%
        gap 1px

        > div
          border 1px solid $light_grey

        > *::before
          content ""
          display inline-block
          height 0
          padding-bottom 80%
          width 1px

        > *.today
          color black
          border 0.1em solid $grey

          > span
            color $green

        &--card
          position relative
          border-radius 3px
          max-width 70px
          width 100%

          &.past
            background-color $light_grey
            font-weight 700

          &__number
            position absolute
            top 5px
            right 5px
            font-size .8rem
            color black

            &.weekend
              color: $purple

        &--card__events
          position absolute
          top 20px

          &:hover
            z-index 9

        &--card__event
          white-space: nowrap
          max-width: 70px;
          overflow: hidden;
          text-overflow: ellipsis;

          > span
            cursor default
            padding .2rem
            display block
            position absolute
            font-size .8rem

          &:hover
            width 100%
            overflow inherit
            z-index 9

          &-green
            background-color $green

          &-red
            background-color $red

          &-orange
            background-color $orange
</style>