<template>
  <h1>SpeedType</h1>
  <span class="span_title">How fast can you write?</span>
  <div class="box">
    <button :class="'button ' + button_status" @click="button_click"><i :class="button_status == 'Start' ? 'fa fa-play' : 'fa fa-pause'"></i>&nbsp;{{button_status}}</button><span class="time">{{time}}</span><br/>
    <div class="spans"><span class="done">{{done}}</span><span class="underline">{{underlined != ' ' ? underlined : '&nbsp;'}}</span><span class="text">{{random_string}}</span><br/><span class="high_score"></span><br/><span class="book"></span></div>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      name: 'App',
      button_status: 'Start',
      random_string: '',
      done: '',
      time: '00:00',
      underlined: '',
      seconds_left: 0,
      lpm: 0
    }
  },
  methods: {
    button_click () {
      document.querySelector('button').blur()
      if (this.button_status === 'Start') {
        this.random_string = ''
        this.lpm = 0
        document.querySelector('.book').innerText = ''
        if (this.high_score) {
          document.querySelector('.high_score').innerHTML = 'High Score: ' + this.high_score
        } else {
          document.querySelector('.high_score').innerHTML = 'High Score: 0'
        }
        for (let i = 0; i < 150; i++) {
          const randomElement = this.words[Math.floor(Math.random() * this.words.length)]
          if (randomElement.length > 1 && !this.random_string.includes(randomElement)) {
            this.random_string = this.random_string + randomElement + ' '
          }
        }
        this.button_status = 'Stop'
        this.underlined = this.random_string.charAt(0)
        this.random_string = this.random_string.substring(1)
        this.seconds_left = 59
        this.time = '00:' + this.seconds_left
        this.time_interval = setInterval(() => {
          this.seconds_left--
          let seconds = this.seconds_left
          if (seconds < 10) {
            document.querySelector('.time').classList.add('heartBeat')
            seconds = '0' + seconds
          }
          this.time = '00:' + seconds
        }, 1000)
      } else {
        this.stop()
      }
    },
    stop () {
      this.button_status = 'Start'
      this.random_string = ''
      this.underlined = ''
      this.done = ''
      document.querySelector('.time').className = 'time'
      clearInterval(this.time_interval)
      delete this.seconds_left
      const randomBook = this.books[Math.floor(Math.random() * this.books.length)]
      this.random_string = 'Your Score: ' + this.lpm + ' Letters per minute!'
      if (this.lpm > 0) {
        if (!localStorage.getItem('speedtype_high_score') || this.lpm > this.high_score) {
          this.high_score = this.lpm
          localStorage.setItem('speedtype_high_score', this.high_score)
          document.querySelector('.high_score').innerHTML = '<span class="new_hs">New High Score</span>: ' + this.high_score
        } else {
          document.querySelector('.high_score').innerHTML = 'High Score: ' + this.high_score
        }
        document.querySelector('.book').innerText = 'You could write "' + randomBook.title + '" in just ' + ((parseInt(randomBook.words) / (this.lpm / 3.8214)) / 60).toFixed(2) + ' hours.'
      } else {
        document.querySelector('.book').innerText = 'You should try using a keyboard!'
      }
      this.time = '00:00'
    }
  },
  created () {
    this.words = require('@/assets/words.json')
    this.books = require('@/assets/books.json')
    document.addEventListener('keydown', (e) => {
      if (this.button_status === 'Stop') {
        if (e.key.toLowerCase() === this.underlined) {
          this.lpm++
          this.done += this.underlined
          this.underlined = this.random_string.charAt(0)
          this.random_string = this.random_string.substring(1)
          if (this.done.length > 16) {
            this.done = this.done.substring(1)
          }
        }
      }
    })
  },
  mounted () {
    if (localStorage.getItem('speedtype_high_score')) {
      this.high_score = localStorage.getItem('speedtype_high_score')
      document.querySelector('.high_score').innerHTML = 'High Score: ' + this.high_score
    }
  },
  watch: {
    seconds_left: function () {
      if (this.seconds_left === 0) {
        this.stop()
      }
    }
  }
}
</script>
