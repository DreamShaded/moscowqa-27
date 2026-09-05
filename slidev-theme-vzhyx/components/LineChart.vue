<script setup lang="ts">
import { computed } from 'vue'

interface DataPoint {
  x: number
  value: number
}

interface Series {
  id: string
  label: string
  color: string
  dash?: boolean
  fill?: boolean
  data: DataPoint[]
}

interface Annotation {
  x: number
  label: string
  color?: string
}

const props = withDefaults(defineProps<{
  series: Series[]
  annotations?: Annotation[]
  xLabel?: string
  yLabel?: string
  width?: number
  height?: number
  xMin?: number
  xMax?: number
  yMin?: number
  yMax?: number
  revealCount?: number
}>(), {
  annotations: () => [],
  width: 560,
  height: 240,
  yMin: 0,
})

// SVG margins — ML wider to fit 4-digit y-labels at 13px
const ML = 72 // left  (y-axis labels)
const MR = 20 // right
const MT = 28 // top   (annotation labels)
const MB = 50 // bottom (x-axis labels)

const plotW = computed(() => props.width - ML - MR)
const plotH = computed(() => props.height - MT - MB)

const allX = computed(() => props.series.flatMap(s => s.data.map(d => d.x)))
const allY = computed(() => props.series.flatMap(s => s.data.map(d => d.value)))

const xMin = computed(() => props.xMin ?? Math.min(...allX.value))
const xMax = computed(() => props.xMax ?? Math.max(...allX.value))
const yMin = computed(() => props.yMin ?? 0)
const yMax = computed(() => props.yMax ?? Math.ceil(Math.max(...allY.value) * 1.1))

// Which series are currently visible (controlled by revealCount prop)
const visibleIds = computed(() => {
  if (props.revealCount === undefined) return new Set(props.series.map(s => s.id))
  return new Set(props.series.slice(0, props.revealCount).map(s => s.id))
})

// Map data coords → SVG pixel coords
function toSvgX(x: number) {
  return ML + ((x - xMin.value) / (xMax.value - xMin.value)) * plotW.value
}
function toSvgY(y: number) {
  return MT + plotH.value - ((y - yMin.value) / (yMax.value - yMin.value)) * plotH.value
}

// ~5 horizontal gridlines
const yTicks = computed(() => {
  const step = yMax.value / 5
  return Array.from({ length: 6 }, (_, i) => Math.round(i * step))
})

// All unique x values for ticks
const xTicks = computed(() => [...new Set(allX.value)].sort((a, b) => a - b))

// Decide whether to show every x tick or every-other based on density
const xTickStep = computed(() => (xTicks.value.length > 8 ? 2 : 1))

// Stagger annotation labels vertically when x-positions are close (< 60px apart)
const annotationsWithY = computed(() => {
  const list = [...(props.annotations ?? [])]
  return list.map((ann, i) => {
    const prevX = i > 0 ? toSvgX(list[i - 1].x) : -Infinity
    const isClose = toSvgX(ann.x) - prevX < 60
    return { ...ann, labelY: MT - 6 + (isClose ? 14 : 0) }
  })
})

// Polyline points string
function seriesPoints(s: Series) {
  return s.data.map(d => `${toSvgX(d.x)},${toSvgY(d.value)}`).join(' ')
}

// Closed path for filled area under the line
function fillPath(s: Series) {
  if (!s.data.length) return ''
  const first = s.data[0]
  const last = s.data[s.data.length - 1]
  const linePts = s.data.map(d => `${toSvgX(d.x)},${toSvgY(d.value)}`).join(' L')
  const base = toSvgY(yMin.value)
  return `M${toSvgX(first.x)},${base} L${linePts} L${toSvgX(last.x)},${base} Z`
}

const REVEAL_TRANSITION = 'opacity 0.45s ease'
</script>

<template>
  <div class="line-chart">
    <svg
      :width="width"
      :height="height"
      :viewBox="`0 0 ${width} ${height}`"
      overflow="visible"
    >
      <!-- Horizontal grid lines -->
      <line
        v-for="t in yTicks"
        :key="`gy-${t}`"
        :x1="ML" :y1="toSvgY(t)"
        :x2="ML + plotW" :y2="toSvgY(t)"
        stroke="currentColor" stroke-opacity="0.12" stroke-width="1"
      />

      <!-- Axes -->
      <line :x1="ML" :y1="MT" :x2="ML" :y2="MT + plotH"
        stroke="currentColor" stroke-opacity="0.35" stroke-width="1" />
      <line :x1="ML" :y1="MT + plotH" :x2="ML + plotW" :y2="MT + plotH"
        stroke="currentColor" stroke-opacity="0.35" stroke-width="1" />

      <!-- Y-axis ticks + labels -->
      <g
        v-for="t in yTicks"
        :key="`yt-${t}`"
        style="font-size:13px"
        fill="currentColor"
        fill-opacity="0.65"
        font-family="JetBrains Mono, monospace"
      >
        <line :x1="ML - 4" :y1="toSvgY(t)" :x2="ML" :y2="toSvgY(t)"
          stroke="currentColor" stroke-opacity="0.35" />
        <text :x="ML - 7" :y="toSvgY(t)" text-anchor="end" dominant-baseline="middle">
          {{ t }}
        </text>
      </g>

      <!-- X-axis ticks + labels -->
      <g
        v-for="(x, i) in xTicks"
        :key="`xt-${x}`"
        style="font-size:13px"
        fill="currentColor"
        fill-opacity="0.65"
        font-family="JetBrains Mono, monospace"
      >
        <template v-if="i % xTickStep === 0">
          <line :x1="toSvgX(x)" :y1="MT + plotH" :x2="toSvgX(x)" :y2="MT + plotH + 4"
            stroke="currentColor" stroke-opacity="0.35" />
          <text :x="toSvgX(x)" :y="MT + plotH + 20" text-anchor="middle">{{ x }}</text>
        </template>
      </g>

      <!-- Axis labels -->
      <text
        v-if="xLabel"
        :x="ML + plotW / 2" :y="height - 4"
        text-anchor="middle" style="font-size:12px" fill="currentColor" fill-opacity="0.5"
        font-family="JetBrains Mono, monospace"
      >{{ xLabel }}</text>
      <text
        v-if="yLabel"
        :x="11" :y="MT + plotH / 2"
        text-anchor="middle" style="font-size:12px" fill="currentColor" fill-opacity="0.5"
        font-family="JetBrains Mono, monospace"
        :transform="`rotate(-90, 11, ${MT + plotH / 2})`"
      >{{ yLabel }}</text>

      <!-- Per-series groups: fill + line + dots, with opacity reveal -->
      <g
        v-for="s in series"
        :key="`series-${s.id}`"
        :style="{ opacity: visibleIds.has(s.id) ? 1 : 0, transition: REVEAL_TRANSITION }"
      >
        <!-- Fill area (rendered behind line) -->
        <path
          v-if="s.fill"
          :d="fillPath(s)"
          :fill="s.color"
          fill-opacity="0.18"
          stroke="none"
        />
        <!-- Series polyline -->
        <polyline
          :points="seriesPoints(s)"
          :stroke="s.color"
          stroke-width="2.5"
          fill="none"
          :stroke-dasharray="s.dash ? '6 3' : undefined"
        />
        <!-- Data points -->
        <circle
          v-for="d in s.data"
          :key="`dot-${s.id}-${d.x}`"
          :cx="toSvgX(d.x)" :cy="toSvgY(d.value)"
          r="3" :fill="s.color"
        />
      </g>

      <!-- Annotations: vertical dashed lines + staggered labels -->
      <g v-for="ann in annotationsWithY" :key="`ann-${ann.x}`">
        <line
          :x1="toSvgX(ann.x)" :y1="MT"
          :x2="toSvgX(ann.x)" :y2="MT + plotH"
          :stroke="ann.color ?? '#888'"
          stroke-dasharray="4 3" stroke-width="1" stroke-opacity="0.65"
        />
        <text
          :x="toSvgX(ann.x) + 3" :y="ann.labelY"
          style="font-size:9px" :fill="ann.color ?? 'currentColor'" fill-opacity="0.8"
          font-family="JetBrains Mono, monospace"
        >{{ ann.label }}</text>
      </g>
    </svg>

    <!-- Legend row — items fade in together with their series line -->
    <div class="line-chart__legend">
      <span
        v-for="s in series"
        :key="`leg-${s.id}`"
        class="line-chart__legend-item"
        :style="{ opacity: visibleIds.has(s.id) ? 1 : 0, transition: REVEAL_TRANSITION }"
      >
        <svg width="24" height="12" style="display: inline-block; vertical-align: middle">
          <line x1="0" y1="6" x2="24" y2="6"
            :stroke="s.color" stroke-width="2.5"
            :stroke-dasharray="s.dash ? '5 3' : undefined" />
        </svg>
        {{ s.label }}
      </span>
    </div>
  </div>
</template>

<style scoped>
.line-chart {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  max-width: 700px;
  margin: 0 auto;
}

.line-chart__legend {
  display: flex;
  gap: 1.5rem;
  flex-wrap: wrap;
  justify-content: center;
  font-size: 0.85rem;
  font-family: 'JetBrains Mono', monospace;
  opacity: 0.9;
}

.line-chart__legend-item {
  display: flex;
  align-items: center;
  gap: 0.35rem;
}
</style>
