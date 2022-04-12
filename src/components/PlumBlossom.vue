<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { PlumService } from './PlumBlossom.service';

let plumService: PlumService = null
const canvasEl = ref<HTMLCanvasElement>()

onMounted(() => {
  init()
})

function init() {
  const initLen = 5 // px
  plumService = new PlumService(canvasEl.value)

  plumService.beginPlumGrow([
    // 从上往下
    {
      begin: {
        x: Math.floor(document.body.clientWidth / 2),
        y: 0
      },
      length: initLen,
      angle: Math.PI / 2
    },

    // 从下往上
    {
      begin: {
        x: Math.floor(document.body.clientWidth / 2),
        y: Math.floor(document.body.clientHeight)
      },
      length: initLen,
      angle: -Math.PI / 2
    },

    // 从左往右
    {
      begin: {
        x: 0,
        y: Math.floor(document.body.clientHeight / 2)
      },
      length: initLen,
      angle: 0
    },

    // 从右往左
    {
      begin: {
        x: Math.floor(document.body.clientWidth),
        y: Math.floor(document.body.clientHeight / 2)
      },
      length: initLen,
      angle: -Math.PI
    }
  ])
}
</script>

<template>
  <div class="container">
    <canvas ref="canvasEl" id="ui"></canvas>
  </div>
</template>

<style scoped>
.container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: row;
  margin: 0 auto;
}

#ui {
  display: block;
  width: 100%;
  height: 100%;
}

.copywriter {
  writing-mode: vertical-lr;
  font-size: 50px;
}
</style>
