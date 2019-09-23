<template>
  <div class="home">
    <h1>スマホ揺れ画像ジェネレーター</h1>
    <div v-show="step==='file'">
      <div class="image-selector">
        <p>画像を選択してください</p>
        <input type="file" @change="onUpload">
      </div>
      <p class="sozaimoto">素材元：<a href="https://twitter.com/fZnHNsrh8yzwyAe/status/1175464068519362560" target="_blank">ぺろさんツイート</a><br>作った人：<a href="https://twitter.com/the_minojiro" target="_blank">みのじろー</a></p>
    </div>
    <div v-show="step==='area'">
      <p>下の画像で揺らしたいエリアを描いて、決定ボタンを押してください</p>
      <div class="area-selector">
        <div class="area-selector-inner">
          <img :src="srcImage" />
          <div class="area-selector-area" :style="areaStyle"></div>
          <div class="area-selector-over" ref="selectorOver"></div>
        </div>
      </div>
      <br>
      <span @click="step='file'">画像を選び直す</span>

      <br><br>
      <p>オーバーレイの濃さ</p>
      <input type="range" step="0.1" min="0.1" max="1" v-model="opacity">{{opacity}}<br><br>
      <button class="button" @click="generate" :disabled="areaPosition.width===0">決定</button>

    </div>
    <div class="canvas-wrap" v-show="step==='result'">
      <p>完成しました！</p>
      <canvas ref="canvas" v-show="false"></canvas>
      <p><img :src="resultImage"></p>
      <p>上の画像を長押し（PCの場合は右クリック）して保存してお使いください</p>
      <br>
      <br>
      <button class="button" @click="step='area'">再調整する</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'home',
  data() {
    return {
      step: 'file',
      srcImage: null,
      x1: 0,
      y1: 0,
      x2: 0,
      y2: 0,
      w: 1,
      h: 1,
      inTouch: false,
      touchAreaX: 0,
      touchAreaY: 0,
      areaFixed: false,
      fileResult: null,
      opacity: 0.6,
      resultImage: null,
    }
  },
  computed: {
    areaPosition() {
      const left = Math.min(this.x1, this.x2) / this.w * 100
      const right = Math.max(this.x1, this.x2) / this.w * 100
      const top = Math.min(this.y1, this.y2) / this.h * 100
      const bottom = Math.max(this.y1, this.y2) / this.h * 100
      return {
        left,
        top,
        width: right - left,
        height: bottom - top,
      }
    },
    areaStyle() {
      const pos = this.areaPosition
      return {
        left: `${pos.left}%`,
        top: `${pos.top}%`,
        width: `${pos.width}%`,
        height: `${pos.height}%`,
      }
    }
  },
  methods: {
    onUpload(e) {
      const file = e.target.files;
      const reader = new FileReader();
      reader.readAsDataURL(file[0]);
      reader.onload = (e)=> {
        this.srcImage = reader.result;
        this.fileResult = e.target.result
        this.step = 'area'
      };
    },
    updateTouchareaPos() {
      const $el = this.$refs.selectorOver
      const clientRect = $el.getBoundingClientRect()
      this.touchAreaX = clientRect.left
      this.touchAreaY = clientRect.top
      this.w = $el.clientWidth
      this.h = $el.clientHeight
    },
    generate() {
      const image = new Image();
      image.onload = () => {
        const image2 = new Image();
        image2.onload = () => {
          const canvas = this.$refs.canvas
          const ctx = canvas.getContext('2d');
          const cnvsH = image.height;
          const cnvsW = image.width;
          canvas.setAttribute('width', cnvsW);
          canvas.setAttribute('height', cnvsH);
          const pos = this.areaPosition
          ctx.drawImage(image, 0, 0, cnvsW, cnvsH);
          ctx.globalAlpha = this.opacity;
          ctx.drawImage(image2,
            pos.left / 100 * cnvsW,
            pos.top / 100 * cnvsH,
            pos.width / 100 * cnvsW,
            pos.height / 100 * cnvsH);
          this.step = 'result'
          const data = canvas.toDataURL("image/png");
          this.resultImage = data;
        }
        image2.src = './alpha.png';
      }
      image.src = this.srcImage;
    },
  },
  mounted() {
    const $el = this.$refs.selectorOver
    $el.onmousedown = e=>{
      this.updateTouchareaPos()
      this.x1 = e.clientX - this.touchAreaX
      this.y1 = e.clientY - this.touchAreaY
      this.x2 = this.x1
      this.y2 = this.y1
      this.inTouch = true
      return false
    }
    $el.onmousemove = e=>{
      if (!this.inTouch) return
      this.x2 = e.clientX - this.touchAreaX
      this.y2 = e.clientY - this.touchAreaY
      if(e.cancelable){
        e.preventDefault();
      }
    }
    $el.onmouseup = e=>{
      this.inTouch = false
      if(e.cancelable){
        e.preventDefault();
      }
    }

    $el.ontouchstart = (e)=> {
      this.updateTouchareaPos()
      this.x1 = e.touches[0].clientX - this.touchAreaX
      this.y1 = e.touches[0].clientY - this.touchAreaY
      this.x2 = this.x1
      this.y2 = this.y1
      this.inTouch = true
      if(e.cancelable){
        e.preventDefault();
      };
    }
    $el.ontouchmove = (e)=>{
      this.x2 = e.touches[0].clientX - this.touchAreaX
      this.y2 = e.touches[0].clientY - this.touchAreaY
      if(e.cancelable){
        e.preventDefault();
      };
    }
    $el.ontouchend = (e)=>{
      this.inTouch = false
      if(e.cancelable){
        e.preventDefault();
      };
    }
  },
}
</script>

<style scoped lang="scss">
h1 {
  text-align: center;
  font-weight: bold;
  margin: 40px 0;
}

.home {
  max-width: 500px;
  margin: 0 auto;
}

.image-selector {
  border: 1px dashed #ccc;
  background: #efefef;
  height: 100px;
  width: 100%;
  display: flex;
  align-items: center;
  position: relative;
  cursor: pointer;
  p {
    flex: 1;
    text-align: center;
  }
  input {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    opacity: 0;
    cursor: pointer;
    display: block;
  }
}
.area-selector {
  padding: 5px;
  background: #000;
  position: relative;
  overflow: hidden;
  img {
    width: 100%;
    vertical-align: middle;
  }
  .area-selector-area {
    position: absolute;
    left: 0;
    top: 0;
    z-index: 1;
    background: rgba(255,0,0,0.2);
    border: 2px dashed #f00;
    border-radius: 50%;
  }

  .area-selector-over {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    cursor: pointer;
    z-index: 2;
  }
}
.canvas-wrap {
  img {
    width: 100%;
  }
}
.button {
  width: 100%;
  font-size: 20px;
  font-weight: bold;
}
.sozaimoto {
  text-align: center;
  font-size: 12px;
  margin-top: 20px;
}
</style>
