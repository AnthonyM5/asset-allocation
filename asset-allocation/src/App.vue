<script setup lang="ts">
import { computed, ref, onMounted } from 'vue'
import AllocationRow from './AllocationRow.vue'

type AllocationRule = {
  idPrefix: string
  symbol: string
  weight: number
}

const allocationRules: AllocationRule[] = [
  { idPrefix: 'btc', symbol: 'BTC', weight: 0.7 },
  { idPrefix: 'eth', symbol: 'ETH', weight: 0.3 },
]

const investAmount = ref(0)
const loading = ref(false)
const usdToBtc = ref<number | null>(null)
const usdToEth = ref<number | null>(null)
const error = ref<string | null>(null)

const ratesBySymbol = computed<Record<string, number | null>>(() => ({
  BTC: usdToBtc.value,
  ETH: usdToEth.value,
}))

async function fetchData() {
  loading.value = true
  error.value = null

  try {
    const response = await fetch('https://api.coinbase.com/v2/exchange-rates?currency=USD')
    if (!response.ok) throw new Error(`HTTP ${response.status}`)

    const data = await response.json()
    usdToBtc.value = data?.data?.rates?.BTC ?? null
    usdToEth.value = data?.data?.rates?.ETH ?? null
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'Failed to fetch exchange rate'
  } finally {
    loading.value = false
  }
}

onMounted(fetchData)
</script>

<template>
  <h1 class="title">Asset Allocation Calculator</h1>
  <section class="grid">
    <div class="allocation">
      <label for="invest-amount" class="label">Investable assets </label>
      <input
        id="invest-amount"
        class="invest-amount"
        min="0"
        step="0.01"
        type="number"
        v-model.number="investAmount"
      />
    </div>
    <div class="allocation">
      <AllocationRow
        v-for="rule in allocationRules"
        :key="rule.symbol"
        :id-prefix="rule.idPrefix"
        :symbol="rule.symbol"
        :weight="rule.weight"
        :invest-amount="investAmount"
        :rate="ratesBySymbol[rule.symbol] ?? null"
      />
    </div>
  </section>
  <p v-if="loading">Loading exchange rate...</p>
  <p v-else-if="error">Error: {{ error }}</p>
</template>

<style scoped>
.title {
  text-align: center;
}
.grid {
  display: flex;
  gap: 10rem;
  padding: 2rem;
}

.invest-amount {
  padding: 0.5rem;
  border-radius: 4px;
}

.allocation {
  display: flex;
  gap: 1rem;
  flex-direction: column;
}

.label {
  font-weight: bold;
}
</style>
