<script setup>
import { computed } from 'vue';
import { useFontSize } from 'dashboard/composables/useFontSize';

const props = defineProps({
  value: { type: String, default: 'default' },
  label: { type: String, default: '' },
  description: { type: String, default: '' },
});

const emit = defineEmits(['change']);
const { fontSizeOptions } = useFontSize();

const selectedValue = computed({
  get: () => props.value,
  set: value => emit('change', value),
});
</script>

<template>
  <div
    class="flex gap-3 justify-between w-full items-center rounded-xl border border-white/10 bg-white/5 px-4 py-3 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)]"
  >
    <div class="flex flex-col gap-0.5">
      <span
        class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
        >{{ label }}</span
      >
      <p class="text-xs text-n-slate-9">{{ description }}</p>
    </div>
    <select
      v-model="selectedValue"
      class="bg-white/5 border border-white/10 rounded-lg text-sm text-n-slate-9 px-2 py-1 outline-none cursor-pointer hover:border-[rgba(74,222,128,0.4)] transition-all duration-200 min-w-28"
    >
      <option
        v-for="option in fontSizeOptions"
        :key="option.value"
        :value="option.value"
        class="bg-n-solid-3"
      >
        {{ option.label }}
      </option>
    </select>
  </div>
</template>
