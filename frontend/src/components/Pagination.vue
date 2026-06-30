<template>
  <div v-if="totalPages > 1" class="pagination">
    <button
      class="page-btn"
      :disabled="current === 1"
      @click="emit('change', current - 1)"
    >
      <svg width="14" height="14" viewBox="0 0 24 24" fill="none"
        stroke="currentColor" stroke-width="2.5">
        <polyline points="15 18 9 12 15 6"/>
      </svg>
    </button>

    <template v-for="page in pageRange" :key="page">
      <span v-if="page === '...'" class="page-ellipsis">…</span>
      <button
        v-else
        class="page-btn"
        :class="{ active: page === current }"
        @click="emit('change', page)"
      >{{ page }}</button>
    </template>

    <button
      class="page-btn"
      :disabled="current === totalPages"
      @click="emit('change', current + 1)"
    >
      <svg width="14" height="14" viewBox="0 0 24 24" fill="none"
        stroke="currentColor" stroke-width="2.5">
        <polyline points="9 18 15 12 9 6"/>
      </svg>
    </button>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  current:    { type: Number, required: true },
  totalPages: { type: Number, required: true }
})

const emit = defineEmits(['change'])

// 최대 7개 버튼: 처음/끝 + 현재 주변 2개 + 생략 부호
const pageRange = computed(() => {
  const total = props.totalPages
  const cur   = props.current
  if (total <= 7) return Array.from({ length: total }, (_, i) => i + 1)

  const pages = []
  pages.push(1)
  if (cur > 3)           pages.push('...')
  for (let i = Math.max(2, cur - 1); i <= Math.min(total - 1, cur + 1); i++) {
    pages.push(i)
  }
  if (cur < total - 2)   pages.push('...')
  pages.push(total)
  return pages
})
</script>

<style scoped>
.pagination {
  display: flex;
  align-items: center;
  gap: 4px;
  justify-content: center;
  margin-top: 20px;
}

.page-btn {
  min-width: 34px;
  height: 34px;
  padding: 0 6px;
  border: 1px solid #E5E7EB;
  border-radius: 6px;
  background: #fff;
  color: #374151;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.15s;
}

.page-btn:hover:not(:disabled):not(.active) {
  background: #F3F4F6;
  border-color: #D1D5DB;
}

.page-btn.active {
  background: #6366F1;
  border-color: #6366F1;
  color: #fff;
}

.page-btn:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}

.page-ellipsis {
  min-width: 34px;
  text-align: center;
  color: #9CA3AF;
  font-size: 13px;
}
</style>
