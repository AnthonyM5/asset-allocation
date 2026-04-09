<script setup lang="ts">
import { ref, onMounted } from 'vue'

const investAmount = ref(0)
const loading = ref(false)
const usdToBtc = ref<string | null>(null)
const usdToEth = ref<string | null>(null)
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
    <label class="label">Investable assets</label>
    <input type="number" v-model="investAmount" />
    <label class="label">70% BTC allocation</label>
    <input
      class="input"
      type="text"
      :value="usdToBtc ? usdToBtc * (investAmount * 0.7) + ' BTC' : 'N/A'"
      readonly
    />
    <label class="label">30% ETH allocation</label>
    <input
      class="input"
      type="text"
      :value="usdToEth ? usdToEth * (investAmount * 0.3) + ' ETH' : 'N/A'"
      readonly
    />
  </section>
  <p v-if="loading">Loading exchange rate...</p>
  <p v-else-if="error">Error: {{ error }}</p>
</template>

<style scoped>
.title {
  text-align: center;
}
.grid {
  display: grid;
  gap: 1rem;
  max-width: 400px;
  margin: auto;
  padding: 2rem;
}
</style>
