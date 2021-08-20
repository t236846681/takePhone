<template>
  <div class="camera_outer" ref="camera_outer">
    <canvas
        style="position: absolute;left: -99999px; top: -9999px;"
        :width="canvasWidth"
        :height="canvasHeight"
        id="canvasCamera">
    </canvas>
    <!--    <p class="square"></p>-->
    <div class="camera_outer-video" ref="camera_outer-video">
      <video
          ref="photo"
          webkit-playsinline="true"
          x-webkit-airplay="true"
          x5-playsinline="true"
          playsinline="true"
          id="videoCamera"
          :width="videoWidth"
          :height="videoHeight"
          autoplay>
      </video>

      <div class="shade">
        <img :src="'https://image.xinfud.com/image/xinluobh5/back.png?v=1'" alt="">
      </div>
    </div>
    <div class="board" ref="board">
      <div class="border-wrap">
        <div class="circle-wrap">
          <div class="circle" @click="setImage()"></div>
        </div>
        <span>{{ print }}5</span>
      </div>
    </div>

    <div v-if="imgSrc" class="img_bg_camera">
      <img :src="imgSrc" alt="" class="tx_img"/>
    </div>
  </div>
</template>
<script>

export default {
  data() {
    return {
      showLoading: false,
      canvasWidth: 600,
      canvasHeight: 800,
      videoWidth: 600,
      videoHeight: 800,
      imgSrc: "",
      thisCancas: null,
      thisContext: null,
      thisVideo: null,
      print: {}
    };
  },
  // components: {
  //   Loading,
  // },
  mounted() {
    this.getCompetence();
  },
  methods: {
    close() {
      this.$router.go(-1);
      // this.$emit('addImg',"");
    },      // 调用权限（打开摄像头功能）
    getCompetence() {
      let _this = this;
      this.videoWidth = this.$refs.photo.clientWidth  //宽度
      let height1 = this.$refs.camera_outer.offsetHeight;
      let height2 = this.$refs.board.offsetHeight;
      this.videoHeight = height1 - height2;
      // this.print = {
      //   height1: height1,
      //   height2: height1 - height2,
      //   photo: this.$refs.photo.offsetHeight
      // }
      // this.$refs['camera_outer-video'].style.height = (height1 - height2) + 'px';
      this.thisCancas = document.getElementById("canvasCamera");

      this.thisVideo = document.getElementById("videoCamera");
      this.thisCancas.width = this.videoWidth
      this.thisCancas.height = this.videoHeight
      this.thisContext = this.setupCanvas(this.thisCancas);
      // 旧版本浏览器可能根本不支持mediaDevices，我们首先设置一个空对象
      if (navigator.mediaDevices === undefined) {
        navigator.mediaDevices = {};
      }
      // 一些浏览器实现了部分mediaDevices，我们不能只分配一个对象
      // 使用getUserMedia，因为它会覆盖现有的属性。
      // 这里，如果缺少getUserMedia属性，就添加它。
      if (navigator.mediaDevices.getUserMedia === undefined) {
        navigator.mediaDevices.getUserMedia = function (constraints) {
          // 首先获取现存的getUserMedia(如果存在)
          var getUserMedia =
              navigator.webkitGetUserMedia ||
              navigator.mozGetUserMedia ||
              navigator.getUserMedia;
          // 有些浏览器不支持，会返回错误信息
          // 保持接口一致
          if (!getUserMedia) {
            return Promise.reject(
                new Error("getUserMedia is not implemented in this browser")
            );
          }
          // 否则，使用Promise将调用包装到旧的navigator.getUserMedia
          return new Promise(function (resolve, reject) {
            getUserMedia.call(navigator, constraints, resolve, reject);
          });
        };
      }
      if (window.stream) {//一进来的时候判断是否开着摄像头
        window.stream.getTracks().forEach(track => {
          track.stop();
        });
      }
      this.$nextTick(() => {
        let constraints = {//配置默认前置摄像头,以及手机上摄像头取到的画面宽高
          audio: false,
          video: {
            width: {ideal: this.videoHeight * 2},
            height: {ideal: this.videoWidth * 2},
            // width:  550 ,
            // height: 300 ,
            // transform: "rotateX(-180deg)",
            sourceId: 'default',
            // facingMode:  { exact: "user" }
            facingMode: {exact: 'environment'} // 后置
            // facingMode: "user" // 前置
          },
        };
        navigator.mediaDevices
            .getUserMedia(constraints)
            .then(function (stream) {
              // 旧的浏览器可能没有srcObject
              if ("srcObject" in _this.thisVideo) {
                _this.$nextTick(() => {
                  _this.thisVideo.srcObject = stream;//that.video是video标签节点，请自行获取节点，updated周期里可以拿到！
                  _this.thisVideo.play();
                });
              } else {
                // 避免在新的浏览器中使用它，因为它正在被弃用。
                _this.thisVideo.src = window.URL.createObjectURL(stream);
              }
              // _this.thisVideo.onloadedmetadata = function (e) {
              //   _this.thisVideo.play();
              // };
            })
            .catch((err) => {
              console.log("错误");
              console.log(err);
            });
      })
    },
    //  绘制图片（拍照功能）
    setImage() {
      let _this = this;
      // 点击，canvas画图
      _this.thisContext.scale(1, 1)//如果你上传图片是镜像的加上这段，我这边图片上传后会镜像
      // _this.thisContext.rotate(90*Math.PI/180)
      // console.warn(_this.videoHeight, _this.videoWidth)
      // console.warn(_this.thisVideo)
      _this.thisContext.drawImage(
          _this.thisVideo,
          0,
          // -parseInt(_this.videoWidth),  //镜像翻转要往负方向移动图片的距离
          0,
          _this.videoWidth,
          _this.videoHeight
      );
      // );
      // 获取图片base64链接
      var image = this.thisCancas.toDataURL("image/jpeg", 1);
      _this.imgSrc = image;
      // let file = _this.dataURLtoFile(image, 'pic')
      // _this.upload(file);
      // 停止摄像机
      _this.thisVideo.pause();
      this.stopNavigator();
    },
    setupCanvas(canvas) {
      // Get the device pixel ratio, falling back to 1.
      var dpr = 2;
      // console.warn('window.devicePixelRatio', window.devicePixelRatio)
      // Get the size of the canvas in CSS pixels.
      var rect = canvas.getBoundingClientRect();
      // Give the canvas pixel dimensions of their CSS
      // size * the device pixel ratio.
      canvas.width = rect.width * dpr;
      canvas.height = rect.height * dpr;
      var ctx = canvas.getContext('2d');
      // Scale all drawing operations by the dpr, so you
      // don't have to worry about the difference.
      ctx.scale(dpr, dpr);
      return ctx;
    },
    // playCanvas() {
    //   let _this = this
    //   _this.thisContext.scale(1, 1)//如果你上传图片是镜像的加上这段，我这边图片上传后会镜像
    //   // _this.thisContext.drawImage(
    //   //     _this.thisVideo,
    //   //     0,
    //   //     0,
    //   //     _this.videoWidth,
    //   //     _this.videoHeight
    //   // );
    //    window.setInterval(function () {
    //     _this.thisContext.drawImage(_this.thisVideo, 0, 0,
    //         _this.videoWidth,
    //         _this.videoHeight)
    //   }, 20)
    // },
    // base64转文件
    dataURLtoFile(dataurl, filename) {
      var arr = dataurl.split(",");
      var mime = arr[0].match(/:(.*?);/)[1];
      var bstr = atob(arr[1]);
      var n = bstr.length;
      var u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }
      return new File([u8arr], filename, {
        type: mime
      });
    },      // 上传图片
    upload(file) {
      this.showLoading = true
      var formdata = new FormData();
      formdata.append("file", file, file.name)
      let _this = this;
      _this.axios.post("https://xxxxxxx/upload", formdata).then(res => {
        if (res.data.code == "200") {
          this.showLoading = false;
          // let _this = this;
          // _this.$emit('addImg',res.data.data);
          // setTimeout(function(){
          //   _this.$router.go(-1);
          // },300)
        } else {
          this.showLoading = false;
          alert(res.msg);
        }
      });
    },
    // 关闭摄像头
    stopNavigator() {
      this.thisVideo.srcObject.getTracks()[0].stop();
    },
  }
  ,
}
;
</script>
<style lang="scss" scoped>
#videoCamera {
  width: 100%;
  height: 85vh;
  transform: rotateY(0deg);
}

body {
  margin: 0;
}

.camera_outer-video {
  height: 85vh;
  position: relative;
  width: 100%;
}

body {
  margin: 0;
}

#videoCamera {
  width: 100%;
}

.close {
  position: absolute;
  top: 0.3rem;
  right: 0.3rem;
}

.vux-x-icon {
  fill: #000;
}

/*遮罩*/
.shade {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  //padding: 20px;
  //box-sizing: border-box;
  img {
    width: 85%;
  }
}

.square {
  margin: 0;
  height: 15vh;
  background: #fff;
}

//.shade img {
//  width: 100%;
//}

.board {
  background: #4b4956;
  height: 15vh;
  width: 100%;
  //margin-top: -5px;

  .border-wrap {
    width: 100%;
    position: absolute;
    bottom: 0;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
  }

  .tip {
    width: 100%;
    height: 1em;
    text-align: center;
    font-size: 18px;
    color: #fff;
  }

  .circle-wrap {
    width: 100%;
    height: 120px;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;

    .circle {
      bottom: 1rem;
      width: 1.5rem;
      height: 1.5rem;
      background: #fff;
      border: 5px solid #ccc;
      border-radius: 50%; /*transform: all 1s ;*/
    }

    .circle:active {
      width: 1rem;
      height: 1rem;
    }
  }
}

.camera_outer {
  height: 100vh;
  position: relative;
  //overflow: hidden;
  background-size: 100%;

  //{
  //    -moz-transform: scaleX(-1);
  //    -webkit-transform: scaleX(-1);
  //    -o-transform: scaleX(-1);
  //    transform: scaleX(-1);
  //  }

  .btn_camera {
    position: absolute;
    bottom: 4px;
    left: 0;
    right: 0;
    height: 50px;
    background-color: rgba(0, 0, 0, 0.3);
    line-height: 50px;
    text-align: center;
    color: #ffffff;
  }

  .bg_r_img {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    top: 0;
  }

  .img_bg_camera {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 85vh;
    z-index: 9999;

    img {
      width: 100%;
      //width: 300px;
      //height: 300px;
    }
  }
}
</style>
