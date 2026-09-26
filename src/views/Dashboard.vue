<script setup>
import { computed, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useTravelStore } from '../stores/travel'
import { TRIP_TYPES } from '../constants'
import { formatMoney } from '../utils/format'
import BarChart from '../components/charts/BarChart.vue'
import DonutChart from '../components/charts/DonutChart.vue'

const store = useTravelStore()
const router = useRouter()

const stats = computed(() => store.dashboardStats)
const money = (v) => formatMoney(v)

// 各次出行花费对比：按出行类型筛选，空串表示全部
const tripTypeFilter = ref('')
const perTripSpend = computed(() =>
  tripTypeFilter.value
    ? stats.value.perTripSpend.filter((d) => d.tripType === tripTypeFilter.value)
    : stats.value.perTripSpend
)

const statCards = computed(() => [
  { label: '累计出行次数', value: stats.value.totalTrips, suffix: '次' },
  { label: '总花费', value: money(stats.value.totalSpend), suffix: '' },
  { label: '最常去目的地', value: stats.value.mostVisited, suffix: '' },
  { label: '平均每次出行天数', value: stats.value.avgDays, suffix: '天' },
  { label: '行李打包完成率', value: stats.value.avgPackingRate, suffix: '%' },
])
</script>

<template>
  <div>
    <div v-if="store.plans.length" class="stat-grid">
      <div v-for="(card, i) in statCards" :key="i" class="stat-card">
        <span class="stat-label">{{ card.label }}</span>
        <strong class="stat-value">{{ card.value }}<small v-if="card.suffix">{{ card.suffix }}</small></strong>
      </div>
    </div>

    <div v-else class="card empty">
      <p class="empty-icon">+</p>
      <p>还没有出行计划，先创建你的第一个出行计划吧</p>
      <button class="btn btn-primary mt-16" @click="router.push('/plans/new')">新建出行计划</button>
    </div>

    <template v-if="store.plans.length">
      <div class="grid-2 mt-16">
        <div class="card">
          <h3 class="card-title">出行类型分布</h3>
          <BarChart :data="stats.tripTypeDist" />
        </div>
        <div class="card">
          <h3 class="card-title">花费分类汇总</h3>
          <DonutChart :data="stats.categorySpend" :formatter="money" />
        </div>
      </div>

      <div class="card mt-16">
        <h3 class="card-title">
          各次出行花费对比
          <select v-model="tripTypeFilter" class="select type-filter" aria-label="按出行类型筛选">
            <option value="">全部类型</option>
            <option v-for="t in TRIP_TYPES" :key="t" :value="t">{{ t }}</option>
          </select>
        </h3>
        <BarChart
          :data="perTripSpend"
          :formatter="money"
          color="#10b981"
          empty-text="该出行类型下暂无计划，换个类型看看"
        />
      </div>
    </template>
  </div>
</template>

<style scoped>
.stat-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 16px;
}

.stat-card {
  background: var(--surface);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  padding: 18px;
  display: flex;
  flex-direction: column;
}

.stat-label {
  font-size: 13px;
  color: var(--text-secondary);
}

.stat-value {
  font-size: 26px;
  font-weight: 700;
  margin-top: 6px;
}

.stat-value small {
  font-size: 14px;
  font-weight: 500;
  color: var(--text-secondary);
  margin-left: 2px;
}

.type-filter {
  width: auto;
  padding: 5px 10px;
  font-size: 13px;
  font-weight: 400;
}
</style>
