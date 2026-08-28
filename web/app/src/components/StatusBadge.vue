<template>
  <div class="inline-flex items-center rounded-full border px-2.5 py-0.5 text-xs font-semibold transition-colors
              focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2 border-transparent bg-primary
              text-primary-foreground hover:bg-primary/80 flex items-center gap-1 text-white"
       :style="`background-color: ${color};`">
    <span :style="`background-color: ${color}; filter: brightness(115%)`" class="w-2 h-2 rounded-full"></span>
    {{ label }}
  </div>
</template>

<script setup>
import { computed , ref, onMounted, watch} from 'vue'
import { getStateColor } from '@/utils/color'
import { useRoute } from 'vue-router'
import { Badge } from '@/components/ui/badge'

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
  if (!props.status) return 'Unknown'
  return props.status.charAt(0).toUpperCase() + props.status.slice(1).replace(/_/g, ' ') // TODO#227 Capitalize every word
})

const color = computed(() => {
  if (!props.status) return window.config?.localStateColors.unknown
  return getStateColor(props.status)
})
</script>
