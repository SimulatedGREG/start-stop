<template>
  <div>
    <header class="columns">
      <div class="column col-6">
        <button class="btn btn-lg btn-primary" :disabled="isRecording" @click="beginRecording">Start Timer</button>
      </div>
      <div class="column col-6">
        <button class="btn btn-lg btn-primary" :disabled="!isRecording" @click="endRecording">Stop Timer</button>
      </div>

      <div class="column col-12">
        <h5 class="text-center">
          {{ rep | human }}&nbsp;<span v-show="isRecording">{{ timeDiff }}</span>
        </h5>
      </div>
    </header>

    <main>
      <div class="time-gen" v-for="(time, index) in generation">
        <span class="index">{{ index + 1 }}.</span>
        <span class="time">{{ time.human }}</span>
      </div>
    </main>
  </div>
</template>

<script>
  import moment from 'moment'

  export default {
    name: 'app',
    created () {
      this.now = moment()

      setInterval(() => {
        this.now = moment()
      }, 1000)

      if (window.localStorage.getItem('startTime') && window.localStorage.getItem('endTime')) {
        this.startTime = JSON.parse(window.localStorage.getItem('startTime'))
        this.endTime = JSON.parse(window.localStorage.getItem('endTime'))
      } else {
        window.localStorage.setItem('startTime', JSON.stringify(this.startTime))
        window.localStorage.setItem('endTime', JSON.stringify(this.endTime))
      }
    },
    data () {
      return {
        isRecording: false,
        startTime: +new Date(),
        endTime: +new Date() + (1000 * 60 * 22),
        now: null
      }
    },
    computed: {
      timeDiff () {
        let diff = Math.floor(moment(this.now).diff(this.startTime) / 1000)
        return diff > 60
          ? Math.floor(diff / 60) + 'm ' + (diff % 60) + 's'
          : diff + 's'
      },
      rep () {
        return this.isRecording
          ? 'Recording...'
          : moment(this.endTime).diff(this.startTime)
      },
      generation () {
        if (this.isRecording) return []

        let times = []

        for (let i = 1; i < 51; i++) {
          let iteration = moment(this.endTime).add(moment(this.rep) * i)

          times.push({
            raw: iteration,
            human: moment(iteration).format('LTS')
          })
        }

        return times
      }
    },
    methods: {
      beginRecording () {
        this.isRecording = true
        this.startTime = moment()
        window.localStorage.setItem('startTime', JSON.stringify(this.startTime))
      },
      endRecording () {
        this.isRecording = false
        this.endTime = moment()
        window.localStorage.setItem('endTime', JSON.stringify(this.endTime))
      }
    },
    filters: {
      human (time) {
        if (time === 'Recording...') return time

        let seconds = Math.floor(time / 1000)

        return seconds > 60
          ? Math.floor(seconds / 60) + 'm ' + (seconds % 60) + 's'
          : Math.floor(time / 1000) + 's'
      }
    }
  }
</script>

<style>
  @import "~spectre.css";

  header {
    background: #fff;
    border-bottom: 1px solid rgba(0, 0, 0, .2);
    height: 124px;
    padding: 10px 20px;
    position: fixed;
    top: 0;
    width: 100vw;
  }

  main { margin-top: 124px; }

  h5 { margin: 0; }

  button { width: 100%; }

  .time-gen {
    display: flex;
    font-size: 2em;
    padding: 20px;
  }

  .time-gen:hover { background: #f0f1f4 !important; }
  .time-gen:nth-child(2n - 1) { background: #f8f9fa; }

  .index {
    margin-right: 10px;
    text-align: right;
    width: 40px;
  }
</style>
