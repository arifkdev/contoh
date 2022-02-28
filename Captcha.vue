<template>
  <div class="puzzle-captcha pb-5">
    <canvas :width="width" :height="height" ref="canvasRef"></canvas
    ><canvas
      class="puzzle-block"
      :width="width"
      :height="height"
      ref="blockRef"
    ></canvas>
    <i class="refreshIcon fa fa-redo cursor" @click="onReset"></i>
    <div class="sliderContainer">
      <div class="sliderbg"></div>
      <div class="sliderMask" style="width: 0px">
        <div class="slider" style="left: 0px">
          <i class="fa fa-arrow-right sliderIcon"></i>
        </div>
      </div>
      <span class="sliderText">{{ slideText }}</span>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    label: {
      type: String,
      default() {
        return 'Geser ke Kanan'
      },
    },
  },
  data() {
    return {
      slideText: 'Geser ke Kanan',
      src: null,
      canvas: null,
      block: null,
      img: null,
      width: 280, // canvas宽度
      height: 155, // canvas高度
      PI: Math.PI,
      sliderL: 42,
      sliderR: 9,
      offset: 5,
      isIE: null,
      isMouseDown: false,
      originX: 0,
      clientY: 0,
      slider: null,
      sliderContainer: null,
      sliderMask: null,
      blockClass: null,
      trail: [],
    }
  },
  mounted() {
    this.onGenerate()
    this.slider = document.querySelector('.slider')

    this.sliderContainer = document.querySelector('.sliderContainer')
    this.blockClass = document.querySelector('.puzzle-block')
    this.sliderMask = document.querySelector('.sliderMask')
    let self = this
    this.slider.addEventListener('mousedown', function (e) {
      self.handleDragStart(e)
    })
    this.slider.addEventListener('touchstart', function (e) {
      self.handleDragStart(e)
    })

    this.slider.addEventListener('mousemove', function (e) {
      self.handleDragMove(e)
    })
    this.slider.addEventListener('touchmove', function (e) {
      self.handleDragMove(e)
    })
    this.slider.addEventListener('mouseup', function (e) {
      self.handleDragEnd(e)
    })
    this.slider.addEventListener('touchend', function (e) {
      self.handleDragEnd(e)
    })
  },
  methods: {
    onGenerate() {
      this.src = '/captcha/Pic' + Math.round(Math.random() * 18) + '.jpg'

      this.canvas = this.$refs.canvasRef.getContext('2d')
      this.block = this.$refs.blockRef.getContext('2d')

      this.img = new Image()
      this.img.src = this.src
      var L = this.sliderL + this.sliderR * 2 + 3
      this.isIE = window.navigator.userAgent.indexOf('Trident') > -1
      this.img.onload = function (e) {
        this.x = this.getRandomNumberByRange(L + 10, this.width - (L + 10))
        this.y = this.getRandomNumberByRange(
          10 + this.sliderR * 2,
          this.height - (L + 10)
        )
        this.drawImg(this.canvas, 'fill')
        this.drawImg(this.block, 'clip')

        this.canvas.drawImage(this.img, 0, 0, this.width - 2, this.height)
        this.block.drawImage(this.img, 0, 0, this.width - 2, this.height)
        var y = this.y - this.sliderR * 2 - 1
        var ImageData = this.block.getImageData(this.x - 3, y, L, L)
        this.$refs.blockRef.width = L
        this.block.putImageData(ImageData, 0, y + 1)
        this.slideText = this.label
      }.bind(this)
    },
    drawImg(ctx, operation) {
      var l = this.sliderL
      var r = this.sliderR
      var PI = this.PI
      var x = this.x
      var y = this.y
      ctx.beginPath()
      ctx.moveTo(x, y)
      ctx.arc(x + l / 2, y - r + 2, r, 0.72 * PI, 2.26 * PI)
      ctx.lineTo(x + l, y)
      ctx.arc(x + l + r - 2, y + l / 2, r, 1.21 * PI, 2.78 * PI)
      ctx.lineTo(x + l, y + l)
      ctx.lineTo(x, y + l)
      ctx.arc(x + r - 2, y + l / 2, r + 0.4, 2.76 * PI, 1.24 * PI, true)
      ctx.lineTo(x, y)
      ctx.lineWidth = 2
      ctx.fillStyle = 'rgba(255, 255, 255, 0.7)'
      ctx.strokeStyle = 'rgba(255, 255, 255, 0.7)'
      ctx.stroke()
      ctx[operation]()
      ctx.globalCompositeOperation = this.isIE ? 'xor' : 'destination-over'
    },
    getRandomNumberByRange(start, end) {
      return Math.round(Math.random() * (end - start) + start)
    },
    handleDragStart(e) {
      this.originX = e.clientX || e.touches[0].clientX
      this.originY = e.clientY || e.touches[0].clientY
      this.isMouseDown = true
    },
    handleDragMove(e) {
      if (!this.isMouseDown) return false
      var eventX = e.clientX || e.touches[0].clientX
      var eventY = e.clientY || e.touches[0].clientY
      var moveX = eventX - this.originX
      var moveY = eventY - this.originY
      if (moveX < 0 || moveX + 40 > this.width) return false
      this.slider.style.left = moveX - 1 + 'px'
      var blockLeft = ((this.width - 40 - 20) / (this.width - 40)) * moveX

      this.blockClass.style.left = blockLeft + 'px'

      this.sliderContainer.classList.add('sliderContainer_active')
      this.sliderMask.style.width = moveX + 4 + 'px'
      this.trail.push(Math.round(moveY))
    },
    handleDragEnd(e) {
      if (!this.isMouseDown) return false
      this.isMouseDown = false
      var eventX = e.clientX || e.changedTouches[0].clientX
      if (eventX === this.originX) return false
      this.sliderContainer.classList.remove('sliderContainer_active')
      // this.trail = trail;
      var data = this.verify()
      if (data.spliced && data.verified) {
        this.sliderContainer.classList.add('sliderContainer_success')
        this.$emit('success', true)
        setTimeout(
          function () {
            this.onReset()
          }.bind(this),
          750
        )
      } else {
        this.sliderContainer.classList.add('sliderContainer_fail')
        this.$emit('fail', true)
        setTimeout(
          function () {
            this.onReset()
          }.bind(this),
          750
        )
      }
    },
    verify() {
      var arr = this.trail // 拖动时y轴的移动距离
      var left = parseInt(this.blockClass.style.left)
      var verified = false
      var sum = function (x, y) {
        return x + y
      }
      var square = function (x) {
        return x * x
      }
      var average = arr.reduce(sum) / arr.length
      var deviations = arr.map(function (x) {
        return x - average
      })
      var stddev = Math.sqrt(deviations.map(square).reduce(sum) / arr.length)
      verified = stddev !== 0

      return {
        spliced: Math.abs(left - this.x) < this.offset,
        verified: verified,
      }
    },

    onReset() {
      this.sliderContainer.classList.remove('sliderContainer_fail')
      this.sliderContainer.classList.remove('sliderContainer_success')
      this.slider.style.left = 0
      this.blockClass.style.left = 0
      this.sliderMask.style.width = 0
      this.canvas.clearRect(0, 0, this.width, this.height)
      this.block.clearRect(0, 0, this.width, this.height)
      this.$refs.blockRef.width = this.width
      this.onGenerate()
    },
  },
}
</script>

<style>
.puzzle-captcha {
  position: relative;
  width: 280px;
  margin: 0px auto;
}
.puzzle-block {
  position: absolute;
  left: 0;
  top: 0;
}

.sliderContainer {
  position: relative;
  text-align: center;
  line-height: 40px;
  background: #f7f9fa;
  color: #45494c;
  border-radius: 2px;
}

.sliderbg {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  background-color: #f7f9fa;
  height: 40px;
  border-radius: 2px;
  border: 1px solid #e6e8eb;
}

.sliderContainer_active .slider {
  top: -1px;
  border: 1px solid #1991fa;
}

.sliderContainer_active .sliderMask {
  border-width: 1px 0 1px 1px;
}

.sliderContainer_success .slider {
  top: -1px;
  border: 1px solid #52ccba;
  background-color: #52ccba !important;
}

.sliderContainer_success .sliderMask {
  border: 1px solid #52ccba;
  border-width: 1px 0 1px 1px;
  background-color: #d2f4ef;
}

.sliderContainer_success .sliderIcon:before {
  content: '\f00c';
}

.sliderContainer_fail .slider {
  top: -1px;
  border: 1px solid #f57a7a;
  background-color: #f57a7a !important;
}

.sliderContainer_fail .sliderMask {
  border: 1px solid #f57a7a;
  background-color: #fce1e1;
  border-width: 1px 0 1px 1px;
}

.sliderContainer_fail .sliderIcon:before {
  content: '\f00d';
}

.sliderContainer_active .sliderText,
.sliderContainer_success .sliderText,
.sliderContainer_fail .sliderText {
  display: none;
}

.sliderMask {
  position: absolute;
  left: 0;
  top: 0;
  height: 40px;
  border: 0 solid #1991fa;
  background: #d1e9fe;
  border-radius: 2px;
}

.slider {
  position: absolute;
  top: 0;
  left: 0;
  width: 40px;
  height: 40px;
  background: #fff;
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.3);
  cursor: pointer;
  transition: background 0.2s linear;
  border-radius: 2px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.slider:hover {
  background: #1991fa;
}

.slider:hover .sliderIcon {
  background-position: 0 -13px;
}

.sliderText {
  position: relative;
}

.refreshIcon {
  position: absolute;
  right: 0;
  top: 0;
  cursor: pointer;
  margin: 6px;
  color: #fff;
  font-size: 1rem;
  z-index: 5;
  transition: color 0.3s linear;
}

.refreshIcon:hover {
  color: rgb(214, 214, 214);
}
</style>