<style lang="scss">
.veer-slider {
  position:relative;
}

.veer-arrows {
  button {
    position: absolute;
    top: 50%;
    width: 2rem;
    height: 2rem;
    background: transparent;
    border-width:0;
    border-top: .4rem solid black;
    border-right: .4rem solid black;
    box-shadow: 0 0 0 lightgray;
    opacity:0.5;
    transition: all 600ms ease;
    z-index:20;
    cursor:pointer;
    outline: none;

    &.veer-prev {
    	left: 10px;
    	transform: translate3d(0,-50%,0) rotate(-135deg);
    }

    &.veer-next {
    	right: 10px;
    	transform: translate3d(0,-50%,0) rotate(45deg);
    }

    &:hover {
    	opacity:1;
      border-color:white;

    	&.left {
    		left: -.2rem;
    	}

    	&.right {
    		right: -.2rem;
    	}
    }
  }
}

.veer-slides-mask {
  position:relative;
  overflow:hidden;

  .veer-slides {
    position:absolute;
    top:0;
    left:0;
    display:flex;
    flex-wrap: nowrap;
    transition:all 1s;

    .veer-slide {
      background-color:black;
      transition:all 1s;
    }
  }
}

.veer-pages {
  position:absolute;
  bottom:0;
  width:100%;
  display:flex;
  justify-content: center;
  list-style-type: none;
  z-index:20;
  padding:0;

  li {
    width:10px;
    height:10px;
    background-color:black;
    border-radius:10px;
    margin:0 5px;
    opacity:0.5;
    cursor:pointer;
    transition: all 600ms ease;

    &:hover {
      opacity:1;
      background-color:white;
    }
  }
}
</style>

<template>
  <div class="veer-slider" @mouseenter="pause" @mouseleave="unpause">
    <div
      class="veer-slides-mask"
      :style="{
        width: sliderMaskWidth + 'px',
        height: sliderMaskHeight + 'px'
        }">
        <div ref="slides" class="veer-slides" :style="{
          width: sliderWidth,
          height: sliderHeight
          }">
          <slot></slot>
        </div>
    </div>
    <div :class="o.arrowClass" v-if="o.showPrevNext">
      <button class="veer-prev" @click="goPrevious()"></button>
      <button class="veer-next" @click="goNext()"></button>
    </div>
    <ul :class="o.pageClass" v-if="o.showPages">
      <li v-for="(page, n) in slideCount" @click="goToSlide(n)"></li>
    </ul>
  </div>
</template>

<script>
  const defaultOptions = {
    autoplay: true,
    mode: 'fade',
    initialDelay: 0,
    speed: 4000,
    showPrevNext: true,
    showPages: true,
    pageClass: 'veer-pages',
    arrowClass: 'veer-arrows',
    width: 'auto',
    height: 'auto'
  }

  export default {
    data () {
      return {
        engine: null,
        slides: [],
        current: 1,
        playing: true,
        sliderMaskWidth: 0,
        sliderMaskHeight: 0,
        sliderWidth: 0,
        sliderHeight: 0,
        o: {}
      }
    },
    mounted: function () {
      this.mergeOptionsWithDefault()

      this.setSlideArray()
      this.prepareSliderMask()
      this.prepareSlider()
      this.prepareSlides()
      this.initialize()
    },
    methods: {
      initialize () {
        let self = this

        // Set the slides to autoplay
        this.playing = this.o.autoplay

        // This is the engine sucker!!!
        // Initial Delay
        setTimeout(function () {
          self.startEngine()
        }, self.o.initialDelay)

        if (this.slides.length <= 1) {
          this.playing = false
          this.o.showPages = false
          this.o.showPrevNext = false
        }
      },
      startEngine () {
        let self = this

        self.engine = setInterval(function () {
          if (self.playing) {
            self.goNext()
          }
        }, self.o.speed)
      },
      pause () {
        if (this.o.autoplay) {
          this.playing = false
        }
      },
      unpause () {
        if (this.o.autoplay) {
          this.playing = true
        }
      },
      mergeOptionsWithDefault () {
        this.o = Object.assign(defaultOptions, this.options)
      },
      animateToCurrent () {
        clearInterval(this.engine)

        switch (this.o.mode) {
          case 'vertical':
            break
          case 'fade':
            for (var i = 0; i < this.slides.length; i++) {
              this.slides[i].style.zIndex = 1
              this.slides[i].style.opacity = 0

              if (i === this.current - 1) {
                this.slides[i].style.zIndex = 10
                this.slides[i].style.opacity = 1
              }
            }
            break
          default:
            this.$refs.slides.style.left = (-(this.sliderMaskWidth * (this.current - 1))) + 'px'
        }

        this.startEngine()
      },
      goPrevious () {
        this.current--
        if (this.current < 1) {
          this.current = this.slideCount
        }
        this.animateToCurrent()
      },
      goNext () {
        this.current++
        if (this.current > this.slideCount) {
          this.current = 1
        }
        this.animateToCurrent()
      },
      goToSlide (index) {
        this.current = index + 1
        this.animateToCurrent()
      },
      prepareSlides () {
        var newSlides = []

        for (var i = 0; i < this.slides.length; i++) {
          var parent = document.createElement('div')
          parent.classList.add('veer-slide')
          parent.style.height = this.sliderMaskHeight + 'px'
          parent.style.width = this.sliderMaskWidth + 'px'

          if (this.o.mode === 'fade') {
            parent.style.position = 'absolute'
            parent.style.top = '0px'
            parent.style.left = '0px'
            parent.style.bottom = '0px'
            parent.style.right = '0px'
            parent.style.opacity = 0

            if (this.current === i + 1) {
              parent.style.opacity = 1
            }
          }

          // Append this slide to it's container
          parent.appendChild(this.slides[i])

          newSlides.push(parent)
        }

        // Clean out the slider container
        this.$refs.slides.innerHtml = ''

        for (i = 0; i < newSlides.length; i++) {
          // Append this slide container to the slider
          this.$refs.slides.appendChild(newSlides[i])
        }

        this.setSlideArray()
      },
      prepareSlider () {
        this.sliderWidth = 'auto'
        this.sliderHeight = this.sliderMaskHeight + 'px'

        if (this.o.mode === 'vertical') {
          this.$refs.slides.style.flexWrap = 'wrap'
        }
      },
      prepareSliderMask () {
        // Set the slider mask dimensions
        this.sliderMaskWidth = this.getSliderContainerWidth()
        this.sliderMaskHeight = this.getSliderContainerHeight()
      },
      getSliderContainerWidth () {
        if (typeof this.$el !== 'undefined') {
          let parentWidth = this.$el.parentElement.offsetWidth
          if (typeof parentWidth === 'undefined') {
            return this.$el.offsetWidth
          }
          return parentWidth
        }
        return 0
      },
      getSliderContainerHeight () {
        if (typeof this.$el !== 'undefined') {
          let parentHeight = this.$el.parentElement.offsetHeight
          if (typeof parentHeight === 'undefined') {
            return this.$el.offsetHeight
          }
          return parentHeight
        }
        return 0
      },
      setSlideArray () {
        this.slides = []
        let children = this.$refs.slides.childNodes
        let slideArray = Array.from(children)

        for (var i = 0; i < slideArray.length; i++) {
          if (slideArray[i].nodeType !== 3) {
            this.slides.push(slideArray[i])
          }
        }
      }
    },
    computed: {
      slideCount () {
        return this.$refs.slides.childElementCount
      }
    },
    props: {
      options: {
        type: Object
      }
    }
  }
</script>
