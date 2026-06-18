<template>
  <Badge :variant="variant" class="flex items-center gap-1">
    <span :class="['w-2 h-2 rounded-full', dotClass]"></span>
    {{ label }}
  </Badge>
</template>

<script setup>
import { Badge } from '@/components/ui/badge'
import { computed, ref, onMounted, watch } from 'vue'
import { useRoute } from 'vue-router'

const props = defineProps({
  status: {
    type: String,
    required: true,
    validator: (value) => ['healthy', 'unhealthy', 'degraded', 'unknown'].includes(value)
  },
  endpointKey: {
    type: String,
    required: false,
    default: ''
  }
})

const route = useRoute()
const isHomepage = computed(() => route.path === '/')
const slaText = ref('')

const fetchSla = async () => {
  if (!isHomepage.value || !props.endpointKey) return

  try {
    const response = await fetch(`/api/v1/endpoints/${props.endpointKey}/uptimes/30d`)
    const rawText = await response.text()
    
    const numValue = parseFloat(rawText.trim())
    if (!isNaN(numValue)) {
      // Formats 1.000000 to "100.00%" or 0.998700 to "99.87%"
      slaText.value = `${(numValue * 100).toFixed(2)}%`
    }
  } catch (error) {
    console.error('Failed to fetch SLA:', error)
    slaText.value = ''
  }
}

onMounted(() => {
  fetchSla()
})

watch(() => props.endpointKey, (newKey) => {
  if (newKey) fetchSla()
})

const variant = computed(() => {
  switch (props.status) {
    case 'healthy':
      return 'success'
    case 'unhealthy':
      return 'destructive'
    case 'degraded':
      return 'warning'
    default:
      return 'secondary'
  }
})

// display label text or SLA override
const label = computed(() => {
  if (isHomepage.value && props.endpointKey && slaText.value) {
    return slaText.value
  }
  switch (props.status) {
    case 'healthy':
      return 'Healthy'
    case 'unhealthy':
      return 'Unhealthy'
    case 'degraded':
      return 'Degraded'
    default:
      return 'Unknown'
  }
})

const dotClass = computed(() => {
  switch (props.status) {
    case 'healthy':
      return 'bg-green-400'
    case 'unhealthy':
      return 'bg-red-400'
    case 'degraded':
      return 'bg-yellow-400'
    default:
      return 'bg-gray-400'
  }
})
</script>
