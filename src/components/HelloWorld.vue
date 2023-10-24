<template>
  <div>
    <canvas class="visualizer" height="60px"></canvas>
    <button class="record" @click="recordAudio()">Record</button>
    <button class="stop" @click="stopAudio()">Stop</button>
    <section class="sound-clips"></section>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  name: "HelloWorld",
  data() {
    return {
      canvas: null,
      canvasCtx: null,
      chunks: [],
      mediaRecorder: null,
      record: null,
      stop: null,
      dataArray: null,
      analyser: null,
      audioCtx: null,
    };
  },
  mounted() {
    this.canvas = document.querySelector(".visualizer");
    this.canvasCtx = this.canvas.getContext("2d");
    this.record = document.querySelector(".record");
    this.stop = document.querySelector(".stop");
    this.soundClips = document.querySelector(".sound-clips");

    if (navigator.mediaDevices.getUserMedia) {
      const constraints = { audio: true };
      navigator.mediaDevices
        .getUserMedia(constraints)
        .then(this.onSuccess, this.onError);
    }
  },
  methods: {
    recordAudio() {
      this.mediaRecorder.start();
      console.log("recorder started");
      this.record.style.background = "red";

      this.stop.disabled = false;
      this.record.disabled = true;
    },
    stopAudio() {
      this.mediaRecorder.stop();
      console.log("recorder stopped");
      this.record.style.background = "";
      this.record.style.color = "";
      this.mediaRecorder.requestData();

      this.stop.disabled = true;
      this.record.disabled = false;
    },
    onSuccess(stream) {
      console.log("You let me use your mic!");
      this.mediaRecorder = new MediaRecorder(stream);
      this.visualize(stream);
      this.mediaRecorder.onstop = function () {
        console.log("data available after MediaRecorder.stop() called.");

        const clipName = prompt(
          "Enter a name for your sound clip?",
          "My unnamed clip"
        );

        const clipContainer = document.createElement("article");
        const clipLabel = document.createElement("p");
        const audio = document.createElement("audio");
        const deleteButton = document.createElement("button");

        clipContainer.classList.add("clip");
        audio.setAttribute("controls", "");
        deleteButton.textContent = "Delete";
        deleteButton.className = "delete";

        if (clipName === null) {
          clipLabel.textContent = "My unnamed clip";
        } else {
          clipLabel.textContent = clipName;
        }

        clipContainer.appendChild(audio);
        clipContainer.appendChild(clipLabel);
        clipContainer.appendChild(deleteButton);
        this.soundClips.appendChild(clipContainer);

        audio.controls = true;
        const blob = new Blob(this.chunks, { type: "audio/ogg; codecs=opus" });
        this.chunks = [];
        const audioURL = window.URL.createObjectURL(blob);
        audio.src = audioURL;
        console.log("recorder stopped");

        deleteButton.onclick = function (e) {
          e.target.closest(".clip").remove();
        };

        clipLabel.onclick = function () {
          const existingName = clipLabel.textContent;
          const newClipName = prompt("Enter a new name for your sound clip?");
          if (newClipName === null) {
            clipLabel.textContent = existingName;
          } else {
            clipLabel.textContent = newClipName;
          }
        };
      };
      this.mediaRecorder.ondataavailable = function (e) {
        this.chunks.push(e.data);
        
      };
    },
    onError() {
      console.log("No mic for you!");
    },
    visualize(stream) {
      if (!this.audioCtx) {
        this.audioCtx = new AudioContext();
      }
      const source = this.audioCtx.createMediaStreamSource(stream);

      this.analyser = this.audioCtx.createAnalyser();
      this.analyser.fftSize =1024;
      this.bufferLength = this.analyser.frequencyBinCount;
      this.dataArray = new Uint8Array(this.bufferLength);

      source.connect(this.analyser);
      this.draw();
    },
    draw() {
      const WIDTH = this.canvas.width;
      const HEIGHT = this.canvas.height;

      requestAnimationFrame(this.draw);

      this.analyser.getByteFrequencyData(this.dataArray);

      this.canvasCtx.fillStyle = "rgb(200, 200, 200)";
      this.canvasCtx.fillRect(0, 0, WIDTH, HEIGHT);

      this.canvasCtx.lineWidth = 2;
      this.canvasCtx.strokeStyle = "rgb(0, 0, 0)";

      this.canvasCtx.beginPath();

      //  let sliceWidth = (WIDTH * 1.0) / this.bufferLength;
      const barWidth = (WIDTH / this.bufferLength) * 2.5;
      let barHeight;
      let x = 0;
      for (let i = 0; i < this.bufferLength; i++) {
        
        barHeight = this.dataArray[i] /4;

        this.canvasCtx.fillStyle = `rgb(${barHeight + 100}, 50, 50)`;
        this.canvasCtx.fillRect(x, (HEIGHT - barHeight) / 1.5, barWidth, barHeight);

        x += barWidth + 1;
      }

      this.canvasCtx.lineTo(this.canvas.width, this.canvas.height / 2);
      this.canvasCtx.stroke();
      
    },
    
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
