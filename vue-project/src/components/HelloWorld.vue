<template>
  <div>
    <audio
      autoplay
      controls
      v-if="recordingDataURL"
      :src="recordingDataURL"
    ></audio>
    <br />
    <a download="file.wav" :href="recordingDataURL">audio file</a>
    <br />
    Format: {{ format }}
    <br />
    Connection type: {{ connectionRef }}
    <br />
    kbps result {{ kbpsRef }}
    <br />
    Bitrate
    <button @click="kbpsRefOverride.current = 320">set 320</button>
    <button @click="kbpsRefOverride.current = 128">set 128</button>
    <br />
    <button @click="micCheck">start</button>
    <button @click="finish">stop</button>

    {{ JSON.stringify(options) }}
  </div>
</template>
<script setup lang="ts">
import { ref } from "vue";
import recordrtc from "recordrtc";
import lamejs from "lamejs";

let recordingDataURL = ref("");
let format = ref("");
let connectionRef = ref("");
let kbpsRef = ref();
let kbpsRefOverride = ref();
let recorderRef = ref<recordrtc.RecordRTCPromisesHandler>();

const options = {
  recorderType: recordrtc.StereoAudioRecorder,
  type: "audio" as const,
  mimeType: "audio/wav" as const,
  disableLogs: !import.meta.env.DEV,
  bufferSize: 4096 as const,
  numberOfAudioChannels: 1 as const,
  sampleRate: 48000,
  desiredSampleRate: 48000,
};

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
  await recorderRef.value?.stopRecording();

  let channels = 1; //1 for mono or 2 for stereo
  let sampleRate = 48000; //44.1khz (normal mp3 samplerate)

  let kbps = 256; //encode 128kbps mp3

  let connection =
    (navigator as any).connection ||
    (navigator as any).mozConnection ||
    (navigator as any).webkitConnection;

    connectionRef.value = connection?.effectiveType;

  switch (connection?.effectiveType) {
    case "slow-2g":
      kbps = 128;
    case "2g":
      kbps = 192;
    case "3g":
      kbps = 256;
    case "4g":
      kbps = 320;
    default:
      kbps = 256;
  }

  kbpsRef.value = kbps;

  if (kbpsRefOverride.value) {
    kbps = kbpsRefOverride.value;
  }

  let mp3encoder = new lamejs.Mp3Encoder(channels, sampleRate, kbps);

  let blobSource = await recorderRef.value?.getBlob();
  if (blobSource) {
    // samples = new Int16Array(44100); //one second of silence (get your data from the source you have)
    let samples123 = await blobSource?.arrayBuffer(); //one second of silence (get your data from the source you have)
    const samples = new Int16Array(samples123);

    let sampleBlockSize = 1152; //can be anything but make it a multiple of 576 to make encoders life easier

    let mp3Data = [];
    for (let i = 0; i < samples.length; i += sampleBlockSize) {
      let sampleChunk = samples.subarray(i, i + sampleBlockSize);
      let mp3buf = mp3encoder.encodeBuffer(sampleChunk);
      if (mp3buf.length > 0) {
        mp3Data.push(mp3buf);
      }
    }
    let mp3buf = mp3encoder.flush(); //finish writing mp3

    if (mp3buf.length > 0) {
      mp3Data.push(new Int8Array(mp3buf));
    }

    let blob = new Blob(mp3Data, { type: "audio/mp3" });
    let url = window.URL.createObjectURL(blob);
    console.log("MP3 URl: ", url);
    recordingDataURL.value = url;
    format.value = blob.size * 0.000001 + " " + blob.type;
  }
};
</script>
