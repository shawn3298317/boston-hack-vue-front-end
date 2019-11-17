<template>
  <div class="content">
    <div class="container-fluid">
      <card>
        <div class="row justify-content-center video-container">
          <div class="col-md-10 text-center">
            <h5>Parse & Add Event to Google Calender</h5>
            
            <div class="">
              <!-- <h5></h5> -->
              <button class="nc-icon nc-camera-20 snap-btn" v-on:click="capture()"></button>
            </div>

            <div class="justify-content-center">
              <video ref="video" id="video" width="640" height="480" autoplay/>
              <canvas ref="canvas" id="canvas" width="640" height="480"></canvas>
            </div>
            
            
            
            <!-- <ul>
                <li v-for="c in captures">
                    <img v-bind:src="c" height="50" />
                </li>
            </ul> -->

          </div>
          
        </div>
        
      </card>
    </div>
  </div>
</template>
<!-- Load TensorFlow.js-->
<!-- <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.0.0/dist/tf.min.js"></script> -->
<!-- Load the coco-ssd model. -->
<!-- <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script> -->
<script>
  import Card from 'src/components/Cards/Card.vue'
  import axios from "axios"

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
      capture() {
        this.canvas = this.$refs.canvas;
        var context = this.canvas.getContext("2d").drawImage(this.videoRef, 0, 0, 640, 480);
        // console.log(this.videoRef, this.videoRef.srcObject)
        // this.captures.push(this.canvas.toDataURL("image/png"));
        // console.log(this.canvas.toDataURL("image/jpeg"))
        // data = {'img': this.canvas.toDataURL("image/jpeg"), 'uid': 123}
        axios.post('http://localhost:5000/submit-img', {
          'img': this.canvas.toDataURL("image/jpeg"),
          'uid': 123
        }).then((response) => {
          console.log(response)
        }, (error) => {
          console.log(error)
        });
        // this.$http({method: 'POST', url: 'http://www.mocky.io/v2/5dd09e0c2f00007e003f20d8', data: data}).then(response => {
        //     this.postResults = data;
        //     this.ajaxRequest = false;
        //     console.log(response.data)
        //   }).catch((err) => {
        //     console.log(err)
        //   });

      }
    },
    mounted() {
      this.videoRef = this.$refs.video;
      // Show video
      if(navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
          navigator.mediaDevices.getUserMedia({ video: true }).then(stream => {
              // this.videoRef.src = window.URL.createObjectURL(stream);
              this.videoRef.srcObject = stream
              this.videoRef.play();
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
    top:40px;
    left:80px;
}

#canvas {
  position: absolute;
  top:40px;
  left:80px;
  visibility: hidden;

}

.video-container {
  width: 100%;
  height: 600;
}

.snap-btn {
  position: absolute;
  top:0px;
  left:100px;
  width:10%;
}
</style>
