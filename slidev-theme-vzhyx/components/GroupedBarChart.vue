<script setup lang="ts">
import { computed } from 'vue'

interface Framework {
  id: string
  label: string
  color: string
}

interface Criterion {
  id: string
  label: string
  values: Record<string, number> // frameworkId → 0–10 normalised score
}

const props = withDefaults(defineProps<{
  criteria: Criterion[]
  frameworks: Framework[]
  width?: number
  height?: number
  maxValue?: number
}>(), {
  width: 560,
  height: 240,
  maxValue: 10,
})

// SVG margins
const ML = 46 // left  (y-axis labels)
const MR = 16 // right
const MT = 16 // top
const MB = 62 // bottom (criterion labels, rotated — extra space)

const plotW = computed(() => props.width - ML - MR)
const plotH = computed(() => props.height - MT - MB)

const nCriteria = computed(() => props.criteria.length)
const nFrameworks = computed(() => props.frameworks.length)

// Each criterion occupies an equal column; bars fill 80% of it
const groupW = computed(() => plotW.value / nCriteria.value)
const barPad = computed(() => groupW.value * 0.1)        // 10% left+right gap
const barAreaW = computed(() => groupW.value * 0.8)
const barW = computed(() => barAreaW.value / nFrameworks.value)

// Map 0–maxValue → SVG bar height
function barH(value: number) {
  return (value / props.maxValue) * plotH.value
}

// X origin for a bar: criterion index i, framework index j
function barX(i: number, j: number) {
  return ML + i * groupW.value + barPad.value + j * barW.value
}

// Y origin (top of bar — SVG y increases downward)
function barY(value: number) {
  return MT + plotH.value - barH(value)
}

// Centre x of a criterion group (for label)
function groupCx(i: number) {
  return ML + i * groupW.value + groupW.value / 2
}

// Y-axis tick values: 0, 2, 4, 6, 8, 10
const yTicks = computed(() =>
  Array.from({ length: props.maxValue / 2 + 1 }, (_, i) => i * 2)
)

function toSvgY(v: number) {
  return MT + plotH.value - (v / props.maxValue) * plotH.value
}
</script>

<template>
  <div class="bar-chart">
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
        style="font-size:10px"
        fill="currentColor"
        fill-opacity="0.65"
        font-family="JetBrains Mono, monospace"
      >
        <line :x1="ML - 3" :y1="toSvgY(t)" :x2="ML" :y2="toSvgY(t)"
          stroke="currentColor" stroke-opacity="0.35" />
        <text :x="ML - 5" :y="toSvgY(t)" text-anchor="end" dominant-baseline="middle">
          {{ t }}
        </text>
      </g>

      <!-- Bars per criterion per framework -->
      <template v-for="(criterion, i) in criteria" :key="`group-${criterion.id}`">
        <rect
          v-for="(fw, j) in frameworks"
          :key="`bar-${criterion.id}-${fw.id}`"
          :x="barX(i, j)"
          :y="barY(criterion.values[fw.id] ?? 0)"
          :width="barW - 1"
          :height="barH(criterion.values[fw.id] ?? 0)"
          :fill="fw.color"
          fill-opacity="0.85"
          rx="2"
        />

        <!-- Criterion label below group (rotated -40°) -->
        <text
          :x="groupCx(i)"
          :y="MT + plotH + 6"
          text-anchor="end"
          dominant-baseline="auto"
          style="font-size:9px"
          fill="currentColor"
          fill-opacity="0.7"
          font-family="JetBrains Mono, monospace"
          :transform="`rotate(-40, ${groupCx(i)}, ${MT + plotH + 6})`"
        >{{ criterion.label }}</text>
      </template>
    </svg>

    <!-- Legend row -->
    <div class="bar-chart__legend">
      <span v-for="fw in frameworks" :key="`leg-${fw.id}`" class="bar-chart__legend-item">
        <span class="bar-chart__legend-swatch" :style="{ background: fw.color }" />
        {{ fw.label }}
      </span>
    </div>
  </div>
</template>

<style scoped>
.bar-chart {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.4rem;
  /* Prevent Slidev from stretching the SVG beyond its natural width */
  max-width: 540px;
  margin: 0 auto;
}

.bar-chart__legend {
  display: flex;
  gap: 1.25rem;
  flex-wrap: wrap;
  justify-content: center;
  font-size: 0.7rem;
  font-family: 'JetBrains Mono', monospace;
  opacity: 0.85;
}

.bar-chart__legend-item {
  display: flex;
  align-items: center;
  gap: 0.35rem;
}

.bar-chart__legend-swatch {
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 2px;
  flex-shrink: 0;
}
</style>
