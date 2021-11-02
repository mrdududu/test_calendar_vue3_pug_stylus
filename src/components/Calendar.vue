<template lang="pug">
.cal
  .cal-header
    .cal-header-nav(@click="setPrevMonth")
     i(class="arrow left")
    .cal-header-title {{ monthName }} {{ year }}
    .cal-header-nav(@click="setNextMonth")
      i(class="arrow right")
  table.cal-table
    thead
      tr
        th(v-for="(date, jdx) in calendar[0]" :key="`wnames_${jdx}`") {{ getWeekdayName(date) }}
    tbody
      tr(v-for="(week, idx) in calendar" :key="`week_${idx}`")
        td(
          v-for="(date, jdx) in week"
          :key="`date_${jdx}`"
          :class="{withmonth: 1 === date.getDate(), othermonth: month !== date.getMonth()}"
        )
          div(v-if="1 === date.getDate()" class="td-month") {{ getMonthName(date) }}
          div {{ date.getDate() }}
</template>

<script lang="ts">
import { defineComponent, ref, computed, PropType } from "vue";

export default defineComponent({
  props: {
    initialYear: {
      type: Number as PropType<number>,
      required: false,
      default() {
        return new Date().getFullYear();
      },
    } as const,
    initialMonth: {
      type: Number,
      required: false,
      default() {
        return new Date().getMonth();
      },
    } as const,
  },
  setup(props) {
    const year = ref<number>(props.initialYear);
    const month = ref<number>(props.initialMonth);

    const monthName = computed(() =>
      new Date(year.value, month.value, 1).toLocaleString("default", {
        month: "long",
      })
    );

    const daysInMonth = computed(() =>
      new Date(year.value, month.value + 1, 0).getDate()
    );

    const firstDay = computed(() => new Date(year.value, month.value, 1));

    const lastDay = computed(
      () => new Date(year.value, month.value, daysInMonth.value)
    );

    const firstWeekday = computed(() => firstDay.value.getDay());
    const lastWeekday = computed(() => lastDay.value.getDay());

    const calendar = computed(() => {
      const calGrid = [];
      let week = [];

      for (
        let i = 0 - firstWeekday.value;
        i < 6 - lastWeekday.value + daysInMonth.value;
        i++
      ) {
        const dayDate = new Date(
          firstDay.value.getTime() + 24 * 60 * 60 * 1000 * i
        );
        week.push(dayDate);

        if (6 === dayDate.getDay()) {
          calGrid.push(week);
          week = [];
        }
      }
      return calGrid;
    });

    const getWeekdayName = (date: Date) =>
      date.toLocaleDateString("default", { weekday: "short" });

    const getMonthName = (date: Date) =>
      date.toLocaleString("default", { month: "short" });

    const setNextMonth = () => {
      if (11 === month.value) {
        month.value = 0;
        year.value++;
        return;
      }

      month.value++;
    };

    const setPrevMonth = () => {
      if (0 === month.value) {
        month.value = 11;
        year.value--;
        return;
      }

      month.value--;
    };

    return {
      year,
      month,
      monthName,
      daysInMonth,
      firstDay,
      lastDay,
      firstWeekday,
      lastWeekday,
      calendar,
      getWeekdayName,
      getMonthName,
      setNextMonth,
      setPrevMonth,
    };
  },
});
</script>

<style lang="stylus" scoped>
@import "https://fonts.googleapis.com/css2?family=Roboto&display=swap"
.arrow
  border: solid #666;
  border-width: 0 2px 2px 0;
  display: inline-block;
  padding: 4px;

.right
  transform: rotate(-45deg);
  -webkit-transform: rotate(-45deg);

.left
  transform: rotate(135deg);
  -webkit-transform: rotate(135deg);

.cal
  font-family: "Roboto", sans-serif;
  font-size: 11px;
  color: #666;
  border: 1px solid #bbb;
  width: 428px;
  box-shadow: 0px 0px 4px #999;

.cal-header
  display: flex;
  height: 60px;
  justify-content: space-between;
  align-items: center;

.cal-header-nav
  width: 60px;
  height: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;

.cal-header-title
  font-size: 19px;

.cal-table
  width: 100%;
  border-collapse: collapse;

.cal-table th
  text-transform: uppercase;
  color: #aaa;

.cal-table thead
  border-bottom: 1px solid #ddd;

.cal-table td
  font-size: 14px;
  text-align: center;
  line-height: 14px;
  padding-top: 14px;
  padding-bottom: 14px;

.cal-table td.withmonth
  padding-top: 0px;

.cal-table td.othermonth
  background-color: #efefef;

.td-month
  text-transform: uppercase;
  color: #ddd;
  font-size: 9px;

.cal-table td.othermonth .td-month
  color: #666;
</style>
