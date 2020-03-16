<template>
  <div id="app">
    <div class="sky" id="app" ref="sky" :style="skyStyles">
      <div class="sun planet" :style="[sunStyles]"></div>
      <div class="moon planet" ref="moon" :style="[moonStyles]" @mousedown="moonPickUp" @touchstart="moonPickUpTouch"></div>
      <!-- <ul>
        <li>Distances X: {{ distances.x }}</li>
        <li>Distances Y: {{ distances.y }}</li>
        <li>Distances Hyp: {{ distances.hyp }}</li>
        <li>Angle: {{ distances.angle }}</li>
        <li>Touched: {{ touched }}</li>
      </ul> -->
    </div>
  </div>
</template>

<script>

function mapRange (value, low1, high1, low2, high2) {
  return low2 + (high2 - low2) * (value - low1) / (high1 - low1)
}

function radiansToDegrees (radians) {
  var pi = Math.PI
  return radians * (180 / pi)
}

function degreesToRadians (degrees) {
  var pi = Math.PI
  return degrees * (pi / 180)
}

export default {
  name: 'App',
  components: {
  },
  data: function () {
    const self = this
    return {
      touched: 'not touched',
      show: false,
      skyHeight: 0,
      skyWidth: 0,
      transition: true,
      moonPosition: {
        x: 0,
        y: 0
      },
      orbit: {
        x: 0,
        y: 0,
        timer: {
          timer: null,
          start: function () {
            this.timer = setInterval(self.updateOrbit, 100)
          },
          stop: function () {
            clearInterval(this.timer)
          }
        }
      },
      orbitTimer: null
    }
  },
  mounted () {
    this.$nextTick(function () {
      window.addEventListener('resize', this.getWindowWidth)
      window.addEventListener('resize', this.getWindowHeight)

      // Init
      this.getWindowWidth()
      this.getWindowHeight()

      this.moonPosition.x = this.moonStartingPoint.x
      this.moonPosition.y = this.moonStartingPoint.y

      this.orbit.timer.start()
    })
  },
  props: {
    bgColor: {
      type: String,
      default: '#0099CC'
    },
    height: {
      type: Number,
      default: 100
    },
    planetHeight: {
      type: Number,
      default: 100
    }
  },
  computed: {
    sunDiameter () { return this.skyWidth / 3.6 },
    moonDiameter () { return this.sunDiameter - 2 },
    moonStartingPoint () {
      return {
        x: this.sunDiameter / 2,
        y: this.skyHeight / 2
      }
    },
    sunPosition () {
      return {
        x: this.skyWidth / 2,
        y: this.skyHeight / 2
      }
    },
    skyBG () {
      return {
        pointA: {
          r: mapRange(this.distances.hyp, this.skyWidth / 2, 0, 220, 228),
          g: mapRange(this.distances.hyp, this.skyWidth / 2, 0, 244, 170),
          b: mapRange(this.distances.hyp, this.skyWidth / 2, 0, 248, 116)
        },
        pointB: {
          r: mapRange(this.distances.hyp, this.skyWidth / 2, 0, 160, 20),
          g: mapRange(this.distances.hyp, this.skyWidth / 2, 0, 230, 3),
          b: mapRange(this.distances.hyp, this.skyWidth / 2, 0, 239, 3)
        }
      }
    },
    distances () {
      const x = this.sunPosition.x - this.moonPosition.x
      const y = this.sunPosition.y - this.moonPosition.y

      const angle = radiansToDegrees(Math.atan2(x, y))

      return {
        x: x,
        y: y,
        hyp: (Math.sqrt((x * x) + (y * y))),
        angle: -angle
      }
    },
    sunStyles () {
      return {
        background: 'radial-gradient(50% 50% at 50% 50%, #FFD600 0%, #FEEB8B 100%)',
        boxShadow: '0px 0px 177px #FFFA78',
        top: `${this.sunPosition.y}px`,
        left: `${this.sunPosition.x}px`,
        height: `${this.sunDiameter}px`,
        width: `${this.sunDiameter}px`,
        marginTop: `-${this.sunDiameter / 2}px`,
        marginLeft: `-${this.sunDiameter / 2}px`
      }
    },
    moonStyles () {
      const blur = mapRange(this.distances.hyp, this.skyWidth / 2, 0, 10, this.sunDiameter / 8)
      const y = mapRange(this.distances.hyp, this.skyWidth / 4, 0, -this.sunDiameter / 40, 0)
      const alpha = mapRange(this.distances.hyp, this.skyHeight / 2, 0, 0.6, 1)
      return {
        backgroundColor: `rgb(${this.skyBG.pointB.r}, ${this.skyBG.pointB.g}, ${this.skyBG.pointB.b})`,
        boxShadow: `inset 0px ${y}px ${blur}px rgba(255, 255, 255, ${alpha}`,
        top: `${this.moonPosition.y}px`,
        left: `${this.moonPosition.x}px`,
        height: `${this.moonDiameter}px`,
        width: `${this.moonDiameter}px`,
        marginTop: `-${this.sunDiameter / 2}px`,
        marginLeft: `-${this.sunDiameter / 2}px`,
        transform: `rotate(${this.distances.angle}deg)`,
        transition: this.transition ? 'all 0.5s' : '',
        cursor: this.transition ? 'grab' : 'grabbing'
      }
    },
    skyStyles () {
      const gradientRadius = mapRange(this.distances.hyp, 0, this.skyHeight, 60, 100)
      return {
        background: `radial-gradient(50% 50% at 50% 50%, rgb(${this.skyBG.pointA.r}, ${this.skyBG.pointA.g}, ${this.skyBG.pointA.b}) 0%, rgb(${this.skyBG.pointB.r}, ${this.skyBG.pointB.g}, ${this.skyBG.pointB.b})  ${gradientRadius}%), #FFFFFF`
      }
    }
  },
  methods: {
    getWindowHeight () {
      this.skyHeight = this.$refs.sky.clientHeight
    },
    getWindowWidth () {
      this.skyWidth = this.$refs.sky.clientWidth
    },
    moonPickUp () {
      window.addEventListener('mousemove', this.moonDrag)
      window.addEventListener('mouseup', this.moonDrop)

      this.orbit.timer.stop()

      this.transition = false
    },
    moonPickUpTouch () {
      this.touched = 'touched'
      this.$refs.moon.addEventListener('touchmove', this.moonDragTouch, false)
      window.addEventListener('touchend', this.moonDropTouch, false)

      this.transition = false

      this.orbit.timer.stop()
    },
    moonDrop () {
      this.transition = true

      window.removeEventListener('mousemove', this.moonDrag)
      window.removeEventListener('mouseup', this.moonDrop)

      this.orbit.timer.start()
    },
    moonDropTouch () {
      this.transition = true
      this.touched = 'untouched'
      this.$refs.moon.removeEventListener('touchmove', this.moonDragTouch)
      window.removeEventListener('touchend', this.moonDropTouch)

      this.orbit.timer.start()
    },
    moonDrag (e) {
      if (this.moonPosition.x < this.skyWidth) {
        this.moonPosition.x += e.movementX
      }

      if (this.moonPosition.y < this.skyHeight) {
        this.moonPosition.y += e.movementY
      }
    },
    moonDragTouch (e) {
      if (this.moonPosition.x < this.skyWidth - (this.sunDiameter / 2)) {
        this.moonPosition.x = e.touches[0].pageX
      }

      if (this.moonPosition.y < this.skyHeight - (this.sunDiameter / 2)) {
        this.moonPosition.y = e.touches[0].pageY
      }
    },
    updateOrbit () {
      this.orbit.y++
      this.orbit.x = -Math.cos(degreesToRadians(this.orbit.y))
      this.moonPosition.y = this.moonStartingPoint.y
      this.moonPosition.x = mapRange(this.orbit.x, -1, 1, this.moonStartingPoint.x, this.skyWidth - (this.moonDiameter / 2))
    }
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.getWindowWidth)
    window.removeEventListener('resize', this.getWindowHeight)
  }
}
</script>

<style lang="scss" scoped>
ul {
  margin:0;
  padding:0;
}
.planet{
  border-radius: 50%;
  position:absolute;
}

.sky {
  height: 100vh;
  width: 100vw;
  max-height:100vh;
  max-width:100vw;
  overflow:hidden;
}

body, html {
  height:100%;
  width:100%;
}

</style>
