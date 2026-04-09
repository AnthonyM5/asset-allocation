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
  <label>Investable Assets:</label>
  <input type="number" v-model="investAmount" />

  <p v-if="loading">Loading exchange rate...</p>
  <p v-else-if="error">Error: {{ error }}</p>
  <template v-else>
    <p v-if="usdToEth">70% ETH Allocation = {{ usdToEth * (investAmount * 0.7) }} ETH</p>
    <p v-if="usdToBtc">30% BTC Allocation = {{ usdToEth * (investAmount * 0.3) }} BTC</p>
  </template>
</template>

<style scoped></style>
