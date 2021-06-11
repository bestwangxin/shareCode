<template>
  <div id="app">
    <canvas id="myCanvas"></canvas>
    <div class="footer">
      <div class="buttonWrap">
          <div @click="clearArea()">重置</div>
          <div @click="saveImageInfo()">提交</div>
        </div>
      <!--<div>
        <input v-model="strokeStyle" type="color">
      </div>-->
    </div>
  </div>
</template>

<script>
import {Dialog} from 'vant'
import {upload} from "@/api/system";
import {approve} from '@/api/todoTask'
export default {
  data() {
    return {
      touchPressed: false,
      ctx: null,
      strokeStyle: '#000',
      lineWidth: 4,
      lastX: null,
      lastY: null,
      canvas: null,
      img: null,
      text:this.$route.query.text,
      radio:this.$route.query.radio,
      toDoTaskType:this.$route.query.toDoTaskType,
      taskId:this.$route.query.taskId
    }
  },
  mounted() {
    this.$nextTick(() => {
      let canvas = document.getElementById('myCanvas');
      this.canvas = canvas;
      this.ctx = canvas.getContext('2d');
      let winW = window.innerWidth;
      let winH = window.innerHeight;
      canvas.width = winW;
      canvas.height = winH;
      this.Init()
    });
  },
  methods: {
    Init() {
      this.canvas.addEventListener(
          'touchstart',
          (event) => {
            if (event.targetTouches.length == 1) {
              event.preventDefault(); // 阻止浏览器默认事件，重要
              var touch = event.targetTouches[0];
              this.touchPressed = true;
              this.draw(touch.pageX - this.canvas.offsetLeft, touch.pageY - this.canvas.offsetTop, false);
            }
          },
          false
      );
      this.canvas.addEventListener(
          'touchmove',
          (event) => {
            if (event.targetTouches.length == 1) {
              event.preventDefault(); // 阻止浏览器默认事件，重要
              var touch = event.targetTouches[0];
              if (this.touchPressed) {
                this.draw(touch.pageX - this.canvas.offsetLeft, touch.pageY - this.canvas.offsetTop, true);
              }
            }
          },
          false
      );
      this.canvas.addEventListener(
          'touchend',
          (event) => {
            if (event.targetTouches.length == 1) {
              event.preventDefault(); // 阻止浏览器默认事件，防止手写的时候拖动屏幕，重要
              this.touchPressed = false;
            }
          },
          false
      );
    },
    goBack() {
      this.$router.go(-1);
    },
    draw(x, y, isDown) {
      let ctx = this.ctx
      if (isDown) {
        ctx.beginPath();
        ctx.strokeStyle = this.strokeStyle;
        ctx.lineWidth = this.lineWidth;
        ctx.lineJoin = "round";
        ctx.moveTo(this.lastX, this.lastY);
        ctx.lineTo(x, y);
        ctx.closePath();
        ctx.stroke();
      }
      this.lastX = x;
      this.lastY = y;
    },
    clearArea() {
      this.ctx.setTransform(1, 0, 0, 1, 0, 0);
      this.ctx.clearRect(0, 0, this.ctx.canvas.width, this.ctx.canvas.height);
    },
    saveImageInfo() {
      Dialog.confirm({
        title: '提示',
        message: '确认签收么',
      })
          .then(() => {
            this.dataURItoBlob(this.canvas.toDataURL());
          })
          .catch(() => {
          });
    },
    dataURItoBlob(dataURI) {
      // base64转buffer
      let byteString = atob(dataURI.split(",")[1]);
      let mimeString = dataURI.split(",")[0].split(":")[1].split(";")[0];
      let ab = new ArrayBuffer(byteString.length);
      let ia = new Uint8Array(ab);
      for (let i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
      }
      this.img = new Blob([ab], {type: mimeString});
      let file = new File([this.img], new Date().getTime() + '.png');
      let params = new FormData();
      params.append("files", file);
      upload(params).then(res=>{
        approve({approveStatus:this.radio,comments:this.text,imageUrl:res.data[0].url,toDoTaskType:this.toDoTaskType,taskId:this.taskId}).then(res=>{
          if(res.code==0){
            this.$toast('签收成功')
          }
        })
      })
    },
  }
};
</script>

<style scoped lang="scss">
.footer {
  height: 55px;
  position: fixed;
  width: 100%;
  left: 0;
  bottom: 0.1rem;
  z-index: 10;
  display: flex;
  .buttonWrap {
        width: calc(100% );
        height: 0.46rem;
        ;
        margin-bottom: 0.15rem;
        div {
          width: calc(50% - 0.2rem);
          font-size: 0.18rem;
          height: 0.44rem;
          line-height: 0.44rem;
          &:nth-child(1) {
            background: #ffffff;
            border: 1px solid #0482ff;
            border-radius: 0.05rem;
            float: left;
            color:#0482FF;
            margin-right: 0.1rem;
          }
          &:nth-child(2) {
            background: #0482ff;
            border-radius: 0.05rem;
            float: right;
            color: white;
          }
        }
      }
  > div {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
  }

}

</style>