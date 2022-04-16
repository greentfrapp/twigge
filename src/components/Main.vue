<template>
  <div class="flex flex-col justify-center items-center min-h-screen w-screen sm:w-max mx-auto overflow-x-hidden gap-4 sm:px-4">
    <div v-if="numSwipes < 3" class="bg-white flex justify-evenly py-4 w-full shadow-around sm:rounded-xl z-20 sm:z-0 px-4 text-gray-700 text-center flex flex-col gap-1">
      <h1 class="font-bold">Twigge</h1>
      <h2 class="italic">A tweet at a time</h2>
      <div class="font-light">
        Swipe right on tweets you like, otherwise swipe left.<br/>
        Swipe up and down to view threads.<br/>
        Arrow keys work as well.
      </div>
    </div>
    <div class="board relative w-max max-w-lg flex flex-col items-center justify-center gap-2">
      <div v-if="tweets.length > 1" class="next-container absolute top-0 h-screen sm:h-[32rem] overflow-auto sm:rounded-xl sm:border flex flex-col items-center bg-white w-[32rem]">
        <div v-for="(tweet, i) in tweets[1]" :key="i"
          class="bg-white min-h-screen sm:min-h-full w-screen sm:w-full flex flex-col justify-center px-6 sm:px-10 py-4 sm:py-10 gap-6 text-center z-10">
          <div class="font-serif text-2xl max-w-prose whitespace-pre-wrap text-gray-800">
            {{ tweet.text }}
          </div>
          <div class="font-serif text-xl text-gray-700">
            {{ tweet.author }}, {{ tweet.date }}
          </div>
        </div>
      </div>
      <div class="tweet-container h-screen sm:h-[32rem] overflow-auto z-10 shadow-around sm:rounded-xl cursor-pointer flex flex-col divide-y items-center bg-white w-[32rem]">
        <div v-if="tweets.length === 0" class="text-gray-700 min-h-screen sm:min-h-full w-screen sm:w-full flex justify-center items-center">
          Loading...
        </div>
        <div v-else v-for="(tweet, i) in tweets[0]" :key="i" :id="`${i}`"
          class="relative tweet bg-white min-h-screen sm:min-h-full w-screen sm:w-full flex flex-col justify-center py-4 sm:py-16 gap-6 text-center z-10">
          <div class="font-serif text-2xl max-w-prose whitespace-pre-wrap text-gray-800 px-6 sm:px-10">
            {{ tweet.text }}
          </div>
          <div class="font-serif text-xl text-gray-700 px-6 sm:px-10">
            {{ tweet.author }}, {{ tweet.date }}
          </div>
          <div class="absolute bottom-20 sm:bottom-4 w-full flex justify-center items-center">
            <div v-if="i > 0" class="w-1/2 flex justify-center px-10" @click="swipe('down')">
              <ChevronUpIcon class="h-6 text-gray-300 z-50" />
            </div>
            <div v-if="i < (tweets[0].length - 1)" class="w-1/2 flex justify-center px-10" @click="swipe('up')">
              <ChevronDownIcon class="h-6 text-gray-300 z-50" />
            </div>
          </div>
        </div>
      </div>
      <CheckCircleIcon class="absolute text-gray-500 h-36 pointer-events-none z-20" :style="{ opacity: yesOpacity, transition: hintTransition }" />
      <XCircleIcon class="absolute text-gray-500 h-36 pointer-events-none z-20" :style="{ opacity: noOpacity, transition: hintTransition }" />
      <!-- <div class="fixed bottom-0 sm:static bg-white flex justify-evenly py-4 w-full shadow-around sm:rounded-xl z-20 sm:z-0">
        <HeartIcon class="text-gray-500 h-6 w-6 cursor-pointer" />
        <RefreshIcon class="text-gray-500 h-6 w-6 cursor-pointer" />
      </div> -->
    </div>
  </div>
</template>

<style scoped>
.tweet-container * {
  touch-action: none;
}

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
</style>

<script lang="ts">
import { defineComponent } from 'vue'
import { HeartIcon, RefreshIcon, ChevronDownIcon, ChevronUpIcon } from '@heroicons/vue/outline'
import { CheckCircleIcon, XCircleIcon } from '@heroicons/vue/solid'
import Hammer from 'hammerjs'
import smoothscroll from 'smoothscroll-polyfill'

export default defineComponent({
  mounted () {
    smoothscroll.polyfill()
    this.searchTweets()
    this.board = document.querySelector('.board')
    const tweet = document.querySelector('.tweet-container') as HTMLElement
    this.topCard = tweet
    const hammer = new Hammer(tweet)
    hammer.on('pan', this.panHandler)
    document.addEventListener('keydown', (event) => {
      const key = event.key
      if (key === 'ArrowUp') {
        event.preventDefault()
        this.swipe('down')
      } else if (key === 'ArrowDown') {
        event.preventDefault()
        this.swipe('up')
      } else if (key === 'ArrowRight') {
        event.preventDefault()
        this.swipe('right')
      } else if (key === 'ArrowLeft') {
        event.preventDefault()
        this.swipe('left')
      }
    }, false)
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
        // [{
        //   text: 'The evolution of API for running cutting edge AI:\n- run it on your own machine \n- run it in the cloud\n- apply pay for and query an api endpoint\n- pretty please ask one of the authors to run it for you on Twitter \n🥲',
        //   author: 'Andrej Karpathy',
        //   date: 'Apr 8',
        // }, {
        //   text: 'Test 1',
        //   author: 'Andrej Karpathy',
        //   date: 'Apr 8',
        // }, {
        //   text: 'Test 2',
        //   author: 'Andrej Karpathy',
        //   date: 'Apr 8',
        // }], [{
        //   text: 'Paradox of ‘repairability’ - the choices that make it more reliable also make it harder to repair.',
        //   author: 'Benedict Evans',
        //   date: 'Apr 11',
        // }], 
      ] as any[],
      topCard: null as any,
      isPanning: false,
      startPosX: 0,
      yesOpacity: 0,
      noOpacity: 0,
      hintTransition: 'none',
      lambdaUrl: 'https://y6c1a626wf.execute-api.us-east-1.amazonaws.com/default/twigge-dev',
      nextToken: null as any,
      querySuffix: "-is:retweet -has:links -has:hashtags -has:mentions -has:media lang:en -dm -is:reply -👇 -🧵 -🧵⬇",
      replyId: 0,
      scrolling: false,
      likedContexts: [] as string[],
      numSwipes: 0,
      readIds: [] as string[],
      defaultContexts: [
          'context:66.848921413196984320', // Computer programming
          'context:67.1103294729079349250', // Nature
          'context:46.781974597218340864', // Video games
          'context:45.781974597310615553', // Entertainment
          'context:65.1280550787207147521', // Arts & culture
          'context:65.844603730221707264', // Fashion
          'context:45.781974596148793345', // Business & finance
      ],
      contexts: [] as string[],
    }
  },
  methods: {
    async getRelatedTweets () {
      if (!!this.tweets[0][0].context_annotations) {
        this.tweets[0][0].context_annotations.forEach((annotation:any) => {
          const contextId = `${annotation.domain.id}.${annotation.entity.id}`
          this.contexts.push(`context:${contextId}`)
        })
      }
    },
    reset () {
      this.topCard.style.transition = null
      this.topCard.style.transform = null
      this.topCard.style.opacity = 1
    },
    async updateTweet () {
      setTimeout(() => {
        this.tweets.shift()
        if (this.tweets.length < 5) {
          this.searchTweets()
        }
      }, 200)
    },
    async searchTweets (searchTerm:string='') {
      if (!searchTerm) {
        if (this.contexts.length) searchTerm = this.contexts[Math.floor(Math.random() * this.contexts.length)]
        else searchTerm = this.defaultContexts[Math.floor(Math.random() * this.defaultContexts.length)]
      }
      const numTweets = this.tweets.length

      while (this.tweets.length <= numTweets) {
        await fetch(this.lambdaUrl, {
          method: 'POST',
          body: JSON.stringify({
            query: `${searchTerm ? searchTerm + ' ' : ''}${this.querySuffix}`,
            next_token: this.nextToken,
          })
        }).then(async response => {
          const data = await response.json()
          this.nextToken = data.next_token
          data.tweets
            .filter((tweets:any) => tweets[0].text.split(/\r\n|\r|\n/).length <= 5)
            .filter((tweets:any) => tweets[0].text.length > 20)
            .filter((tweets:any) => !tweets[0].text.includes('https://'))
            .filter((tweets:any) => !this.readIds.includes(tweets[0].id))
            .forEach((tweets:any) => {
              this.nextToken = null
              this.readIds.push(tweets[0].id)
              this.tweets.push(tweets.filter((tweet:any) => tweet.text.split(/\r\n|\r|\n/).length <= 5)
                .map((tweet:any) => {
                  return {
                    text: tweet.text.replace('&gt;', '>').replace('&lt;', '<').replace('&amp;', '&'),
                    author: tweet.author,
                    date: tweet.date,
                    context_annotations: tweet.context_annotations,
                  }
                })
              )
            })
        })
      }
    },
    swipe (dir:string) {
      if (['up', 'down'].includes(dir)) {
        if (!this.scrolling) {
          this.scrolling = true
          if (dir === 'down') this.replyId = Math.max(0, this.replyId - 1)
          else this.replyId = Math.min(this.tweets[0].length - 1, this.replyId + 1)
          document.getElementById(`${this.replyId}`)?.scrollIntoView({behavior: "smooth", block: "end", inline: "nearest"})
          setTimeout(() => this.scrolling = false, 300)
        }
      } else if (['left', 'right'].includes(dir)) {
        this.numSwipes += 1
        if (dir === 'right') this.getRelatedTweets()
        this.hintTransition = 'all 500ms ease-out'
        this.yesOpacity = 0
        this.noOpacity = 0
        this.isPanning = false
        // set back transition property
        this.topCard.style.transition = 'all 500ms ease-out'
        this.updateTweet()
          .then(() => {
            // get right border position
            const posX = this.board.clientWidth * (dir === 'left' ? -1 : 1)
            // throw card towards the left/right border
            this.topCard.style.transform =
              'translateX(' + posX + 'px) rotate(' + (dir === 'left' ? '-' : '') + '45deg)'
            this.topCard.style.opacity = 0
            this.topCard.scrollTop = 0
          })
          .then(() => setTimeout(this.reset, 200))
      }
    },
    panHandler (e:any) {
      if (e.deltaY < -10) {
        this.swipe('up')
      } else if (e.deltaY > 10) {
        this.swipe('down')
      }
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
        
        // check threshold
        if (propX > 0.25 && e.direction === Hammer.DIRECTION_RIGHT) {
          this.getRelatedTweets()
          this.swipe('right')
        } else if (propX < -0.25 && e.direction === Hammer.DIRECTION_LEFT) {
          this.swipe('left')
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

