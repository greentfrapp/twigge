<template>
  <div class="flex justify-center items-center min-h-screen w-screen overflow-x-hidden">
    <div class="board relative w-max max-w-lg flex flex-col items-center justify-center gap-2">
      <div class="next-container absolute top-0 h-screen sm:h-[32rem] overflow-auto sm:rounded-xl sm:border flex flex-col items-center bg-white w-[32rem]">
        <div v-for="(tweet, i) in tweets[1]" :key="i"
          class="bg-white min-h-screen sm:min-h-full w-screen sm:w-full flex flex-col justify-center px-6 sm:px-10 py-4 sm:py-10 gap-6 text-center z-10">
          <div class="font-serif text-2xl sm:text-3xl max-w-prose whitespace-pre-wrap text-gray-800">
            {{ tweet.text }}
          </div>
          <div class="font-serif text-xl text-gray-700">
            {{ tweet.author }}, {{ tweet.date }}
          </div>
        </div>
      </div>
      <div class="tweet-container h-screen sm:h-[32rem] overflow-auto z-10 shadow-around sm:rounded-xl cursor-pointer flex flex-col divide-y items-center bg-white w-[32rem]">
        <div v-for="(tweet, i) in tweets[0]" :key="i" :id="`${i}`"
          class="relative tweet bg-white min-h-screen sm:min-h-full w-screen sm:w-full flex flex-col justify-center py-4 sm:py-10 gap-6 text-center z-10">
          <div class="font-serif text-2xl sm:text-3xl max-w-prose whitespace-pre-wrap text-gray-800 px-6 sm:px-10">
            {{ tweet.text }}
          </div>
          <div class="font-serif text-xl text-gray-700 px-6 sm:px-10">
            {{ tweet.author }}, {{ tweet.date }}
          </div>
          <div class="absolute bottom-20 sm:bottom-4 w-full flex justify-center">
            <div class="w-1/2 flex justify-end px-10" @click="scrollUp">
              <ChevronUpIcon v-if="i > 0" class="h-6 text-gray-300 z-50" />
            </div>
            <div class="w-1/2 flex px-10" @click="scrollDown">
              <ChevronDownIcon v-if="i < (tweets[0].length - 1)" class="h-6 text-gray-300 z-50" />
            </div>
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
.tweet-container * {
  touch-action: none;
}

/* .tweet-container {
  scroll-snap-type: y mandatory;
} */

.tweet-container::-webkit-scrollbar,
.next-container::-webkit-scrollbar {
  display: none;
}

/* Hide scrollbar for IE, Edge and Firefox */
.tweet-container,
.next-container {
  -ms-overflow-style: none;  /* IE and Edge */
  scrollbar-width: none;  /* Firefox */
}

/* .tweet {
  scroll-snap-align: start;
} */
</style>

<script lang="ts">
import { defineComponent } from 'vue'
import { HeartIcon, RefreshIcon, ChevronDownIcon, ChevronUpIcon } from '@heroicons/vue/outline'
import { CheckCircleIcon, XCircleIcon } from '@heroicons/vue/solid'
import Hammer from 'hammerjs'
import smoothscroll from 'smoothscroll-polyfill';

export default defineComponent({
  mounted () {
    smoothscroll.polyfill()
    // this.searchTweets()
    this.board = document.querySelector('.board')
    const tweet = document.querySelector('.tweet-container') as HTMLElement
    this.topCard = tweet
    const hammer = new Hammer(tweet)
    hammer.on('pan', this.panHandler)
  },
  components: {
    HeartIcon,
    RefreshIcon,
    CheckCircleIcon,
    XCircleIcon,
    ChevronUpIcon,
    ChevronDownIcon,
  },
  data () {
    return {
      board: null as any,
      tweets: [
        [{
          text: 'The evolution of API for running cutting edge AI:\n- run it on your own machine \n- run it in the cloud\n- apply pay for and query an api endpoint\n- pretty please ask one of the authors to run it for you on Twitter \n🥲',
          author: 'Andrej Karpathy',
          date: 'Apr 8',
        }, {
          text: 'Test 1',
          author: 'Andrej Karpathy',
          date: 'Apr 8',
        }, {
          text: 'Test 2',
          author: 'Andrej Karpathy',
          date: 'Apr 8',
        }], [{
          text: 'Paradox of ‘repairability’ - the choices that make it more reliable also make it harder to repair.',
          author: 'Benedict Evans',
          date: 'Apr 11',
        }], 
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
      setTimeout(() => {
        this.tweets.shift()
        if (this.tweets.length < 4) {
          // this.searchTweets()
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
        this.nextToken = data.next_token
        data.tweets.filter((tweets:any) => tweets[0].text.split(/\r\n|\r|\n/).length <= 5)
          .forEach((tweets:any) => {
            this.tweets.push(tweets.filter((tweet:any) => tweet.text.split(/\r\n|\r|\n/).length <= 5)
              .map((tweet:any) => {
                return {
                  text: tweet.text.replace('&gt;', '>').replace('&lt;', '<').replace('&amp;', '&'),
                  author: tweet.author,
                  date: tweet.date,
                }
              })
            )
          })
      })
    },
    scrollUp () {
      this.topCard.scrollBy({
        top: -100,
        behavior: 'smooth',
      })
    },
    scrollDown () {
      this.topCard.scrollBy({
        top: 100,
        behavior: 'smooth',
      })
    },
    panHandler (e:any) {
      // if (e.deltaY < -10) {
      //   this.topCard.scrollBy({
      //     top: 10,
      //     behavior: 'smooth',
      //   })
      // } else if (e.deltaY > 10) {
      //   this.topCard.scrollBy({
      //     top: -10,
      //     behavior: 'smooth',
      //   })
      // }
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
              this.topCard.scrollTop = 0
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
              this.topCard.scrollTop = 0
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

