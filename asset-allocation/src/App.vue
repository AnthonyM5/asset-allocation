<script setup lang="ts">
import { ref, onMounted } from 'vue'

const investAmount = ref(0)
const loading = ref(false)
const usdToBtc = ref<number | null>(null)
const usdToEth = ref<number | null>(null)
const error = ref<string | null>(null)

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
        v-model="investAmount"
      />
    </div>
    <div class="allocation">
      <label for="btc-allocation" class="label">70% BTC allocation</label>
      <output
        id="btc-allocation"
        class="output-allocation"
        type="text"
        :value="usdToBtc ? usdToBtc * (investAmount * 0.7) + ' BTC' : 'N/A'"
      />
      <label for="eth-allocation" class="label">30% ETH allocation</label>
      <output
        id="eth-allocation"
        class="output-allocation"
        type="text"
        :value="usdToEth ? usdToEth * (investAmount * 0.3) + ' ETH' : 'N/A'"
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

.output-allocation {
  border-color: #f0f0f0;
  padding: 0.5rem;
  border: solid 1px;
  border-radius: 4px;
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
