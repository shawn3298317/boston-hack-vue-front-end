<template>
  <div class="content">
    <div class="container-fluid">
      <card>
        <div class="row justify-content-center video-container">
          <div class="col-md-10 text-center">
            <h5>Object Detection Demo for FasterRCNN</h5>
            
            <div class="justify-content-center">
              <video ref="video" id="video" width="640" height="480" autoplay/>
              <canvas ref="canvas" id="canvas" width="640" height="480"></canvas>
            </div>
            
            <!-- <div><button id="snap" v-on:click="capture()">Snap Photo</button></div> -->
            
            <!-- <ul>
                <li v-for="c in captures">
                    <img v-bind:src="c" height="50" />
                </li>
            </ul> -->

          </div>
          
        </div>
        <br>
        <br>
        <!-- <div class="places-buttons">
          <div class="row justify-content-center">
            <div class="col-6 text-center">
              <h5>Notifications Places
                <p class="category">Click to view notifications</p>
              </h5>
            </div>
          </div>
          <div class="row justify-content-center">
            <div class="col-md-3 col-md-offset-1">
              <button class="btn btn-default btn-block" @click="notifyVue('top', 'left')">Top Left</button>
            </div>
            <div class="col-md-3">
              <button class="btn btn-default btn-block" @click="notifyVue('top', 'center')">Top Center</button>
            </div>
            <div class="col-md-3">
              <button class="btn btn-default btn-block" @click="notifyVue('top', 'right')">Top Right</button>
            </div>
          </div>
          <div class="row justify-content-center">
            <div class="col-md-3 col-md-offset-1">
              <button class="btn btn-default btn-block" @click="notifyVue('bottom', 'left')">Bottom Left</button>
            </div>
            <div class="col-md-3">
              <button class="btn btn-default btn-block" @click="notifyVue('bottom', 'center')">Bottom Center</button>
            </div>
            <div class="col-md-3">
              <button class="btn btn-default btn-block" @click="notifyVue('bottom', 'right')">Bottom Right</button>
            </div>

          </div>
        </div> -->
      </card>
    </div>
  </div>
</template>
<!-- Load TensorFlow.js-->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.0.0/dist/tf.min.js"></script>
<!-- Load the coco-ssd model. -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>
<script>
  import Card from 'src/components/Cards/Card.vue'
  import * as cocoSsd from '@tensorflow-models/coco-ssd';

  export default {
    components: {
      Card
    },
    data () {
      return {
        type: ['', 'info', 'success', 'warning', 'danger'],
        notifications: {
          topCenter: false
        },
        video: {},
        canvas: {},
        captures: {}
      }
    },
    methods: {
      notifyVue (verticalAlign, horizontalAlign) {
        const color = Math.floor((Math.random() * 4) + 1)
        this.$notifications.notify(
          {
            message: `<span>Welcome to <b>Light Bootstrap Dashboard</b> - a beautiful freebie for every web developer.</span>`,
            icon: 'nc-icon nc-app',
            horizontalAlign: horizontalAlign,
            verticalAlign: verticalAlign,
            type: this.type[color]
          })
      },
      capture() {
        this.canvas = this.$refs.canvas;
        var context = this.canvas.getContext("2d").drawImage(this.video, 0, 0, 640, 480);
        this.captures.push(canvas.toDataURL("image/png"));
      },
      detectFromVideoFrame(model, video) {
        console.log("Detecting model...")
        model.detect(video).then(predictions => {
          this.showDetections(predictions);

          requestAnimationFrame(() => {
            this.detectFromVideoFrame(model, video);
          });
        }, (error) => {
          console.log("Couldn't start the webcam")
          console.error(error)
        });
      },
      showDetections(predictions) {
        const ctx = this.$refs.canvas.getContext("2d");
        ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
        const font = "24px helvetica";
        ctx.font = font;
        ctx.textBaseline = "top";

        predictions.forEach(prediction => {
          const x = prediction.bbox[0];
          const y = prediction.bbox[1];
          const width = prediction.bbox[2];
          const height = prediction.bbox[3];
          // Draw the bounding box.
          ctx.strokeStyle = "#2fff00";
          ctx.lineWidth = 1;
          ctx.strokeRect(x, y, width, height);
          // Draw the label background.
          ctx.fillStyle = "#2fff00";
          const textWidth = ctx.measureText(prediction.class).width;
          const textHeight = parseInt(font, 10);
          // draw top left rectangle
          ctx.fillRect(x, y, textWidth + 10, textHeight + 10);
          // draw bottom left rectangle
          ctx.fillRect(x, y + height - textHeight, textWidth + 15, textHeight + 10);

          // Draw the text last to ensure it's on top.
          ctx.fillStyle = "#000000";
          ctx.fillText(prediction.class, x, y);
          ctx.fillText(prediction.score.toFixed(2), x, y + height - textHeight);
        });
      }
    },
    mounted() {
      this.videoRef = this.$refs.video;
      /* Show video
      if(navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
          navigator.mediaDevices.getUserMedia({ video: true }).then(stream => {
              // this.video.src = window.URL.createObjectURL(stream);
              this.video.srcObject = stream
              this.video.play();
          });
      }
      */
      if (navigator.mediaDevices.getUserMedia || navigator.mediaDevices.webkitGetUserMedia) {
        // define a Promise that'll be used to load the webcam and read its frames
        const webcamPromise = navigator.mediaDevices
          .getUserMedia({
            video: true,
            audio: false,
          })
          .then(stream => {
            // pass the current frame to the window.stream
            // window.stream = stream;
            // pass the stream to the videoRef
            this.videoRef.srcObject = stream;

            return new Promise(resolve => {
              this.videoRef.onloadedmetadata = () => {
                resolve();
              };
            });
          }, (error) => {
            console.log("Couldn't start the webcam")
            console.error(error)
          });

        // define a Promise that'll be used to load the model
        const loadlModelPromise = cocoSsd.load();
        
        // resolve all the Promises
        Promise.all([loadlModelPromise, webcamPromise])
          .then(values => {
            this.detectFromVideoFrame(values[0], this.videoRef);
          })
          .catch(error => {
            console.error(error);
          });
      }
    }
  }


</script>
<style lang="scss">

#video {
    background-color: #000000;
    align-center: true;
    position: absolute;
    top:60px;
    left:100px;
}

#canvas {
  position: absolute;
  top:60px;
  left:100px;

}

.video-container {
  width: 100%;
  height: 550px;
}
</style>
