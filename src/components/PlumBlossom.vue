<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { useRaf } from '../utils';

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

const growChance = 0.5
const firstLeafLen = 5
const minGrowDepth = 5
const extraBounding = 100
const basicGrowAngle = Math.PI / 12

const drawFunList: Function[] = []

function beginDraw() {
  ctx.strokeStyle = '#00000040'
  ctx.lineWidth = 1
  drawRandomBlossom({
    begin: {
      x: Math.floor(uiRect.width / 2),
      y: 0
    },
    length: firstLeafLen,
    angle: Math.PI / 2
  }, 0)
}

function drawRandomBlossom(leaf: Leaf, depth: number) {
  const endPoint: Point = {
    x: leaf.begin.x + leaf.length * Math.cos(leaf.angle),
    y: leaf.begin.y + leaf.length * Math.sin(leaf.angle)
  }

  // 判定边界
  if (endPoint.x < -extraBounding || endPoint.x > uiRect.width + extraBounding ||
    endPoint.y < -extraBounding || endPoint.y > uiRect.height + extraBounding
  ) {
    return
  }

  drawLine(leaf.begin, endPoint)

  if (depth < minGrowDepth || Math.random() < growChance) {
    drawFunList.push(() => {
      drawRandomBlossom(generateLeaf(leaf, endPoint, true), depth + 1)
    })
  }
  if (depth < minGrowDepth || Math.random() < growChance) {
    drawFunList.push(() => {
      drawRandomBlossom(generateLeaf(leaf, endPoint, false), depth + 1)
    })
  }
}

function generateLeaf(leaf: Leaf, lastEndPoint: Point, toRight: boolean): Leaf {
  const randomAngleValue = Math.random() * basicGrowAngle
  const angle = leaf.angle + (toRight ? randomAngleValue : -randomAngleValue)

  return {
    begin: lastEndPoint,
    length: firstLeafLen * Math.random(),
    angle
  }
}

function drawLine(begin: Point, end: Point) {
  ctx.beginPath()
  ctx.moveTo(begin.x, begin.y)
  ctx.lineTo(end.x, end.y)
  ctx.stroke()
}

const { resume, pause } = useRaf(() => {
  const tasks = [...drawFunList]
  drawFunList.length = 0

  tasks.forEach((fun) => { fun() })

  if (!tasks.length) {
    pause()
  }
})

onMounted(() => {
  init()
})

</script>

<template>
  <div>
    <canvas ref="ui" id="ui"></canvas>  </div>
</template>

<style scoped>
#ui {
  width: 800px;
  height: 800px;
}
</style>
