<script setup>
//https://blog.mahoroi.com/posts/2020/05/browser-head-pose-estimation/
import * as faceapi from 'face-api.js';
import { onMounted, ref } from 'vue';
var video;
var canvas;
var canvas2;
var ctxCanvas2;

const landmarks = ref();
const nose = ref();
const leftEye = ref();
const rightEye = ref();
const jaw = ref();
const leftMouth = ref();
const rightMouth = ref();
const leftOutline = ref();
const rightOutline = ref();


const webCameraOn = () => {
  navigator.mediaDevices.getUserMedia({
    video: true,
    audio: false,
  }).then(stream => {
    video.srcObject = stream;
    video.play()
    detect();
  }).catch(e => {
    console.log(e)
  })
};
async function detect() {
  requestAnimationFrame(detect);

  //  webカメラの映像から顔認識を行う
  const useTinyModel = true;
  const detection = await faceapi
    .detectSingleFace(
      video,
      new faceapi.TinyFaceDetectorOptions({
        inputSize: 160,
      })
    )
    .withFaceLandmarks(useTinyModel);

  if (!detection) {
    return;
  }

  // 認識データをリサイズ
  const resizedDetection = faceapi.resizeResults(detection, {
    width: video.width,
    height: video.height,
  });

  // ランドマークをキャンバスに描画
  canvas = document.getElementById('canvas1');
  canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height);
  faceapi.draw.drawFaceLandmarks(canvas, resizedDetection);
  // 以後使用するランドマーク座標
  landmarks.value = resizedDetection.landmarks;
  nose.value = landmarks.value.getNose()[3];
  leftEye.value = landmarks.value.getLeftEye()[0];
  rightEye.value = landmarks.value.getRightEye()[3];
  jaw.value = landmarks.value.getJawOutline()[8];
  leftMouth.value = landmarks.value.getMouth()[0];
  rightMouth.value = landmarks.value.getMouth()[6];
  leftOutline.value = landmarks.value.getJawOutline()[0];
  rightOutline.value = landmarks.value.getJawOutline()[16];
}
onMounted(() => {
  video = document.getElementById("video")
  Promise.all([
    faceapi.nets.tinyFaceDetector.loadFromUri('/static/models/weights'),
    faceapi.nets.faceLandmark68TinyNet.loadFromUri('/static/models/weights'),
  ]).then(webCameraOn);
  canvas2 = document.getElementById("canvas2");
  ctxCanvas2 = canvas2.getContext('2d')


})

</script>
<template>
  <div class="grid grid-cols-2">
    <div style="position: relative;" class="">
      <div>
        <video id="video" width="640" height="480" style="transform: scaleX(-1);"></video>
        <canvas id="canvas1" width="640" height="480"
          style="position: absolute; top: 0px; left: 0px; transform: scaleX(-1);"></canvas>
        <canvas id="canvas2" width="640" height="480"
          style="position: absolute; top: 0px; left: 0px; transform: scaleX(-1);"></canvas>
      </div>
    </div>
    <div class="p-4 border">
      <p>nose:{{ nose }}</p>
      <p>leftEye:{{ leftEye }}</p>
      <p>rightEye:{{ rightEye }}</p>
      <p>jaw:{{ jaw }}</p>
      <p>leftMouth:{{ leftMouth }}</p>
      <p>rightMouth:{{ rightMouth }}</p>
      <p>leftOutline:{{ leftOutline }}</p>
      <p>rightOutline:{{ rightOutline }}</p>
    </div>
  </div>
</template>