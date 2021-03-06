<template>
  <v-card height="600">
    <!-- 顶部工具条 -->
    <v-card-text class="text--accent-1">
      <v-row class="justify-center align-center">
        <v-btn
          text
          class="text--darken-1"
          @click="$router.push('/arrangement_manage')"
          >< 返回</v-btn
        >
        <span class="mx-4">{{ name }} 的月排班</span>
        <v-spacer></v-spacer>
        <v-btn class="mx-1" fab text small color="grey darken-2" @click="prev">
          <v-icon small>mdi-chevron-left</v-icon>
        </v-btn>
        <v-btn class="mx-1" fab text small color="grey darken-2" @click="next">
          <v-icon small>mdi-chevron-right</v-icon>
        </v-btn>
        <v-btn outlined class="mx-2" color="grey darken-2" @click="setToday">
          今日
        </v-btn>
      </v-row>
    </v-card-text>

    <!-- 日历主体 -->
    <v-calendar
      ref="calendar"
      type="month"
      :events="events"
      v-model="focus"
      :now="today"
      :event-color="getEventColor"
      color="primary"
      @click:event="showEvent"
      @click:more="showMore"
      @change="updateEvent"
    ></v-calendar>

    <!-- 日历活动菜单 -->
    <v-menu
      v-model="selectedOpen"
      :close-on-content-click="false"
      :activator="selectedElement"
      offset-x
    >
      <v-card min-width="350px" flat>
        <v-toolbar :color="selectedEvent.color">
          <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
          <v-spacer></v-spacer>

          <v-btn icon @click="edit">
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

    <!-- 日历更多活动对话框 -->
    <v-dialog v-model="moreOpen" max-width="800px">
      <v-card min-width="350px" flat>
        <v-toolbar color="cyan">
          <v-toolbar-title>{{ moreDate }}</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn class="mx-1" icon @click="edit">
            <v-icon color="white">mdi-pencil</v-icon>
          </v-btn>
          <v-btn class="mx-1" icon @click="moreOpen = false">
            <v-icon color="white">mdi-close</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text>
          <v-list two-line>
            <v-list-item v-for="(item, i) in moreEvents" :key="i">
              <v-list-item-content>
                <v-list-item-title v-text="item.name"></v-list-item-title>
                <v-list-item-subtitle
                  >{{ item.start }} - {{ item.end }}
                </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
export default {
  name: "EmployeeArrangement",
  data() {
    return {
      name: "",
      uId: null,
      focus: null,
      today: null,
      events: [],

      selectedEvent: {},
      selectedElement: null,
      selectedOpen: false,

      moreEvents: [],
      moreDate: null,
      moreOpen: false
    };
  },
  created() {
    this.uId = this.$route.params.uId ? this.$route.params.uId : null;
    this.today = new Date().toISOString().substr(0, 10);
    this.focus = this.today;
  },
  mounted() {
    if (!this.uId) {
      alert("员工不存在！");
      this.$route.go(-1);
    }
    this.$refs.calendar.checkChange();
  },
  methods: {
    // 获取活动
    async getEvents() {
      try {
        let result = await this.$http.get("/schedule/view_month_employee.do", {
          uId: this.uId,
          day: this.focus
        });
        this.name = result.data.data.name;
        this.events = result.data.data.events;
      } catch (err) {
        console.error(err);
        let errMsg = err.data ? err.data.msg : "服务器错误，请稍后再试";
        alert(errMsg);
        this.$router.go(-1);
        this.events = [];
      }
    },
    // 获取当前活动颜色
    getEventColor(event) {
      return event.color;
    },
    // 活动菜单 - 显示
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
    // 日历更新
    updateEvent() {
      this.getEvents();
    },
    // 更多活动对话框 - 显示
    showMore(dateEvent) {
      let date = dateEvent.date;
      const open = () => {
        this.getEventsByDate(date);
        this.moreDate = date;
        setTimeout(() => (this.moreOpen = true), 10);
      };
      if (this.moreOpen) {
        this.moreOpen = false;
        setTimeout(open, 10);
      } else {
        open();
      }
    },
    // 以日期获取当日所有活动
    getEventsByDate(date) {
      let events = [];
      for (let event of this.events) {
        if (
          event.start.substr(0, 10) <= date &&
          date <= event.end.substr(0, 10)
        )
          events.push(event);
      }
      this.moreEvents = events;
    },
    // 上一月
    prev() {
      this.$refs.calendar.prev();
    },
    // 下一月
    next() {
      this.$refs.calendar.next();
    },
    // 跳转至今日
    setToday() {
      this.focus = this.today;
    },
    // 编辑员工排班
    edit() {
      this.$router.push("/arrangement_manage/employee/" + this.uId);
    }
  }
};
</script>

<style scoped></style>
