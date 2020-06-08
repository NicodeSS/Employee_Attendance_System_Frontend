<template>
  <v-card height="600">
    <v-card-text class="text--accent-1">
      <v-row class="justify-center align-center">
        <span class="ml-4">{{ name }} 的周排班</span>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-btn text class="text--darken-1" @click="$router.go(-1)"
          >< 返回</v-btn
        >
      </v-row>
    </v-card-text>

    <v-calendar
      type="month"
      :events="events"
      :value="today"
      :now="today"
      :event-color="getEventColor"
      color="primary"
      @click:event="showEvent"
      @click:more="showList"
    ></v-calendar>
    <v-menu
      v-model="selectedOpen"
      :close-on-content-click="false"
      :activator="selectedElement"
      offset-x
    >
      <v-card color="grey lighten-4" min-width="350px" flat>
        <v-toolbar :color="selectedEvent.color" dark>
          <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
          <v-spacer></v-spacer>

          <v-btn icon>
            <v-icon>mdi-pencil</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text>
          <span
            >开始: <strong>{{ selectedEvent.start }}</strong></span
          >
          <v-divider></v-divider>
          <span v-if="selectedEvent.end"
            >结束: <strong>{{ selectedEvent.end }}</strong></span
          >
        </v-card-text>
      </v-card>
    </v-menu>
  </v-card>
</template>

<script>
export default {
  name: "EmployeeArrangement",
  uId: "",
  data() {
    return {
      name: "",
      today: "2020-01-01",
      events: [],

      selectedEvent: {},
      selectedElement: null,
      selectedOpen: false
    };
  },
  mounted() {
    this.uId = this.$route.params.uId;
    this.getArrangement();
  },
  methods: {
    getArrangement() {
      try {
        // let result = await this.$http.get("/schedule/view_week_employee.do", {
        //   uId: uId
        // });
        let result = this.getTestData();
        this.name = result.data.data.name;
        this.today = result.data.data.today;
        this.events = result.data.data.events;
      } catch (err) {
        console.log(err);
        this.events = [];
      }
    },
    getEventColor(event) {
      return event.color;
    },
    showEvent({ nativeEvent, event }) {
      const open = () => {
        this.selectedEvent = event;
        this.selectedElement = nativeEvent.target;
        setTimeout(() => (this.selectedOpen = true), 10);
      };

      if (this.selectedOpen) {
        this.selectedOpen = false;
        setTimeout(open, 10);
      } else {
        open();
      }

      nativeEvent.stopPropagation();
    },
    showList() {},
    getTestData() {
      return {
        code: 200,
        data: {
          status: 0,
          msg: "Success",
          data: {
            name: "test1",
            today: "2020-06-08",
            events: [
              {
                name: "Weekly Meeting",
                start: "2020-06-08 09:00",
                end: "2020-06-08 10:00",
                color: "cyan"
              },
              {
                name: "Thomas' Birthday",
                start: "2020-06-09",
                color: "blue"
              },
              {
                name: "Mash Potatoes",
                start: "2020-06-10 12:30",
                end: "2020-06-10 15:30",
                color: "indigo"
              }
            ]
          }
        }
      };
    }
  }
};
</script>

<style scoped></style>