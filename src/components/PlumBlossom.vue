<script setup lang="ts">
import { onMounted, ref } from 'vue'

interface Point {
  x: number
  y: number
}

/**
 * 定义叶子类
 */
interface Leaf {
  /** 花瓣内坐标 */
  begin: Point
  /** 枝叶长度 单位px */
  length: number
  /** 生长角度 单位弧度 */
  angle: number
}

const ui = ref<HTMLCanvasElement>()
let ctx: CanvasRenderingContext2D = null
let uiRect: DOMRect = null

function init() {
  // 拿到画布上下文
  ctx = ui.value?.getContext('2d')
  uiRect = ui.value?.getBoundingClientRect()
  if (!ctx || !uiRect) {
    console.warn('there has no canvas element!')
    return
  }

  // 要设置，否则按照默认300:150
  ui.value.width = uiRect.width
  ui.value.height = uiRect.height

  beginDraw()
}

/** 枝叶生长概率 0<x<1 */
const growChance = 0.3
const initLength = 30
const growLength = 10
const minGrowDepth = 5
const maxGrowDepth = 15
const basicGrowAngle = 20 * Math.PI / 180

function beginDraw() {
  ctx.fillStyle = '#DDDDDD'

  drawRandomBlossom({
    begin: {
      x: Math.floor(uiRect.width / 2),
      y: uiRect.height
    },
    length: initLength,
    angle: Math.PI / 2
  }, 0)
}

function drawRandomBlossom(leaf: Leaf, depth: number) {
  const endPoint: Point = {
    x: leaf.begin.x + leaf.length * Math.cos(leaf.angle),
    y: leaf.begin.y - leaf.length * Math.sin(leaf.angle)
  }
  drawLine(leaf.begin, endPoint)
  if (depth >= maxGrowDepth) {
    // 限制生长
    return
  }

  if (depth < minGrowDepth || Math.random() < growChance) {
    drawRandomBlossom(generateLeaf(leaf, endPoint, true), depth + 1)
  }
  if (depth < minGrowDepth || Math.random() < growChance) {
    drawRandomBlossom(generateLeaf(leaf, endPoint, false), depth + 1)
  }
}

function generateLeaf(leaf: Leaf, lastEndPoint: Point, toRight: boolean) {
  const randomAngleValue = Math.random() * basicGrowAngle
  const angle = leaf.angle + (toRight ? randomAngleValue : -randomAngleValue)

  return {
    begin: lastEndPoint,
    length: Math.random() < 0.3 ? leaf.length : growLength,
    angle
  }
}

function drawLine(begin: Point, end: Point) {
  ctx.moveTo(begin.x, begin.y)
  ctx.lineTo(end.x, end.y)
  ctx.stroke()
}

onMounted(() => {
  init()
})

</script>

<template>
  <div>
    <canvas ref="ui" id="ui"></canvas>
  </div>
</template>

<style scoped>
#ui {
  width: 800px;
  height: 800px;
}
</style>
