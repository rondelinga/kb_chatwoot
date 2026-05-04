<script setup>
defineProps({
  label: { type: String, required: true },
  items: { type: Array, required: true },
});

const emit = defineEmits(['change']);
const onChange = (id, value) => emit('change', id, value);
</script>

<template>
  <div class="flex flex-col gap-2">
    <span
      class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
      >{{ label }}</span
    >
    <div class="flex flex-col gap-2">
      <div
        v-for="item in items"
        :key="item.id"
        class="flex items-center gap-3 px-4 py-2.5 rounded-xl border border-white/10 bg-white/5 transition-all duration-200 hover:border-[rgba(74,222,128,0.2)] cursor-pointer"
        @click="onChange(item.value, !item.model)"
      >
        <button
          type="button"
          class="w-5 h-5 rounded-md border transition-all duration-200 flex items-center justify-center shrink-0"
          :class="
            item.model
              ? 'bg-[rgba(74,222,128,0.15)] border-[rgba(74,222,128,0.6)] shadow-[0_0_8px_rgba(74,222,128,0.2)]'
              : 'border-white/20 bg-white/5'
          "
        >
          <fluent-icon
            v-if="item.model"
            icon="checkmark"
            size="10"
            class="text-[#4ade80]"
          />
        </button>
        <span class="text-sm text-n-slate-9">{{ item.label }}</span>
      </div>
    </div>
  </div>
</template>
