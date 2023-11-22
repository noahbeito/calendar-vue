<template>
  <!-- *************** Calendar *************** -->
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
            <td v-for="dayInfo in week" @click="handleDayClick(dayInfo)" :key="dayInfo.dayNum + dayInfo.class" :class="dayInfo.class">
              {{ dayInfo.dayNum }}
              <span v-if="hasEvents(dayInfo)" class="event-dot"></span>
            </td>
          </tr>
        </tbody>
      </table>
   </div>
   <!-- *************** Modal *************** -->
   <transition name="modal">
        <div v-if="showModal" class="modal-mask">
          <div class="modal-wrapper">
            <div class="modal-container">
              <div class="modal-header">
                <slot name="header">
                  <h2>{{ selectedDate.toDateString() }}</h2>
                  <button class="modal-default-button" @click="closeModal">
                    &times;
                  </button>
                </slot>
              </div>

              <div class="modal-body">
                <slot name="body">
                  <ul v-if="eventsForSelectedDate.length">
                    <li v-for="(event, index) in eventsForSelectedDate" :key="index">
                      {{ event.time }} - {{ event.description }}
                    </li>
                  </ul>
                </slot>
              </div>

              <div class="modal-footer">
                <slot name="footer">
                  <input type="time" v-model="newEventTime"/>
                  <textarea v-model="newEventDescription"></textarea>
                  <button @click="addEvent" class="add-event-button">Add Event</button>
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
      selectedDate: null,
      showModal: false,
      events: {},
      newEventTime: '',
      newEventDescription: '',
    }),
    computed: {
      eventsForSelectedDate() {
        if (this.selectedDate) {
          const dateString = this.selectedDate.toISOString().split('T')[0];
          const events = this.events[dateString] || [];

          const sortedEvents = events.sort((a, b) => {
            const timeA = a.time.toLowerCase();
            const timeB = b.time.toLowerCase();
            return timeA.localeCompare(timeB);
          });

          const formattedTimeEvents = sortedEvents.map((item) => {
            return {
              ...item,
              // using dummy date of 2000-01-01, only need to format the time
              time: new Date(`2000-01-01T${item.time}`).toLocaleTimeString('en-US', {
                hour: 'numeric',
                minute: '2-digit',
              }),
            };
          });
          return formattedTimeEvents;
        }
        return [];
      }
    },
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
      handleDayClick(dayInfo) {
        this.selectedDate = new Date(this.currentYear, this.currentMonth, dayInfo.dayNum);
        this.showModal = true;
      },
      addEvent() {
        if (this.selectedDate && this.newEventTime && this.newEventDescription) {
          const dateString = this.selectedDate.toISOString().split('T')[0];

          const newEvent = {
            time: this.newEventTime,
            description: this.newEventDescription,
          };

          if (!this.events[dateString]) {
            this.events[dateString] = [newEvent];
          } else {
            this.events[dateString].push(newEvent);
          }
        }

        this.newEventTime = '';
        this.newEventDescription = '';
        this.closeModal();
      },
      hasEvents(dayInfo) {
        if (dayInfo.class.includes('current-month')) {
          const dateString = new Date(this.currentYear, this.currentMonth, dayInfo.dayNum).toISOString().split('T')[0];
          return !!(this.events[dateString] && this.events[dateString].length > 0);
        }
        return false;
      },
      closeModal() {
        this.showModal = false;
        this.selectedDate = null;
      }
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

  td {
    position: relative;
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
    top: 70%;
    left: 50%;
    transform: translateX(-50%);
    width: 8px;
    height: 8px;
    background-color: #4285f4;
    border: 1px solid white;
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
    position: relative;
    width: 300px;
    margin: 0px auto;
    padding: 10px 20px;
    background-color: #fff;
    border-radius: 2px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
    transition: all 0.3s ease;
    color: black;
  }

  .modal-header {
    margin-top: 0;
  }

  .modal-body {
    margin: 20px 0;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  .modal-default-button, .add-event-button {
    background: none;
    border: none;
    cursor: pointer;
  }

  .modal-default-button {
    position: absolute;
    top: 10px;
    right: 10px;
    font-size: 1.4em;
  }
  button.add-event-button {
    width: 100%;
    padding: 12px;
    background-color: #1b72e8;
    color: #fff;
    border: none;
    border-radius: 4px;
    font-size: 1.2em;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  button.add-event-button:hover {
    background-color: #1558c7;
  }

/* Time Selector */
input[type="time"] {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 1em;
  margin-bottom: 10px;
}

/* Text Input */
textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 1em;
  resize: vertical;
  margin-bottom: 10px;
}

input[type="time"]:focus,
textarea:focus {
  border-color: #1b72e8;
  box-shadow: 0 0 8px rgba(27, 114, 232, 0.5);
  outline: none;
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
