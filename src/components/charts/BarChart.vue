<script setup>
import { computed } from 'vue'

const props = defineProps({
  data: { type: Array, default: () => [] },
  color: { type: String, default: '' },
  formatter: { type: Function, default: (v) => v },
  max: { type: Number, default: null },
  maxLabelLength: { type: Number, default: 10 },
  emptyText: { type: String, default: '暂无数据' },
})

const maxValue = computed(
  () => props.max ?? Math.max(1, ...props.data.map((d) => Number(d.value) || 0))
)

function percent(value) {
  return Math.round(((Number(value) || 0) / maxValue.value) * 100)
}

// 超长标签截断，完整名称通过 title 悬停展示
function truncateLabel(label) {
  const text = String(label ?? '')
  return text.length > props.maxLabelLength
    ? text.slice(0, props.maxLabelLength) + '…'
    : text
}
</script>

<template>
  <div v-if="data.length" class="hbar">
    <div v-for="(d, i) in data" :key="i" class="hbar-row">
      <span class="hbar-label" :title="d.label">{{ truncateLabel(d.label) }}</span>
      <div class="hbar-track">
        <div
          class="hbar-fill"
          :style="{ width: percent(d.value) + '%', background: color || 'var(--primary)' }"
        ></div>
      </div>
      <span class="hbar-val">{{ formatter(d.value) }}</span>
    </div>
  </div>
  <div v-else class="hbar-empty">{{ emptyText }}</div>
</template>

<style scoped>
.hbar-row {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
}

.hbar-label {
  width: 110px;
  flex-shrink: 0;
  font-size: 13px;
  color: var(--text-secondary);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.hbar-track {
  flex: 1;
  height: 16px;
  background: #eef0f4;
  border-radius: 8px;
  overflow: hidden;
}

.hbar-fill {
  height: 100%;
  border-radius: 8px;
  transition: width 0.4s;
  min-width: 2px;
}

.hbar-val {
  width: 80px;
  flex-shrink: 0;
  text-align: right;
  font-size: 13px;
  font-weight: 500;
}

.hbar-empty {
  padding: 32px 0;
  text-align: center;
  font-size: 13px;
  color: var(--text-muted);
}
</style>
