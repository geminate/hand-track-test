<template>
  <div class="home-page">
    <div class="show-container">
      <video ref="video"></video>
      <canvas ref="canvas"></canvas>
      <div class="item-container">
        <div class="box" :style="{top:y +'px',left:x +'px'}"></div>
      </div>
    </div>
    <br>

    <div>
      x:{{x}}
      <br>
      y:{{y}}
    </div>

    <br>
    <button class="btn" @click="toggleVideo">{{btnText}}</button>
  </div>
</template>

<script>
  import * as handTrack from 'handtrackjs'

  export default {
    name: 'index',
    data () {
      return {
        btnText: '开始', // 按钮文字
        isVideo: false, // 当前状态
        model: null,
        video: null,
        canvas: null,
        x: 50,
        y: 0
      }
    },
    methods: {

      // 启动、关闭切换
      toggleVideo () {
        if (this.isVideo) {
          this.stopVideo()
        } else {
          this.startVideo()
        }
      },

      // 启动
      startVideo () {
        handTrack.startVideo(this.video).then((status) => {
          if (status) {
            this.isVideo = true
            this.btnText = '关闭'
            this.runDetection()
          }
        })
      },

      // 停止
      stopVideo () {
        handTrack.stopVideo(this.video)
        this.isVideo = false
      },

      detect () {
        this.model.detect(this.video).then(predictions => {
          this.model.renderPredictions(predictions, this.canvas, this.canvas.getContext('2d'), this.video)
          if (this.isVideo) {
            requestAnimationFrame(this.detect)
          }
          if (predictions.length > 0) {
            this.changeText(predictions[0].bbox)
          }
        })
      },

      changeText (value) {
        this.x = value[0] + value[2] / 2
        this.y = value[1] + value[3] / 2
      }
    },
    mounted () {
      this.video = this.$refs.video
      this.canvas = this.$refs.canvas

      const modelParams = {
        flipHorizontal: true,
        maxNumBoxes: 2,
        iouThreshold: 0.5,
        scoreThreshold: 0.79
      }

      handTrack.load(modelParams).then(model => {
        this.model = model
      })
    }
  }
</script>

<style scoped lang="less">
  .home-page {
    width: 100vw;
    height: 100vh;

    .show-container {
      overflow: hidden;

      video, .item-container {
        width: 500px;
        margin: 5px;
        height: 500px;
        border: 1px solid gray;
        float: left;
        position: relative;
      }

      canvas {
        position: absolute;
        width: 500px;
        margin: 5px;
        height: 500px;
        left: 0;
      }

      .box {
        width: 20px;
        height: 20px;
        position: absolute;
        background: deepskyblue;
      }
    }

    .btn {
      width: 100px;
      height: 30px;
    }

  }
</style>
