<template>
  <div class="flex justify-center items-center min-h-screen w-screen overflow-x-hidden">
    <div class="board relative w-max max-w-lg flex flex-col items-center justify-center gap-2">
      <div class="absolute top-0 h-screen sm:h-[32rem] sm:rounded-xl sm:border flex items-center bg-white">
        <div class="bg-white w-screen flex flex-col justify-center px-6 sm:px-10 py-4 sm:py-10 gap-6 text-center">
          <div class="text-prose font-serif text-2xl sm:text-3xl max-w-prose whitespace-pre-wrap text-gray-800">
            {{ tweets[1].text }}
          </div>
          <div class="font-serif text-xl text-gray-700">
            {{ tweets[1].author }}, {{ tweets[1].date }}
          </div>
        </div>
      </div>
      <div class="tweet h-screen sm:h-[32rem] overflow-auto z-10 shadow-around sm:rounded-xl cursor-pointer flex items-center bg-white">
        <div class="bg-white w-screen flex flex-col justify-center px-6 sm:px-10 py-4 sm:py-10 gap-6 text-center z-10">
          <div class="font-serif text-2xl sm:text-3xl max-w-prose whitespace-pre-wrap text-gray-800">
            {{ tweets[0].text }}
          </div>
          <div class="font-serif text-xl text-gray-700">
            {{ tweets[0].author }}, {{ tweets[0].date }}
          </div>
        </div>
      </div>
      <CheckCircleIcon class="absolute text-gray-500 h-36 pointer-events-none z-20" :style="{ opacity: yesOpacity, transition: hintTransition }" />
      <XCircleIcon class="absolute text-gray-500 h-36 pointer-events-none z-20" :style="{ opacity: noOpacity, transition: hintTransition }" />
      <div class="fixed bottom-0 sm:static bg-white flex justify-evenly py-4 w-full shadow-around sm:rounded-xl z-20 sm:z-0">
        <HeartIcon class="text-gray-500 h-6 w-6 cursor-pointer" />
        <RefreshIcon class="text-gray-500 h-6 w-6 cursor-pointer" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.tweet * {
  touch-action: none;
}
</style>

<script lang="ts">
import { defineComponent } from 'vue'
import { HeartIcon, RefreshIcon } from '@heroicons/vue/outline'
import { CheckCircleIcon, XCircleIcon } from '@heroicons/vue/solid'
import Hammer from 'hammerjs'

export default defineComponent({
  mounted () {
    this.searchTweets()
    this.board = document.querySelector('.board')
    const tweet = document.querySelector('.tweet') as HTMLElement
    this.topCard = tweet
    const hammer = new Hammer(tweet)
    hammer.on('pan', this.panHandler)
  },
  components: {
    HeartIcon,
    RefreshIcon,
    CheckCircleIcon,
    XCircleIcon,
  },
  data () {
    return {
      board: null as any,
      tweets: [
        {
          text: 'The evolution of API for running cutting edge AI:\n- run it on your own machine \n- run it in the cloud\n- apply pay for and query an api endpoint\n- pretty please ask one of the authors to run it for you on Twitter \n🥲',
          author: 'Andrej Karpathy',
          date: 'Apr 8',
        }, {
          text: 'Paradox of ‘repairability’ - the choices that make it more reliable also make it harder to repair.',
          author: 'Benedict Evans',
          date: 'Apr 11',
        },
      ] as any[],
      topCard: null as any,
      isPanning: false,
      startPosX: 0,
      yesOpacity: 0,
      noOpacity: 0,
      hintTransition: 'none',
      lambdaUrl: 'https://y6c1a626wf.execute-api.us-east-1.amazonaws.com/default/twigge-dev',
      nextToken: null as any,
    }
  },
  methods: {
    reset () {
      this.topCard.style.transition = null
      this.topCard.style.transform = null
      this.topCard.style.opacity = 1
    },
    async updateTweet () {
      // const tweet = this.tweets[0]
      // this.tweets[0] = this.tweets[1]
      // setTimeout(() => this.tweets[1] = tweet, 200)
      setTimeout(() => {
        this.tweets.shift()
        if (this.tweets.length < 4) {
          console.log('refreshing')
          this.searchTweets()
        }
      }, 200)
    },
    async searchTweets () {
      fetch(this.lambdaUrl, {
        method: 'POST',
        body: JSON.stringify({
          query: "context:66.848921413196984320 -is:retweet -has:links -has:hashtags -has:mentions -has:media lang:en -dm -is:reply -👇 -🧵 -🧵⬇",
          next_token: this.nextToken,
        })
      }).then(async response => {
        const data = await response.json()
        console.log(data)
        this.nextToken = data.next_token
        data.tweets.forEach((tweet:any) => {
          this.tweets.push(tweet)
        })
      })
    },
    panHandler (e:any) {
      if (e.additionalEvent === 'panup') console.log('bad')
      if (e.additionalEvent === 'pandown') console.log('bad')
      if (!this.isPanning) {
    
        this.isPanning = true
        
        // remove transition property
        this.topCard.style.transition = null
        
        // get card coordinates in pixels
        let style = window.getComputedStyle(this.topCard)
        let mx = style.transform.match(/^matrix\((.+)\)$/)
        this.startPosX = mx ? parseFloat(mx[1].split(', ')[4]) : 0
      }
      
      // get new coordinates
      let posX = e.deltaX + this.startPosX
      
      // get ratio between swiped pixels and the axes
      let propX = e.deltaX / this.board.clientWidth
      
      // get swipe direction, left (-1) or right (1)
      let dirX = e.deltaX < 0 ? -1 : 1
      
      // get degrees of rotation (between 0 and +/- 45)
      let deg = dirX * Math.abs(propX) * 45
      
      // move and rotate card
      this.topCard.style.transform =
        'translateX(' + posX + 'px) rotate(' + deg + 'deg)'

      if (dirX === 1) {
        this.yesOpacity = propX
      } else {
        this.noOpacity = -propX
      }
      
      if (e.isFinal) {

        this.hintTransition = 'all 500ms ease-out'
        this.yesOpacity = 0
        this.noOpacity = 0
        
        this.isPanning = false
        
        // set back transition property
        this.topCard.style.transition = 'all 500ms ease-out'
        
        // check threshold
        if (propX > 0.25 && e.direction === Hammer.DIRECTION_RIGHT) {
          this.updateTweet()
            .then(() => {
              // get right border position
              posX = this.board.clientWidth
              // throw card towards the right border
              this.topCard.style.transform =
                'translateX(' + posX + 'px) rotate(' + deg + 'deg)'
              this.topCard.style.opacity = 0
            })
        } else if (propX < -0.25 && e.direction === Hammer.DIRECTION_LEFT) {
          this.updateTweet()
            .then(() => {
              // get left border position
              posX = - this.board.clientWidth
              // throw card towards the left border
              this.topCard.style.transform =
                'translateX(' + posX + 'px) rotate(' + deg + 'deg)'
              this.topCard.style.opacity = 0
            })
        } else {
          // reset card position
          this.topCard.style.transform = 'none'
        }
        setTimeout(this.reset, 200)
        
      } else {
        this.hintTransition = 'none'
      }
    }
  },
})
</script>

