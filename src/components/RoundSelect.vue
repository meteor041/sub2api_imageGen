<script setup lang="ts">
import { computed, ref } from 'vue'

interface RoundSelectOption {
  value: string | number
  label: string
  hint?: string
}

const props = withDefaults(defineProps<{
  modelValue: string | number | null
  options: RoundSelectOption[]
  disabled?: boolean
  title?: string
  placeholder?: string
}>(), {
  disabled: false,
  title: '',
  placeholder: '请选择'
})

const emit = defineEmits<{
  (event: 'update:modelValue', value: string | number): void
}>()

const detailsRef = ref<HTMLDetailsElement | null>(null)

const selectedIndex = computed(() => props.options.findIndex((option) => option.value === props.modelValue))
const selectedOption = computed(() => (
  selectedIndex.value >= 0 ? props.options[selectedIndex.value] : null
))
const canStep = computed(() => !props.disabled && props.options.length > 1)

function closeMenu(): void {
  if (detailsRef.value) {
    detailsRef.value.open = false
  }
}

function selectOption(value: string | number): void {
  if (props.disabled) {
    return
  }
  emit('update:modelValue', value)
  closeMenu()
}

function stepSelection(offset: number): void {
  if (!canStep.value) {
    return
  }
  const currentIndex = selectedIndex.value >= 0 ? selectedIndex.value : 0
  const nextIndex = (currentIndex + offset + props.options.length) % props.options.length
  emit('update:modelValue', props.options[nextIndex].value)
}
</script>

<template>
  <div class="round-select" :class="{ disabled }">
    <details ref="detailsRef" class="round-select-menu">
      <summary :title="title || selectedOption?.label || placeholder">
        <span class="round-select-value">{{ selectedOption?.label || placeholder }}</span>
        <svg viewBox="0 0 24 24" aria-hidden="true">
          <path d="M6 9l6 6 6-6" />
        </svg>
      </summary>
      <div class="round-select-list" role="listbox" :aria-label="title || placeholder">
        <button
          v-for="option in options"
          :key="String(option.value)"
          class="round-select-option"
          :class="{ active: option.value === modelValue }"
          type="button"
          @click="selectOption(option.value)"
        >
          <span>{{ option.label }}</span>
          <small v-if="option.hint">{{ option.hint }}</small>
        </button>
      </div>
    </details>
    <div class="round-select-stepper">
      <button
        type="button"
        :disabled="!canStep"
        :aria-label="`${title || placeholder}上一项`"
        @click="stepSelection(-1)"
      >
        <svg viewBox="0 0 24 24" aria-hidden="true">
          <path d="M6 15l6-6 6 6" />
        </svg>
      </button>
      <button
        type="button"
        :disabled="!canStep"
        :aria-label="`${title || placeholder}下一项`"
        @click="stepSelection(1)"
      >
        <svg viewBox="0 0 24 24" aria-hidden="true">
          <path d="M6 9l6 6 6-6" />
        </svg>
      </button>
    </div>
  </div>
</template>
