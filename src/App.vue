<template>
  <div id="app">
    <div class="container">
      <div class="row">
        <div class="col-md-6">
          <select class="form-control" v-model="selectedSource" @change="getStream()">
            <option
              v-for="source in videoSources"
              :key="source.id"
              :value="source.id"
            >{{source.text}}</option>
          </select>

          <div class="mt-4">
            <video autoplay id="video1" ref="video1"></video>
            <canvas style="display:none;" id="canvas1" ref="canvas1"></canvas>
          </div>

          <div class="mt-3">
            <button class="btn btn-primary" @click="captureImage()">Capture Image</button>
          </div>
        </div>
        <div class="col-md-6">
          <img
            id="usercapture"
            ref="usercapture"
            src="http://via.placeholder.com/150x150"
            width="200"
            height="200"
          />

          <div class="mt-2">
            <button class="btn btn-success" @click="predictImage()">Predict Image</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import "bootstrap/dist/css/bootstrap.css";
import axios from "axios";

export default {
  name: "app",
  data() {
    return {
      videoSources: [],
      selectedSource: null,
      imageURL: null,
      uploadingImage: false,
      gCloudVisionUrl:
        "https://vision.googleapis.com/v1/images:annotate?key=AIzaSyCk-S9i5xMbmiEAIVPtSC8VlyecH1rY8Uo"
    };
  },
  mounted() {
    navigator.mediaDevices
      .enumerateDevices()
      .then(this.gotDevices)
      .catch(this.handleError);
  },
  methods: {
    gotDevices(deviceInfos) {
      for (var i = 0; i !== deviceInfos.length; ++i) {
        var deviceInfo = deviceInfos[i];

        if (deviceInfo.kind === "videoinput") {
          let option = {};

          option.id = deviceInfo.deviceId;
          option.text = deviceInfo.label || "camera " + (i - 1);

          this.videoSources.push(option);
        }
      }

      console.log(this.videoSources);
    },
    getStream() {
      if (window.stream) {
        window.stream.getTracks().forEach(function(track) {
          track.stop();
        });
      }

      console.log(this.selectedSource);

      var constraints = {
        video: {
          deviceId: { exact: this.selectedSource }
        }
      };

      navigator.mediaDevices
        .getUserMedia(constraints)
        .then(this.gotStream)
        .catch(this.handleError);
    },
    gotStream(stream) {
      //window.stream = stream; // make stream available to console
      console.log(this.$refs.video1.srcObject);
      this.$refs.video1.srcObject = stream;
    },
    handleError(error) {
      console.error("Error: ", error);
    },
    captureImage() {
      let canvas = this.$refs.canvas1;
      let videoElement = this.$refs.video1;
      canvas.width = videoElement.videoWidth;
      canvas.height = videoElement.videoHeight;
      canvas.getContext("2d").drawImage(videoElement, 0, 0);

      let imgElement = this.$refs.usercapture;

      // Get image
      let image = canvas.toDataURL("image/png");
      console.log(image);

      //Trim signature to get pure image data
      this.imageURL = image.replace(/^data:image\/(png|jpg);base64,/, "");

      //Set the image element to the data url
      imgElement.src = image;
    },
    async predictImage() {
      let requestBody = {
        requests: [
          {
            image: {
              content: this.imageURL
            },
            features: [
              {
                type: "LABEL_DETECTION",
                maxResults: 1
              }
            ]
          }
        ]
      };
      try {
        let predictionResults = await axios.post(
          this.gCloudVisionUrl,
          requestBody
        );

        let predictionResponse = predictionResults.data.responses[0]; //As we only asked for one response

        console.log(predictionResponse);

        let annotations = predictionResponse.labelAnnotations;

        let allLabelDescriptions = annotations.map(
          annotation => annotation.description
        );

        console.log(allLabelDescriptions);
      } catch (error) {
        console.log(error);
      }
    }
  }
};
</script>

<style scoped>
#video1 {
  width: 500px;
  height: 400px;
  background-color: grey;
}
</style>
