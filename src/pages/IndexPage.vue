<template>
  <q-page>
    <div class="column">
      <div class="row justify-center">
        <h2>Meetings</h2>
      </div>
      <div class="row justify-center">
        <q-input outlined v-model="email" label="Email" />
      </div>
      <div class="row justify-center">
        <q-btn @click="registerUser" color="primary" label="Register user" />
      </div>

      <div class="subcontent">
        <navigation-bar @today="onToday" @prev="onPrev" @next="onNext" />

        <div class="row justify-center">
          <div
            style="display: flex; max-width: 800px; width: 100%; height: 400px"
          >
            <q-calendar-day
              ref="calendar"
              v-model="selectedDate"
              view="week"
              :hour24-format="true"
              :interval-minutes="30"
              :interval-count="48"
              :interval-height="30"
              time-clicks-clamped
              :selected-dates="selectedDates"
              animated
              bordered
              @click-time="onToggleTime"
              @change="onChange"
              @moved="onMoved"
              @click-date="onClickDate"
              @click-interval="onClickInterval"
              @click-head-intervals="onClickHeadIntervals"
              @click-head-day="onClickHeadDay"
            />
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import {
  QCalendarDay,
  today,
  copyTimestamp,
  getDateTime,
} from "@quasar/quasar-ui-qcalendar/src/index.js";
import "@quasar/quasar-ui-qcalendar/src/QCalendarVariables.sass";
import "@quasar/quasar-ui-qcalendar/src/QCalendarTransitions.sass";
import "@quasar/quasar-ui-qcalendar/src/QCalendarDay.sass";

import { defineComponent } from "vue";
import NavigationBar from "../components/NavigationBar.vue";

export default defineComponent({
  name: "IndexPage",
  components: {
    NavigationBar,
    QCalendarDay,
  },

  data() {
    return {
      email: "",
      selectedDate: today(),
      selectedDates: [],
    };
  },
  created() {
    fetch("http://localhost:8000/meetings/all")
      .then((response) => response.json())
      .then((meetings) => {
        console.log(meetings);
        meetings.forEach((meeting) => {
          let split = meeting.date.split("T");
          let time = split[1].split(":");
          let secondPart = `${time[0]}:${time[1]}`;
          let result = split[0] + " " + secondPart;

          this.selectedDates.push(result);
        });
      });
  },
  methods: {
    registerUser() {
      const createUser = {
        method: "POST",
        headers: {
          "Content-type": "application/json; charset=UTF-8",
        },
        body: JSON.stringify({
          email: this.email,
        }),
      };
      fetch("http://localhost:8000/users/create", createUser);
    },
    onToggleTime({ scope }) {
      console.log("click-time (scope)", scope);
      if (scope === undefined) {
        return;
      }

      const ts = copyTimestamp(scope.timestamp);
      const t = getDateTime(ts);
      console.log(t);

      if (this.selectedDates.includes(t)) {
        for (let i = 0; i < this.selectedDates.length; ++i) {
          if (t === this.selectedDates[i]) {
            const deleteMeeting = {
              method: "DELETE",
              headers: {
                "Content-type": "application/json; charset=UTF-8",
              },
              body: JSON.stringify({
                date: t,
                user_email: this.email,
              }),
            };
            fetch("http://localhost:8000/meetings/delete", deleteMeeting).then(
              (response) => {
                if (response.status == 200) {
                  this.selectedDates.splice(i, 1);
                }
              }
            );
          }
        }
      } else {
        // add the date if not outside
        if (scope.outside !== true) {
          const createMeeting = {
            method: "POST",
            headers: {
              "Content-type": "application/json; charset=UTF-8",
            },
            body: JSON.stringify({
              date: t,
              user_email: this.email,
            }),
          };
          fetch("http://localhost:8000/meetings/create", createMeeting).then(
            (response) => {
              if (response.status == 201) {
                this.selectedDates.push(t);
              }
            }
          );
          console.log(this.selectedDates);
        }
      }
    },

    onToday() {
      this.$refs.calendar.moveToToday();
    },
    onPrev() {
      this.$refs.calendar.prev();
    },
    onNext() {
      this.$refs.calendar.next();
    },
    onMoved(data) {
      console.log("onMoved", data);
    },
    onChange(data) {
      console.log("onChange", data);
    },
    onClickDate(data) {
      console.log("onClickDate", data);
    },
    // onClickTime (data) {
    //   console.log('onClickTime', data)
    // },
    onClickInterval(data) {
      console.log("onClickInterval", data);
    },
    onClickHeadIntervals(data) {
      console.log("onClickHeadIntervals", data);
    },
    onClickHeadDay(data) {
      console.log("onClickHeadDay", data);
    },
  },
});
</script>

<style src="@quasar/quasar-ui-qcalendar/dist/index.css"></style>
