<template>
  <div class="Timer-wrapper">
    <app-audio/>
    <app-timer-dial :minutes="minutes">
      <p class="Dial-time" v-if="!timerStarted">{{ prettyMinutes }}</p>
      <p class="Dial-time" v-else>{{ prettyTime }}</p>
    </app-timer-dial>

    <section class="Container Button-wrapper">
      <transition name="fade" mode="out-in">
        <div class="Button" v-if="!timerStarted" @click="startTimer" :key="'start'">
          <div class="Button-icon-wrapper">
            <svg version="1.2" baseProfile="tiny" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
              x="0px" y="0px" viewBox="0 0 7.6 15" xml:space="preserve" height="15px" class="Icon--start">
              <polygon fill="#F6F2EB" points="0,0 0,15 7.6,7.4 "/>
            </svg>
          </div>
        </div>
        <div class="Button" v-if="timerStarted && !timerActive" @click="resumeTimer" :key="'resume'">
          <div class="Button-icon-wrapper">
            <svg version="1.2" baseProfile="tiny" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
              x="0px" y="0px" viewBox="0 0 7.6 15" xml:space="preserve" height="15px">
              <polygon fill="#F6F2EB" points="0,0 0,15 7.6,7.4 "/>
            </svg>
          </div>
        </div>
        <div class="Button" v-else-if="timerStarted && timerActive" @click="pauseTimer" :key="'pause'">
          <div class="Button-icon-wrapper">
            <svg version="1.2" baseProfile="tiny" id="Layer_2" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"   x="0px" y="0px" viewBox="0 0 10.9 18" xml:space="preserve" height="15px" class="Icon--pause">
                <line fill="none" stroke="#F6F2EB" stroke-width="3" stroke-linecap="round" stroke-miterlimit="10" x1="1.5" y1="1.5" x2="1.5" y2="16.5"/>
              
                <line fill="none" stroke="#F6F2EB" stroke-width="3" stroke-linecap="round" stroke-miterlimit="10" x1="9.4" y1="1.5" x2="9.4" y2="16.5"/>
            </svg>
          </div>
        </div>
      </transition>
    </section>

    <app-timer-footer/>

    <!-- <button @click="resetTimer">Reset</button> -->
    <app-timer-controller/>
    <sweet-modal title="What is your main focus?" ref="modal" blocking overlay-theme="dark" modal-theme="dark">
      <input type="text" class="task-input" v-model="task" @keyup.enter="confirmStart" placeholder="Killing bug...">
      <button class="confirm-btn" @click="confirmStart">Let's go</button>
    </sweet-modal>
  </div>
</template>

<script>
import Timer from '@/utils/timer'
import appAudio from '@/components/Audio'
import appTimerController from '@/components/timer/Timer-controller'
import appTimerDial from '@/components/timer/Timer-dial'
import appTimerFooter from '@/components/timer/Timer-footer'
import { SweetModal } from 'sweet-modal-vue'
import { EventBus } from '@/utils/event-bus'

export default {
  components: {
    appAudio,
    appTimerController,
    appTimerDial,
    appTimerFooter,
    SweetModal
  },

  data () {
    return {
      minutes: 1,
      timer: null,
      timerActive: false,
      timerStarted: false,
      task: ''
    }
  },

  computed: {
    // store getters
    currentRound () {
      return this.$store.getters.currentRound
    },

    timeLongBreak () {
      return this.$store.getters.timeLongBreak
    },

    timeShortBreak () {
      return this.$store.getters.timeShortBreak
    },

    timeWork () {
      return this.$store.getters.timeWork
    },

    // local
    prettyMinutes () {
      return this.minutes + ':00'
    },

    prettyTime () {
      return `${this.timeRemaining.remainingMinutes}:${this.timeRemaining.remainingSeconds}`
    },

    timeElapsed () {
      if (this.timer.time !== null) {
        const time = this.timer.time
        const minutes = Math.floor(time / 60)
        const seconds = time - (minutes * 60)
        return {
          minutes,
          seconds
        }
      }
    },

    timeRemaining () {
      if (this.timer.time !== null) {
        const minutes = this.minutes
        const time = this.timer.time
        const elapsedMinutes = Math.floor(time / 60)
        const elapsedSeconds = time - (elapsedMinutes * 60)
        const remainingSeconds = this.formatTimeDouble(60 - elapsedSeconds)
        let remainingMinutes = minutes - elapsedMinutes

        if (elapsedSeconds > 0) {
          remainingMinutes -= 1
        }

        return {
          remainingMinutes,
          remainingSeconds
        }
      }
    }
  },

  methods: {
    formatTimeDouble (time) {
      if (time === 60) {
        return '00'
      } else if (time < 10) {
        return `0${time}`
      } else {
        return time
      }
    },

    createTimer (min) {
      this.timer = new Timer(min)
    },

    initTimer () {
      switch (this.currentRound) {
        case 'work':
          this.minutes = this.timeWork
          this.createTimer(this.timeWork)
          break
        case 'short-break':
          this.minutes = this.timeShortBreak
          this.createTimer(this.timeShortBreak)
          break
        case 'long-break':
          this.minutes = this.timeLongBreak
          this.createTimer(this.timeLongBreak)
          break
        default:
          this.createTimer(25)
          break
      }
    },

    pauseTimer () {
      this.timer.pause()
      this.timerActive = !this.timerActive
    },

    resetTimer () {
      this.timer.reset()
      this.timerActive = !this.timerActive
      this.timerStarted = false
    },

    resumeTimer () {
      this.timer.resume()
      this.timerActive = true
    },

    startTimer () {
      this.$refs.modal.open()
    },

    confirmStart () {
      window.form = this
      console.log(this.task)
      if (!this.timerActive) {
        this.timer.start()
        this.timerActive = true
        this.timerStarted = true
        this.$refs.modal.close()
      }
    }
  },

  mounted () {
    this.initTimer()

    EventBus.$on('timer-init', opts => {
      // clear previous timers
      this.timer.reset()
      this.initTimer()
      if (opts.auto) {
        setTimeout(() => {
          this.startTimer()
        }, 1500)
      }
    })

    EventBus.$on('call-timer-reset', () => {
      this.resetTimer()
    })
  }
}
</script>

<style lang="scss" scoped>
.Button {
  border: 2px solid $colorBlueGrey;
  border-radius: 100%;
  display: flex;
  justify-content: center;
  transition: $transitionDefault;
  width: 50px;
  height: 50px;
  -webkit-app-region: no-drag;
  &:hover {
    background-color: $colorLightNavy;
    & .Icon--pause line {
      stroke: $colorRed;
    }
    & .Icon--start polygon {
      fill: $colorRed;
    }
  }
}

.sweet-modal-overlay.theme-dark {
  background: rgba(24, 32, 40, 0.6);
}

.sweet-content-content {
  text-align: center;
}

.task-input {
  font-size: 16px;
  padding: 10px;
  border-radius: 3px;
  width: 100%;
  border: none;

  &:focus, &:active {
    overflow: hidden;
    outline: none;
  }
}

.confirm-btn {
  margin-top: 18px;
  background-color: #FF4E4D;
  border: none;
  font-size: 16px;
  padding: 10px 16px;
  color: #F6F2EB;
  border-radius: 3px;

  &:focus, &:active {
    overflow: hidden;
    outline: none;
  }
}

.Button-wrapper {
  display: flex;
  justify-content: center;
  margin: 20px 0 10px 0;
}

.Button-icon-wrapper {
  align-items: center;
  display: flex;
  height: 100%;
}

.Dial-time {
  font-family: 'RobotoMono', monospace;
  font-size: 46px;
  margin: 0;
  position: absolute;
  top: 32%;
}

.Timer-wrapper {
  display: flex;
  flex-direction: column;
}
</style>
