<template>
  <div>
    <audio
      autoplay
      controls
      v-if="recordingDataURL"
      :src="recordingDataURL"
    ></audio>

    <a :href="recordingDataURL">audio file</a>

    {{ format }}
    
    <button @click="micCheck">
        hi
    </button>
  </div>
</template>
<script setup lang="ts">
import { ref } from "vue";
import recordrtc from "recordrtc";

let recordingDataURL = ref("");
let format = ref("");

const micCheck = async () => {
  let stream = await navigator.mediaDevices.getUserMedia({
    video: true,
    audio: true,
  });
  let recorder = new recordrtc.RecordRTCPromisesHandler(stream, {
    type: "audio",
    mimeType: "audio/wav",
    recorderType: recordrtc.StereoAudioRecorder,
  });
  recorder.startRecording();

  const sleep = (m: number) => new Promise((r) => setTimeout(r, m));
  await sleep(3000);

  await recorder.stopRecording();
  let dataURL = await recorder.getDataURL();
  recordingDataURL.value = dataURL;

  let blob = await recorder.getBlob();
  format.value = blob.size + " " + blob.type
};
</script>
