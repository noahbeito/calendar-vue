<template>
  <div class="calendar-wrapper">
    <div class="header-container">
      <div class="header-month-year">{{ getMonth() }} {{ currentYear }}</div>
      <div class="header-buttons">
        <button @click="handlePreviousMonth" class="left-button">&lt;</button>
        <button @click="handleNextMonth" class="right-button">&gt;</button>
        <button @click="handleToday" class="today-button">Today</button>
      </div>
    </div>
      <table>
        <thead>
          <tr>
            <th v-for="day in daysOfWeek" :key="day" class="days-row">{{ day }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(week, index) in renderDays()" :key="index">
            <td v-for="dayInfo in week" :key="dayInfo.dayNum + dayInfo.class" :class="dayInfo.class">
              {{ dayInfo.dayNum }}
            </td>
          </tr>
        </tbody>
      </table>
   </div>
</template>

<script>
  export default {
    data: () => ({
      daysOfWeek: ['S', 'M', 'T', 'W', 'T', 'F', 'S'],
      currentMonth: new Date().getMonth(),
      currentYear: new Date().getFullYear(),
    }),
    methods: {
      getMonth() {
        const months = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];
        return months[this.currentMonth];
      },
      handleNextMonth() {
        this.currentMonth = (this.currentMonth + 1) % 12;
        if (this.currentMonth === 0) {
          this.currentYear++;
        }
      },
      handlePreviousMonth() {
        this.currentMonth = (this.currentMonth - 1 + 12) % 12;
        if (this.currentMonth === 11) {
          this.currentYear--;
        }
      },
      handleToday() {
        this.currentMonth = new Date().getMonth();
        this.currentYear = new Date().getFullYear()
      },
      renderDays() {
        const firstDayOfMonthIndex = new Date(this.currentYear, this.currentMonth, 1).getDay();
        const daysInMonth = new Date(this.currentYear, this.currentMonth + 1, 0).getDate();
        let daysInPrevMonth = new Date(this.currentYear, this.currentMonth, 0).getDate();

        const weeksInMonth = Math.ceil((firstDayOfMonthIndex + daysInMonth) / 7);
        const totalListOfDays = [];
        const prevMonthDays = [];
        let prevMonthOverflowCount = firstDayOfMonthIndex;
        let day = 1;

        while (prevMonthOverflowCount > 0) {
          const prevMonthDayInfo = {
            dayNum: daysInPrevMonth,
            class: 'other-month'
          }
          prevMonthDays.push(prevMonthDayInfo);
          daysInPrevMonth--;
          prevMonthOverflowCount--;
        }

        prevMonthDays.reverse();

        totalListOfDays.push(...prevMonthDays);

        while (day <= daysInMonth) {
          const date = new Date(this.currentYear, this.currentMonth, day);
          const isCurrentMonth = day <= daysInMonth;
          const isCurrentDay = isCurrentMonth && date.toDateString() === new Date().toDateString();

          const dayInfo = {
            dayNum: day,
            class: isCurrentDay ? 'current-month current-day' : 'current-month',
          }
          totalListOfDays.push(dayInfo);
          day++;
        }

        let nextMonthOverflow = (weeksInMonth * 7) - totalListOfDays.length;
        let nextMonthDay = 1;

        while (nextMonthOverflow > 0) {
          const nextMonthDayInfo = {
            dayNum: nextMonthDay,
            class: 'other-month'
          }
          totalListOfDays.push(nextMonthDayInfo);
          nextMonthDay++;
          nextMonthOverflow--;
        }

        const calendarGrid = [];
        let count = 0;

        for (let week = 0; week < weeksInMonth; week++) {
          const weekArray = [];
          for (let day = 0; day < 7; day++) {
            weekArray.push(totalListOfDays[count])
            count++;
          }
          calendarGrid.push(weekArray);
        }
        return calendarGrid;
      },
    }
  }
</script>

<style scoped>

  body {
    font-family: 'Roboto', sans-serif;
  }

  .calendar-wrapper {
    width: 400px;
  }

  .header-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 15px;
  }

  .header-month-year {
    font-size: 1.4em;
    padding: 1px 6px;
  }

  .header-buttons {
    display: flex;
  }

  .left-button, .right-button, .today-button {
    background: none;
    border: none;
    padding: 6px;
    cursor: pointer;
  }

  .left-button, .right-button {
    width: 30px;
  }

  .right-button:hover, .left-button:hover {
    background-color: lightgray;
    border-radius: 50%;
  }

  .today-button:hover {
    color: #4285f4;
  }

  table {
    width: 100%;
    border-collapse: collapse;
  }

  th, td {
    box-sizing: border-box;
    text-align: center;
    width: calc(100% / 7);
    height: 60px;
  }

  .current-month:hover, .other-month:hover {
    background-color: lightblue;
    border-radius: 50%;
  }

  .current-month {
    color: black;
  }

  .other-month {
    color: grey;
  }

  .current-day {
    background-color: #1b72e8;
    color: white;
    border-radius: 50%;
  }
  .event-dot {
    display: block;
    position: absolute;
    width: 6px;
    height: 6px;
    left: 50%;
    transform: translate(-50%);
    background-color: #4285f4;
    border-radius: 50%;
  }

</style>
