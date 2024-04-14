<template>
  <div>
    <audio
      autoplay
      controls
      v-if="recordingDataURL"
      :src="recordingDataURL"
    ></audio>

    <a download="file.wav" :href="recordingDataURL">audio file</a>

    {{ format }}

    <button @click="micCheck">
        start
    </button>

    <button @click="finish">
        stop
    </button>

    {{ JSON.stringify(options) }}
  </div>
</template>
<script setup lang="ts">
import { ref } from "vue";
import recordrtc from "recordrtc";

let recordingDataURL = ref("");
let format = ref("");
let recorderRef = ref();

const options = {
    recorderType: recordrtc.StereoAudioRecorder,
    type: 'audio',
    mimeType: 'audio/wav',
    disableLogs: !import.meta.env.DEV,
    bufferSize: 4096,
    numberOfAudioChannels: 1,
    sampleRate: 22050,
    desiredSampRate: 22050,
  }

const micCheck = async () => {
  let stream = await navigator.mediaDevices.getUserMedia({
    video: false,
    audio: true,
  });
  let recorder = new recordrtc.RecordRTCPromisesHandler(stream, options);
  recorderRef.value = recorder;

  recorderRef.value.startRecording();
};

const finish = async () => {
await recorderRef.value.stopRecording();
let dataURL = await recorderRef.value.getDataURL();
recordingDataURL.value = dataURL;

let blob = await recorderRef.value.getBlob();
format.value = blob.size * 0.000001 + " " + blob.type
}
</script>
