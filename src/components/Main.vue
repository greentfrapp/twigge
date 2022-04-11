<template>
  <div class="flex justify-center items-center min-h-screen w-screen overflow-x-hidden">
    <div class="board relative w-max max-w-lg flex flex-col items-center justify-center gap-2 pb-16">
      <div class="absolute top-0 bg-white sm:rounded-xl flex flex-col justify-center min-w-screen px-6 sm:px-10 py-4 sm:py-10 gap-6 text-center">
        <div class="text-prose font-serif text-2xl sm:text-3xl max-w-prose whitespace-pre-wrap text-gray-800">
          {{ tweet }}
        </div>
        <div class="font-serif text-2xl text-gray-700">
          {{ author }}, {{ date }}
        </div>
      </div>
      <div class="tweet bg-white sm:shadow-around sm:rounded-xl flex flex-col justify-center min-w-screen px-6 sm:px-10 py-4 sm:py-10 gap-6 text-center z-10">
        <div class="text-prose font-serif text-2xl sm:text-3xl max-w-prose whitespace-pre-wrap text-gray-800">
          {{ tweet }}
        </div>
        <div class="font-serif text-2xl text-gray-700">
          {{ author }}, {{ date }}
        </div>
      </div>
      <CheckCircleIcon class="absolute text-gray-500 h-36 pointer-events-none z-20" :style="{ opacity: yesOpacity }" />
      <XCircleIcon class="absolute text-gray-500 h-36 pointer-events-none z-20" :style="{ opacity: noOpacity }" />
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
      tweet: 'The evolution of API for running cutting edge AI:\n- run it on your own machine \n- run it in the cloud\n- apply pay for and query an api endpoint\n- pretty please ask one of the authors to run it for you on Twitter \n🥲',
      author: '@karpathy',
      date: 'Apr 8', 
      topCard: null as any,
      isPanning: false,
      startPosX: 0,
      yesOpacity: 0,
      noOpacity: 0,
    }
  },
  methods: {
    reset () {
      this.topCard.style.transition = null
      this.topCard.style.transform = null
      this.topCard.style.opacity = 1
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

        this.yesOpacity = 0
        this.noOpacity = 0
        
        this.isPanning = false
        
        // set back transition property
        this.topCard.style.transition = 'all 200ms ease-out'
        
        // check threshold
        if (propX > 0.25 && e.direction === Hammer.DIRECTION_RIGHT) {
          // get right border position
          posX = this.board.clientWidth
          // throw card towards the right border
          this.topCard.style.transform =
            'translateX(' + posX + 'px) rotate(' + deg + 'deg)'
          this.topCard.style.opacity = 0
          
        } else if (propX < -0.25 && e.direction === Hammer.DIRECTION_LEFT) {
          // get left border position
          posX = - this.board.clientWidth
          // throw card towards the left border
          this.topCard.style.transform =
            'translateX(' + posX + 'px) rotate(' + deg + 'deg)'
          this.topCard.style.opacity = 0
          
        } else {
          
          // reset card position
          this.topCard.style.transform = 'none'
          
        }
        setTimeout(this.reset, 200)
        
      }
    }
  },
})
</script>

