<script setup>
import {ref} from 'vue'

const barWidth = ref(1)
const hd = ref(false)

const start = async function (mode) {
  let file
  if (mode === 'sample') {
    file = await fetch('/public/rdr.mp3')
      .then(response => {
        if (!response.ok) {
          console.log("error")
          throw new Error('Network response was not ok');
        }
        return response.blob();
      })
  } else {
    file = document.getElementById('input').files[0];
  }
  if (!file) {
    console.log('Please upload a file');
    return
  }
  const reader = new FileReader();

  reader.readAsArrayBuffer(file);

  reader.onload = function (event) {
    const arrayBuffer = event.target.result;
    const blob = new Blob([arrayBuffer], {type: 'audio/mp3'});
    const url = URL.createObjectURL(blob);
    const audio = new Audio(url);
    audio.play().catch(error => console.error("Error playing audio:", error));

    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    let analyser = null;

    let audioSource = audioCtx.createMediaElementSource(audio);
    analyser = audioCtx.createAnalyser();
    if(hd.value===true){
      analyser.fftSize = 2048;
    }
    else{
      analyser.fftSize = 512;
    }
    analyser.smoothingTimeConstant = 0;

    audioSource.connect(analyser);
    analyser.connect(audioCtx.destination);

    const bufferLength = analyser.frequencyBinCount
    const dataArray = new Uint8Array(bufferLength);


    const canvas = document.getElementById("canvas");
    canvas.width = bufferLength;
    canvas.height = window.innerHeight;
    const ctx = canvas.getContext("2d");
    ctx.imageSmoothingEnabled = false;
    ctx.mozImageSmoothingEnabled = false;
    ctx.webkitImageSmoothingEnabled = false;
    ctx.msImageSmoothingEnabled = false;


    console.log("data array len: " + dataArray.length)
    console.log("canvas width: " + canvas.width)
    console.log("canvas height: " + canvas.height)


    let image = ctx.createImageData(canvas.width, canvas.height / 2)
    const bytewidth = 4 * image.width;
    ctx.putImageData(image, 0, canvas.height / 2);

    let x = 0;

    function animate() {
      x = 0;
      ctx.clearRect(0, 0, canvas.width, canvas.height / 2);

      analyser.getByteFrequencyData(dataArray);
      for (let i = 0; i < bufferLength; i++) {
        let barHeight = dataArray[i];
        const color = heatColors(barHeight)
        ctx.fillStyle = `rgb(${color[0]}, ${color[1]}, ${color[2]}, ${color[3]})`;

        ctx.fillRect(x, canvas.height / 2, parseInt(barWidth.value), -barHeight);
        x += parseInt(barWidth.value);
      }
      let j = 0;
      //Move the bitmap one bit below
      for (let i = image.data.length - 1; i >= bytewidth; i--) {
        image.data[i] = image.data[i - bytewidth];
      }
      //Set new color row

      for (let i = 0; i < bytewidth; i+=4*parseInt(barWidth.value)) {
        const colors = heatColors(dataArray[j]);
        for (let k=0; k<4*parseInt(barWidth.value); k+=4){
          image.data[i+k] = colors[0];
          image.data[i+1+k] = colors[1];
          image.data[i+2+k] = colors[2];
          image.data[i+3+k] = colors[3];
        }
        j++
      }
      //Move the bitmap one bit below
      for (let i = image.data.length - 1; i >= bytewidth; i--) {
        image.data[i] = image.data[i - bytewidth];
      }
      ctx.putImageData(image, 0, canvas.height / 2);

      const pixelsPerThousandHz = bufferLength/22
      for(let i = 0; i < bufferLength; i+=pixelsPerThousandHz) {
        ctx.fillStyle = "rgb(255,255,255,64)";
        ctx.fillRect(i, 0, 1, canvas.height);
      }
      requestAnimationFrame(animate);
    }

    animate();
  };
  reader.onerror = function (event) {
    console.error("Error reading file:", event.target.error);
  };

}

const heatColors = function(i){
  //with this array i save lots of time instead of calculating the color with map functions and multiplications
  const colors = [
    [0, 0, 0, 255], //black
    [0, 0, 8, 255],
    [0, 0, 16, 255],
    [0, 0, 24, 255],
    [0, 0, 32, 255],
    [0, 0, 40, 255],
    [0, 0, 48, 255],
    [0, 0, 56, 255],

    [0, 0, 64, 255],//dark blue
    [0, 0, 72, 255],
    [0, 0, 80, 255],
    [0, 0, 88, 255],
    [0, 0, 96, 255],
    [0, 0, 104, 255],
    [0, 0, 112, 255],
    [0, 0, 120, 255],

    [0, 0, 128,255], // blue
    [16, 0, 128,255],
    [32, 0, 128,255],
    [48, 0, 128,255],
    [64, 0, 128,255],
    [80, 0, 128,255],
    [96, 0, 128,255],
    [112, 0, 128,255],

    [128, 0, 128,255],// magenta
    [144, 0, 112,255],
    [160, 0, 96,255],
    [176, 0, 80,255],
    [192, 0, 64,255],
    [208, 0, 48,255],
    [224, 0, 32,255],
    [240, 0, 16,255],

    [255, 0, 0,255], // red
    [255, 16, 0,255],
    [255, 32, 0,255],
    [255, 48, 0,255],
    [255, 64, 0,255],
    [255, 80, 0,255],
    [255, 96, 0,255],
    [255, 112, 0,255],

    [255, 128, 0,255],// orange
    [255, 144, 0,255],
    [255, 160, 0,255],
    [255, 176, 0,255],
    [255, 192, 0,255],
    [255, 208, 0,255],
    [255, 224, 0,255],
    [255, 240, 0,255],

    [255, 255, 0, 255],// yellow
    [255, 255, 16,255],
    [255, 255, 32,255],
    [255, 255, 48,255],
    [255, 255, 64,255],
    [255, 255, 80,255],
    [255, 255, 96,255],
    [255, 255, 112,255],

    [255, 255, 128,255],// white
    [255, 255, 144,255],
    [255, 255, 160,255],
    [255, 255, 176,255],
    [255, 255, 192,255],
    [255, 255, 208,255],
    [255, 255, 224,255],
    [255, 255, 240,255],
  ];

  if(i<0||i>255){
    return [0,0,0,255]
  }
  else{
    return colors[i>>2]
  }
}
</script>

<template>
  <div class="app">
    <div class="analyzer">
      <canvas id="canvas"></canvas>
      <audio id="audio"></audio>
    </div>
    <div class="side">
      <div class="table-cell">
        <h1>VISUAUDIO!</h1>
        <marquee>spectrum analyser, graphical visualiser</marquee>
        <input class="fileinput" type="file" id="input" accept=".mp3, .m4a" />
        <button @click="start">Upload</button>
      </div>
      <input class="slider" type="range" v-model="barWidth" min="1" max="5" />
      <label>{{barWidth}}</label>
      <div class="table-cell">
        <input type="checkbox" v-model="hd">High FFT mode</input>
      </div>
    </div>
  </div>
</template>

<style scoped>
  canvas{
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: #000
  }
  .app{
    display: flex;
    flex-direction: row;
    background-color: #030305;
    color: aliceblue;
  }
  .analyzer{
    border-style: solid;
    width: 80vw;
    height: 97vh;
  }
  .side{
    display: flex;
    flex-direction: column;
    border-style: solid;
    width: 20vw;
    height: 97vh;
    font-family: sans-serif;
    text-align: center;
    justify-content: center;
   }
  .table-cell{
    display : table-cell;
    vertical-align : middle;
    text-align : center;
    justify-content: center;
  }
  .fileinput{
  }
  .slider{
    margin:0 30%;
  }
  @media (max-width: 900px){
    .app{
      flex-direction: column-reverse;
    }
    .analyzer{
      width: 100vw;
      height: 80vh;
    }
    .side{
      width: 100vw;
      height: 18vh;
    }
  }
</style>
