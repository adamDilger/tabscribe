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
    waveColor: '#6ee7b7',
    progressColor: '#065f46',
    media: videoRef.value,
    dragToSeek: true,
    hideScrollbar: true,
    plugins: [
      Hover.create({
        lineColor: '#b91c1c',
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

  wv.on('click', () => (currentScroll = wv!.getCurrentTime()))
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
  <div class="max-h-full w-full flex flex-col justify-center text-white">
    <div class="flex justify-center">
      <video ref="videoRef" src="/video.webm?url" class="max-w-7xl" />
    </div>
    <div ref="waveformRef" class="mx-8"></div>

    <div class="flex mx-8 my-4 gap-8 justify-end">
      <div>
        <label>
          <div class="font-bold text-sm">Zoom</div>
          <input ref="sliderRef" type="range" min="1" max="500" step="1" />
        </label>
      </div>

      <div>
        <label>
          <div class="font-bold text-sm">Speed</div>
          <input type="range" min="0" max="3" step="1" v-model="currentSpeed" />
          {{ speeds[currentSpeed] }}x
        </label>
      </div>
    </div>
  </div>
</template>
