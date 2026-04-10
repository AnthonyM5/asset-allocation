<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  idPrefix: string
  symbol: string
  weight: number
  investAmount: number
  rate: number | null
}>()

const labelText = computed(() => `${Math.round(props.weight * 100)}% ${props.symbol} allocation`)

const outputText = computed(() => {
  if (props.rate == null) return 'N/A'
  const amount = props.rate * (props.investAmount * props.weight)
  return `${amount} ${props.symbol}`
})
</script>

<template>
  <label :for="`${idPrefix}-allocation`" class="label">{{ labelText }}</label>
  <output :id="`${idPrefix}-allocation`" class="output-allocation" aria-live="polite">
    {{ outputText }}
  </output>
</template>

<style scoped>
.output-allocation {
  border-color: #f0f0f0;
  padding: 0.5rem;
  border: solid 1px;
  border-radius: 4px;
}

.label {
  font-weight: bold;
}
</style>
