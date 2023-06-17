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

      <div
        class="calendar-weekly"
        v-for="(week, index) in calender"
        :key="index"
      >
        <div
          class="calendar-daily"
          :class="{ other: currentMonth !== day.month }"
          v-for="(day, index) in week"
          :key="index"
        >
          <div class="calendar-day">{{ day.day }}</div>

          <div v-for="dayEvent in day.dayEvents" :key="dayEvent.id">
            <div
              class="calendar-event"
              :style="`--event-bg:${dayEvent.color};width:${dayEvent.width}%`"
              draggable="true"
            >
              {{ dayEvent.name }}
            </div>
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
    // getDayEvents(date) {
    //   return this.events.filter((event) => {
    //     let startDate = moment(event.start).format("YYYY-MM-DD");
    //     let endDate = moment(event.end).format("YYYY-MM-DD");
    //     let Date = date.format("YYYY-MM-DD");
    //     if (startDate <= Date && endDate >= Date) {
    //       return true;
    //     }
    //   });
    // },
    getDayEvents(date, day) {
      let dayEvents = [];
      this.sortedEvents().forEach((event) => {
        let startDate = moment(event.start).format("YYYY-MM-DD");
        let endDate = moment(event.end).format("YYYY-MM-DD");
        let Date = date.format("YYYY-MM-DD");
        if (startDate <= Date && endDate >= Date) {
          if (startDate === Date) {
            let width = this.getEventWidth(startDate, endDate, day);
            dayEvents.push({ ...event, width });
          } else if (day === 0) {
            let width = this.getEventWidth(date, endDate, day);
            dayEvents.push({ ...event, width });
          }
        }
      });
      return dayEvents;
    },
    getEventWidth(end, start, day) {
      console.log({ end, start, day });
      let betweenDays = moment(start).diff(moment(end), "days");
      console.log(betweenDays);
      if (betweenDays > 6 - day) {
        return (6 - day) * 100 + 95;
      } else {
        return betweenDays * 100 + 90;
      }
    },
    sortedEvents() {
      return this.events.slice().sort(function (a, b) {
        let startDate = moment(a.start).format("YYYY-MM-DD");
        let startDate_2 = moment(b.start).format("YYYY-MM-DD");
        if (startDate < startDate_2) {
          return -1;
        }
        if (startDate > startDate_2) {
          return 1;
        }
        return 0;
      });
    },
  },
  mounted() {
    console.log(this.getCalendar());
  },
};
</script>

<style lang="scss" scoped>
.main {
  --border-color: #000;
  margin: 2em auto;
  width: 900px;
}

.calendar-month {
  text-align: center;
  font-size: 2rem;
}

.button-wrap {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 1rem 0;
}
.button {
  padding: 0.5rem 1rem;
}

.calendar {
  max-width: 900px;
  border-top: 1px solid var(--border-color);
  font-size: 0.8em;
}
.calendar-weekly {
  display: flex;
  border-left: 1px solid var(--border-color);
}
.calendar-headLine {
  flex: 1;
  border-right: 1px solid var(--border-color);
  border-bottom: 1px solid var(--border-color);
  margin-right: -1px;
  text-align: center;
  font-weight: bold;
  font-size: 1.3rem;
  padding: 0.5rem;
}
.calendar-daily {
  flex: 1;
  min-height: 125px;
  border-right: 1px solid var(--border-color);
  border-bottom: 1px solid var(--border-color);
  margin-right: -1px;
  &.other {
    background-color: #f7f7f7;
  }
}
.calendar-day {
  text-align: center;
  font-weight: bold;
  font-size: 1.2rem;
}
.calendar-event {
  color: #fff;
  padding: 0.3rem;
  background-color: var(--event-bg);
}
</style>
