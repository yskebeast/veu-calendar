<template>
  <div class="main">
    <h2 class="calendar-month">CALENDAR {{ displayMonth }}</h2>

    <div class="button-wrap">
      <button @click="lastMonth" class="button">LAST MONTH</button>
      <button @click="nextMonth" class="button">NEXT MONTH</button>
    </div>

    <div class="calendar">
      <div class="calendar-weekly">
        <div class="calendar-headLine" v-for="d in 7" :key="d">
          {{ getWeekOfDay(d - 1) }}
        </div>
      </div>

      <div class="calendar-weekly" v-for="(week, i) in calender" :key="i">
        <div class="calendar-daily" :class="{ other: currentMonth !== day.month }" v-for="(day, j) in week" :key="j">
          <div class="calendar-day">{{ day.day }}</div>
          <div v-for="(dayEvent, l) in day.dayEvents" :key="l">
            <div v-if="dayEvent.width" class="calendar-event" :style="`--event-bg:${dayEvent.color};width:${dayEvent.width}%`" draggable="true">
              {{ dayEvent.name }}
            </div>
            <div else style="height: 26px"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import events from "~/data/event";

export default {
  name: "Calendar",
  data() {
    return {
      currentDate: moment(),
    };
  },
  computed: {
    events: () => events,

    calender() {
      return this.getCalendar();
    },

    displayMonth() {
      return this.currentDate.format("YYYY[/]M");
    },

    currentMonth() {
      return this.currentDate.format("YYYY-MM");
    },

    sortedEvents() {
      return this.events.slice().sort(function (a, b) {
        let startDate = moment(a.start).format("YYYY-MM-DD");
        let startDate_2 = moment(b.start).format("YYYY-MM-DD");
        if (startDate < startDate_2) return -1;
        if (startDate > startDate_2) return 1;
        return 0;
      });
    },
  },
  methods: {
    getWeekOfDay(dayIndex) {
      const week = ["SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"];
      return week[dayIndex];
    },

    lastMonth() {
      this.currentDate = moment(this.currentDate).subtract(1, "month");
    },

    nextMonth() {
      this.currentDate = moment(this.currentDate).add(1, "month");
    },

    getStartDate() {
      let date = moment(this.currentDate);
      date.startOf("month");
      const weekNum = date.day();
      return date.subtract(weekNum, "days");
    },

    getEndDate() {
      let date = moment(this.currentDate);
      date.endOf("month");
      const weekNum = date.day();
      return date.add(6 - weekNum, "days");
    },

    getCalendar() {
      let calendarDate = this.getStartDate();
      const endDate = this.getEndDate();
      const weekNumber = Math.ceil(endDate.diff(calendarDate, "days") / 7);
      const calendars = [];
      for (let week = 0; week < weekNumber; week++) {
        const weekRow = [];
        for (let day = 0; day < 7; day++) {
          let dayEvents = this.getDayEvents(calendarDate, day);
          weekRow.push({
            day: calendarDate.get("date"),
            month: calendarDate.format("YYYY-MM"),
            dayEvents,
          });
          calendarDate.add(1, "days");
        }
        calendars.push(weekRow);
      }
      return calendars;
    },

    getEventWidth(end, start, day) {
      let betweenDays = moment(start).diff(moment(end), "days");
      if (betweenDays > 6 - day) {
        return (6 - day) * 100 + 95;
      } else {
        return betweenDays * 100 + 90;
      }
    },

    getDayEvents(date, day) {
      let stackIndex = 0;
      let dayEvents = [];
      let startedEvents = [];
      this.sortedEvents.forEach((event) => {
        let startDate = moment(event.start).format("YYYY-MM-DD");
        let endDate = moment(event.end).format("YYYY-MM-DD");
        let Date = date.format("YYYY-MM-DD");
        if (startDate <= Date && endDate >= Date) {
          if (startDate === Date) {
            [stackIndex, dayEvents] = this.getStackEvents(event, day, stackIndex, dayEvents, startedEvents, event.start);
          } else if (day === 0) {
            [stackIndex, dayEvents] = this.getStackEvents(event, day, stackIndex, dayEvents, startedEvents, Date);
          } else {
            startedEvents.push(event);
          }
        }
      });
      return dayEvents;
    },

    getStackEvents(event, day, stackIndex, dayEvents, startedEvents, start) {
      [stackIndex, dayEvents] = this.getStartedEvents(stackIndex, startedEvents, dayEvents);
      let width = this.getEventWidth(start, event.end, day);
      Object.assign(event, {
        stackIndex,
      });
      dayEvents.push({ ...event, width });
      stackIndex++;
      return [stackIndex, dayEvents];
    },

    getStartedEvents(stackIndex, startedEvents, dayEvents) {
      let startedEvent;
      do {
        startedEvent = startedEvents.find((event) => event.stackIndex === stackIndex);
        if (startedEvent) {
          dayEvents.push(startedEvent); //ダミー領域として利用するため
          stackIndex++;
        }
      } while (typeof startedEvent !== "undefined");
      return [stackIndex, dayEvents];
    },
  },
  mounted() {
    this.getCalendar();
  },
};
</script>

<style lang="scss" scoped>
@import "~/design/calendar";
</style>
