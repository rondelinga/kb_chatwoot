<script setup>
defineProps({
  active: {
    type: Boolean,
    default: false,
  },
  title: {
    type: String,
    default: '',
  },
  description: {
    type: String,
    default: '',
  },
});
</script>

<template>
  <button
    class="flex flex-col gap-2 w-full px-4 py-3 rounded-xl border text-left transition-all duration-200 cursor-pointer"
    :class="active
      ? 'border-[rgba(74,222,128,0.5)] bg-[rgba(74,222,128,0.05)] shadow-[0_0_16px_rgba(74,222,128,0.15)]'
      : 'border-white/10 bg-white/5 hover:border-white/20'"
  >
    <span
      class="text-[10px] font-semibold tracking-[0.15em] uppercase"
      :class="active ? 'text-[#4ade80]' : 'text-n-slate-10'"
    >
      {{ title }}
    </span>
    <span class="text-xs text-n-slate-9 line-clamp-2">{{ description }}</span>
  </button>
</template>
