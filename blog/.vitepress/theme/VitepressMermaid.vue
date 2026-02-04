<script setup lang="ts">
import { onMounted, ref, watch, nextTick } from 'vue'
import { useData } from 'vitepress'

interface Props {
  code: string
}

const props = defineProps<Props>()
const { isDark } = useData()
const svgRef = ref<HTMLDivElement>()
const decodedCode = decodeURIComponent(props.code)

const renderDiagram = async () => {
  if (!svgRef.value) return

  try {
    // Dynamic import to avoid SSR issues
    const mermaid = (await import('mermaid')).default
    
    mermaid.initialize({
      startOnLoad: false,
      theme: isDark.value ? 'dark' : 'default',
      securityLevel: 'loose'
    })

    const id = `mermaid-${Math.random().toString(36).substring(7)}`
    const { svg } = await mermaid.render(id, decodedCode)
    svgRef.value.innerHTML = svg
  } catch (error) {
    console.error('Mermaid render error:', error)
    svgRef.value.innerHTML = `<pre style="text-align: left; overflow-x: auto;">${decodedCode}</pre>`
  }
}

onMounted(() => {
  nextTick(() => {
    renderDiagram()
  })
})

watch(() => isDark.value, () => {
  nextTick(() => {
    renderDiagram()
  })
})
</script>

<template>
  <div ref="svgRef" class="mermaid-container"></div>
</template>

<style scoped>
.mermaid-container {
  display: flex;
  justify-content: center;
  margin: 1.5rem 0;
  overflow-x: auto;
}

.mermaid-container :deep(svg) {
  max-width: 100%;
  height: auto;
}
</style>
