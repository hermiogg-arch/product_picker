<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { useData } from 'vitepress'
import mermaid from 'mermaid'

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
    mermaid.initialize({
      startOnLoad: true,
      theme: isDark.value ? 'dark' : 'default',
      securityLevel: 'loose'
    })

    const { svg } = await mermaid.render('mermaid-diagram', decodedCode)
    svgRef.value.innerHTML = svg
  } catch (error) {
    console.error('Mermaid render error:', error)
    svgRef.value.innerHTML = `<pre>${decodedCode}</pre>`
  }
}

onMounted(() => {
  renderDiagram()
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
