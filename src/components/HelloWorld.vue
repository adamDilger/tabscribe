<script setup lang="ts">
import { onMounted, ref, useTemplateRef, watch } from 'vue'
import WaveSurfer from 'wavesurfer.js'
import Hover from 'wavesurfer.js/dist/plugins/hover.esm.js'
import { onKeyStroke } from '@vueuse/core'

const videoRef = useTemplateRef('videoRef')
const waveformRef = useTemplateRef('waveformRef')
const sliderRef = useTemplateRef('sliderRef')

let wv: WaveSurfer | null = null

onMounted(() => {
  if (!videoRef.value || !waveformRef.value) return

  wv = WaveSurfer.create({
    container: waveformRef.value,
    waveColor: 'rgb(200, 0, 200)',
    progressColor: 'rgb(100, 0, 100)',
    media: videoRef.value,
    dragToSeek: true,
    hideScrollbar: true,
    plugins: [
      Hover.create({
        lineColor: '#ff0000',
        lineWidth: 2,
        labelBackground: '#555',
        labelColor: '#fff',
        labelSize: '11px'
      })
    ]
  })

  wv.once('decode', () => {
    sliderRef.value!.addEventListener('input', (e) => {
      const minPxPerSec = (e.target as HTMLInputElement).valueAsNumber
      wv!.zoom(minPxPerSec)
    })
  })
})

let currentScroll = 0
onKeyStroke(' ', async () => {
  if (!wv) return console.error("WaveSurfer isn't ready yet")

  if (wv.isPlaying()) {
    wv.setOptions({ autoCenter: false, autoScroll: false })
    wv.pause()
    wv.setTime(currentScroll)
    wv.setOptions({ autoCenter: true, autoScroll: true })
  } else {
    wv.play()
    currentScroll = wv.getCurrentTime()
  }
})

const speeds = [0.25, 0.5, 0.75, 1]
const currentSpeed = ref(3)
watch(currentSpeed, (speed) => {
  if (!wv) return console.error("WaveSurfer isn't ready yet")
  wv.setPlaybackRate(speeds[speed])
})
</script>

<template>
  <div class="max-h-full w-full flex flex-col justify-center">
    <div class="flex justify-center">
      <video ref="videoRef" src="/video.webm?url" class="max-w-7xl" />
    </div>
    <div ref="waveformRef" class="mx-8"></div>

    <div class="mx-8 my-4">
      <input ref="sliderRef" type="range" min="10" max="1000" step="1" />
    </div>

    <div class="mx-8 my-4">
      <label>
        0.25x
        <input type="range" min="0" max="3" step="1" v-model="currentSpeed" />
        4x
      </label>
    </div>
  </div>
</template>
