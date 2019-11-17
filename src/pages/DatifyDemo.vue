<template>
  <div class="content">
    <div class="container-fluid">
      <card>
        <div class="row justify-content-center video-container">
          <div class="col-md-10 text-center">

            <div class="row" id="step_flow">
              <span v-bind:class="{active: step==1}" v-on:click="goto_step(1)">Step 1</span>
              >
              <span v-bind:class="{active: step==2}" v-on:click="goto_step(2)">Step 2</span>
              >
              <span v-bind:class="{active: step==3}" v-on:click="goto_step(3)">Step 3</span>
            </div>

            <h2>Parse & Add Event to Google Calender</h2>

            <div v-if="step==1">
              <div>
                <!-- <h5></h5> -->
                <button class="nc-icon nc-camera-20" style="font-size: 40px" v-on:click="capture()"></button>
              </div>

              <div>
                <video ref="video" id="video" width="640" height="480" autoplay/>
                <canvas ref="canvas" id="canvas" width="640" height="480"></canvas>
              </div>
            </div>

            <div v-if="step==2">
              <div v-if="event == null">
                <h2>No event detected.</h2>
                <button v-on:click="goto_step(1)">Re-take Image</button>
              </div>
              <div v-if="event != null">
                <h4>Detected Event:</h4>
                <table>
                  <tr>
                    <th>Name</th>
                    <td>
                      <input type="text" v-model="event.name"/>
                    </td>
                  </tr>
                  <tr>
                    <th>Time</th>
                    <td>
                      From <input type="text" v-model="event.date_time.start"/> to <input type="text" v-model="event.date_time.end"/> 
                    </td>
                  </tr>
                  <tr>
                    <th>Location</th>
                    <td><input type="text" v-model="event.location"/></td>
                  </tr>
                </table>

                <button v-on:click="goto_step(1)">Re-take Image</button>
                <button v-on:click="save_event(event)">Save Event</button>
              </div>
            </div>

            <div v-if="step==3">
              <div v-if="saved_event != null">
                <h4>Event has been saved successfully!</h4>
              </div>
              <div v-if="saved_event == null">
                <h4>No saved events detected.</h4>
              </div>
              <button v-on:click="goto_step(1)">Try Again</button>
            </div>
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
        captures: {},
        event: null,
        step: 1,
        event_created: false,
        saved_event: null
      }
    },
    methods: {
      reset_event(){
        this.event = null
        this.saved_event = null
      },
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
          this.event = response.data.data
          this.has_event = true
          this.goto_step(2)
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
      },
      goto_step(step){
        this.step = step
        if(this.step == 1){
          this.reset_event()
        } else if (this.step == 2){
          this.saved_event = null
        }
      },
      save_event(event){
        axios.post('http://localhost:5000/save-event', event).then((response) => {
          this.saved_event = response.data.data
          this.goto_step(3)
        }, (error) => {
          console.log(error)
        });
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
    position: relative;
    top:40px;
    left: 0px;
}

#canvas {
  position: absolute;
  top:40px;
  left:80px;
  visibility: hidden;

}

.video-container {
  width: 100%;
  height: 1000px;
}

.snap-btn {
  position: absolute;
  top:0px;
  left:100px;
  width:10%;
}

.output {
  position: absolute;
  top:560px;
}

.active {
  font-weight: bold
}

table {
  border-collapse: collapse;
  width: 100%;
}

th, td {
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {background-color: #f2f2f2;}

</style>
