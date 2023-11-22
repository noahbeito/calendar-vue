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
            <td v-for="dayInfo in week" @click="showModal = true" :key="dayInfo.dayNum + dayInfo.class" :class="dayInfo.class">
              {{ dayInfo.dayNum }}
            </td>
          </tr>
        </tbody>
      </table>
   </div>
   <transition name="modal">
        <div v-if="showModal" class="modal-mask">
          <div class="modal-wrapper">
            <div class="modal-container">

              <div class="modal-header">
                <slot name="header">
                  default header
                </slot>
              </div>

              <div class="modal-body">
                <slot name="body">
                  default body
                </slot>
              </div>

              <div class="modal-footer">
                <slot name="footer">
                  default footer
                  <button class="modal-default-button" @click="showModal = false">
                    OK
                  </button>
                </slot>
              </div>
            </div>
          </div>
        </div>
      </transition>
</template>

<script>
  export default {
    data: () => ({
      daysOfWeek: ['S', 'M', 'T', 'W', 'T', 'F', 'S'],
      currentMonth: new Date().getMonth(),
      currentYear: new Date().getFullYear(),
      showModal: true,
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

        // Add the days from the previous month to the total list of days to be displayed
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

        // Add the days from the current month to total list of days to be displayed
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

        // Add the days from next month to total list of days to be displayed
        while (nextMonthOverflow > 0) {
          const nextMonthDayInfo = {
            dayNum: nextMonthDay,
            class: 'other-month'
          }
          totalListOfDays.push(nextMonthDayInfo);
          nextMonthDay++;
          nextMonthOverflow--;
        }

        // Build the 2d array of days for the current month
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
    width: 420px;
    background-color: white;
    padding: 10px;
  }

  .header-container {
    display: flex;
    color: black;
    justify-content: space-between;
    align-items: center;
    padding: 0 15px;
  }

  .header-month-year {
    font-size: 1.4em;
    padding: 1px 6px;
    cursor: default;
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

  th {
    color: black;
  }

  th, td {
    box-sizing: border-box;
    text-align: center;
    width: calc(100% / 7);
    height: 60px;
    cursor: pointer;
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

  /* Modal Styling */

  .modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: table;
  transition: opacity 0.3s ease;
}

  .modal-wrapper {
    display: table-cell;
    vertical-align: middle;
  }

  .modal-container {
    width: 300px;
    margin: 0px auto;
    padding: 20px 30px;
    background-color: #fff;
    border-radius: 2px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
    transition: all 0.3s ease;
    color: black;
  }

  .modal-header h3 {
    margin-top: 0;
    color: #42b983;
  }

  .modal-body {
    margin: 20px 0;
  }

  .modal-default-button {
    float: right;
  }

  /* Modal Transitions */

  .modal-enter {
    opacity: 0;
  }

  .modal-leave-active {
    opacity: 0;
  }

  .modal-enter .modal-container,
  .modal-leave-active .modal-container {
    -webkit-transform: scale(1.1);
    transform: scale(1.1);
  }
</style>
