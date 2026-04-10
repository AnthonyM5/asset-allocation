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
    <form class="allocation">
      <label for="invest-amount" class="label">Investable Assets ($) </label>
      <input
        aria-label="Enter amount in USD"
        id="invest-amount"
        class="invest-amount"
        min="0"
        step="0.01"
        type="number"
        v-model.number="investAmount"
      />
    </form>
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
  <p v-if="loading" role="status">Loading exchange rate...</p>
  <p v-else-if="error" role="alert">Error: {{ error }}</p>
</template>

<style scoped>
.title {
  text-align: left;
}
.grid {
  display: flex;
  gap: 2rem;
  padding: 1rem;
  max-width: 100%;
}

.invest-amount {
  padding: 0.5rem;
  border-radius: 4px;
}

.invest-amount:focus-visible {
  outline: 1px solid #007bff;
  border-color: #007bff;
}

@media (min-width: 1024px) {
  .grid {
    gap: 20%;
  }
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
