<template>
  <div class="search-box">
    <svg class="search-icon" width="16" height="16" viewBox="0 0 24 24" fill="none"
      stroke="currentColor" stroke-width="2">
      <circle cx="11" cy="11" r="8"/>
      <line x1="21" y1="21" x2="16.65" y2="16.65"/>
    </svg>
    <input
      ref="inputRef"
      v-model="query"
      type="text"
      :placeholder="placeholder"
      class="search-input"
      @input="handleInput"
      @keydown.escape="clear"
    />
    <button v-if="query" class="clear-btn" @click="clear" type="button">
      <svg width="14" height="14" viewBox="0 0 24 24" fill="none"
        stroke="currentColor" stroke-width="2.5">
        <line x1="18" y1="6" x2="6" y2="18"/>
        <line x1="6" y1="6" x2="18" y2="18"/>
      </svg>
    </button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  placeholder: { type: String, default: '검색...' },
  debounce:    { type: Number, default: 250 }
})

const emit = defineEmits(['search'])

const query    = ref('')
const inputRef = ref(null)
let   timer    = null

function handleInput() {
  clearTimeout(timer)
  timer = setTimeout(() => emit('search', query.value.trim()), props.debounce)
}

function clear() {
  query.value = ''
  emit('search', '')
  inputRef.value?.focus()
}

defineExpose({ clear })
</script>

<style scoped>
.search-box {
  position: relative;
  display: flex;
  align-items: center;
  min-width: 260px;
}

.search-icon {
  position: absolute;
  left: 10px;
  color: #9CA3AF;
  pointer-events: none;
}

.search-input {
  width: 100%;
  padding: 8px 34px 8px 34px;
  border: 1px solid #E5E7EB;
  border-radius: 8px;
  font-size: 14px;
  background: #fff;
  color: #111827;
  outline: none;
  transition: border-color 0.15s;
}

.search-input:focus {
  border-color: #6366F1;
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.1);
}

.search-input::placeholder { color: #9CA3AF; }

.clear-btn {
  position: absolute;
  right: 8px;
  background: none;
  border: none;
  cursor: pointer;
  color: #9CA3AF;
  display: flex;
  align-items: center;
  padding: 2px;
  border-radius: 4px;
  transition: color 0.15s;
}

.clear-btn:hover { color: #374151; }
</style>
