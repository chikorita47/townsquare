<template>
  <div class="timer">
    This is the Timer Module.
    <div v-if="!session.isSpectator">
      <span class="button" v-if="isStopped" @click="decrementTimer">-</span>
      <span class="button" v-if="isStopped" @click="incrementTimer">+</span>
      <!-- <span class="button" @click="setTimer">timer</span> -->
      <span class="button" v-if="!isRunning" @click="startTimer">play</span>
      <span class="button" v-if="isRunning" @click="pauseTimer">pause</span>
      <span class="button" @click="stopTimer">stop</span>
    </div>
    <span>{{ this.timerDuration }}</span>
    <span>{{ this.formattedTime }}</span>

    <span>{{ JSON.stringify(session.timer, undefined, 2) }}</span>
  </div>
</template>
<script>
import { mapState } from 'vuex';

const ONE_MINUTE = 60000;

export default {
  computed: {
    ...mapState(['timer', 'session']),
    formattedTime() {
      const time = this.isStopped ? this.timerDuration : this.timerDisplay;
      const minutes = parseInt(time / ONE_MINUTE);
      const seconds = Math.floor(Math.max(0, time % ONE_MINUTE) / 1000);
      const secondsStr = seconds > 9 ? seconds : `0${seconds}`;
      return `${minutes}:${secondsStr}`;
    },
    timestampEnd() {
      return this.isRunning ? this.timestampStart + this.timeRemaining : null;
    },
  },
  data() {
    const BASE_TIME = 5 * ONE_MINUTE;
    return {
      timestampStart: null,
      timestampPause: null,
      timerDuration: BASE_TIME,
      timerDisplay: 0,

      /** Time in ms remaining - calculated and sent to clients every play/pause, calculated every 200ms  */
      timeRemaining: 0,
      timerInterval: null,
      isRunning: false,
      isStopped: true,
    };
  },
  methods: {
    startTimer() {
      this.isStopped = false;
      this.isRunning = true;
      this.$store.commit('session/setTimer', {
        ...this.$store.state.session.timer,
        isStopped: false,
        isRunning: true,
      });

      // const timerParams = {
      //   timerStart: Math.round(Date.now() / 1000),
      //   isRunning: this.isRunning,
      //   timeRemaining: this.timeRemaining,
      // };
      // // send the timer data to clients (update this to include start timestamp, time remaining, and isRunning)
      // this.$store.commit('session/setTimer', timerParams);

      // if (!this.timerInterval) {
      //   this.timerInterval = setInterval(() => {
      //     if (this.timerDisplay > 0) {
      //       this.timerDisplay =
      //         this.timeRemaining - (Date.now() - this.timestampStart);
      //     } else {
      //       clearInterval(this.timerInterval);
      //     }

      //   }, 200);
      // }
    },
    stopTimer() {
      this.$store.commit('session/setTimer', {
        // ...this.$store.state.session.timer,
        timestampStart: null,
        timestampPause: null,
        timeRemaining: 0,
        isStopped: true,
        isRunning: false,
      });
      // this.isStopped = true;
      // this.isRunning = false;
      clearInterval(this.timerInterval);
      this.timerInterval = null;
      this.timeRemaining = 0;
      this.timerDisplay = 0;
      this.timestampStart = null;
      this.timestampPause = null;
    },
    pauseTimer() {
      this.isRunning = false;

      this.$store.commit('session/setTimer', {
        ...this.$store.state.session.timer,
        isRunning: false,
      });
    },
    incrementTimer() {
      this.timerDuration = Math.min(
        15 * ONE_MINUTE,
        this.timerDuration + ONE_MINUTE
      );
      console.log(this.$store.state.session.timer);
    },
    decrementTimer() {
      this.timerDuration = Math.max(
        ONE_MINUTE,
        this.timerDuration - ONE_MINUTE
      );
    },
    initTimer({
      timestampPause,
      timestampStart,
      timeRemaining,
      isStopped,
      isRunning,
      timerDuration,
    }) {
      this.timestampPause = timestampPause;
      this.timestampStart = timestampStart;
      this.timeRemaining = timeRemaining;
      this.isStopped = isStopped;
      this.isRunning = isRunning;
      this.timerDuration = timerDuration;
    },
  },
  watch: {
    isRunning(value) {
      const timestamp = Date.now();
      // TIMER STARTED
      if (value) {
        this.timestampStart = timestamp;
        this.timeRemaining = this.timestampPause
          ? this.timeRemaining
          : this.timerDuration;

        const timerParams = {
          timestampStart: timestamp,
          timestampPause: this.timestampPause,
          timeRemaining: this.timeRemaining,
          timerDuration: this.timerDuration,
          isRunning: value,
          isStopped: this.isStopped,
        };

        if (!this.timerInterval) {
          this.timerInterval = setInterval(() => {
            if (this.timerDisplay > 0) {
              this.timerDisplay =
                this.timeRemaining - (Date.now() - this.timestampStart);
            } else {
              clearInterval(this.timerInterval);
            }
          }, 200);
        }

        this.$store.commit('session/setTimer', timerParams);
      }
      // TIMER PAUSED
      else {
        this.timestampPause = timestamp;
        this.timeRemaining =
          this.timeRemaining - (this.timestampPause - this.timestampStart);

        const timerParams = {
          timestampPause: timestamp,
          timestampStart: this.timestampStart,
          timeRemaining: this.timeRemaining,
          timerDuration: this.timerDuration,
          isRunning: value,
          isStopped: this.isStopped,
        };

        clearInterval(this.timerInterval);
        this.timerInterval = null;

        this.$store.commit('session/setTimer', timerParams);
      }
    },
    timeRemaining(value) {
      this.timerDisplay = value;
      console.log('timeRemaining', value);
    },
    '$store.state.session.timer': function (value) {
      if (this.$store.state.session.isSpectator) {
        this.initTimer(value);
      }
    },
    '$store.state.session.timer.isRunning': function (value) {
      const { timestampPause, timestampStart, timeRemaining, isStopped } =
        this.$store.state.session.timer;
      // '$store.state.session.setTimer': function (value) {
      console.log('🏳️‍🌈', value);
      this.isRunning = value;
      this.timestampPause = timestampPause;
      this.timestampStart = timestampStart;
      this.timeRemaining = timeRemaining;
      this.isStopped = isStopped;
    },
    '$store.state.session.timer.isStopped': function (value) {
      // '$store.state.session.setTimer': function (value) {
      console.log('🏳️‍🌈🏳️‍🌈🏳️‍🌈', value);
      this.isStopped = value;
    },
    '$store.state.session.timer.timerDuration': function (value) {
      this.timerDuration = value;
    },
  },
  // watch: {
  //   timerEnabled(value) {
  //     if (value) {
  //       setTimeout(() => {
  //         if (value) this.time--;
  //       }, 1000);
  //     }
  //   },

  //   time: {
  //     handler(value) {

  //       if (value > 0 && this.timerEnabled) {
  //         setTimeout(() => {
  //           this.time--;
  //         }, 1000);
  //       }

  //     },
  //     immediate: true // This ensures the watcher is triggered upon creation
  //   },
  // }
};
</script>

<!-- 
Timer shape: 
{ startTime: number, duration: number in seconds, }
-->
<style>
.timer {
  position: absolute;
  top: 5%;
  left: 5%;
  display: flex;
  align-items: center;
  background-color: rgb(71, 115, 127);
  flex-direction: column;
  flex-wrap: wrap;
  text-wrap: wrap;
  max-width: 20%;
}
</style>
