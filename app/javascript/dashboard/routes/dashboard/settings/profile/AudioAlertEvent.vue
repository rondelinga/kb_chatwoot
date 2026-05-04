<script setup>
import { computed } from 'vue';
import { ALERT_EVENTS, EVENT_TYPES } from './constants';

const props = defineProps({
  label: { type: String, default: '' },
  value: { type: String, default: '' },
});

const emit = defineEmits(['update']);

const alertEvents = ALERT_EVENTS;
const alertEventValues = Object.values(EVENT_TYPES);

const selectedValue = computed({
  get: () => {
    if (props.value === 'none') return [];
    if (props.value === 'mine') return [EVENT_TYPES.ASSIGNED];
    if (props.value === 'all') return [...alertEventValues];
    const valid = props.value
      .split('+')
      .filter(v => alertEventValues.includes(v));
    return [...new Set(valid)];
  },
  set: value => {
    const unique = [...new Set(value.filter(Boolean).sort())];
    emit('update', unique.length === 0 ? 'none' : unique.join('+'));
  },
});

const setValue = (isChecked, value) => {
  let updated = [...selectedValue.value];
  if (isChecked) updated.push(value);
  else updated = updated.filter(i => i !== value);
  selectedValue.value = updated;
};

const alertDescription = computed(() => {
  const base =
    'PROFILE_SETTINGS.FORM.AUDIO_NOTIFICATIONS_SECTION.ALERT_COMBINATIONS.';
  if (!props.value || props.value === 'none') return base + 'NONE';
  return base + selectedValue.value.join('+').toUpperCase();
});
</script>

<template>
  <div class="flex flex-col gap-2">
    <span
      class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
      >{{ label }}</span
    >
    <div class="flex flex-col gap-2">
      <div
        v-for="option in alertEvents"
        :key="option.value"
        class="flex items-center gap-3 px-4 py-2.5 rounded-xl border border-white/10 bg-white/5 transition-all duration-200 hover:border-[rgba(74,222,128,0.2)] cursor-pointer"
        @click="setValue(!selectedValue.includes(option.value), option.value)"
      >
        <button
          type="button"
          class="w-5 h-5 rounded-md border transition-all duration-200 flex items-center justify-center shrink-0"
          :class="
            selectedValue.includes(option.value)
              ? 'bg-[rgba(74,222,128,0.15)] border-[rgba(74,222,128,0.6)] shadow-[0_0_8px_rgba(74,222,128,0.2)]'
              : 'border-white/20 bg-white/5'
          "
        >
          <fluent-icon
            v-if="selectedValue.includes(option.value)"
            icon="checkmark"
            size="10"
            class="text-[#4ade80]"
          />
        </button>
        <span class="text-sm text-n-slate-9">
          {{
            $t(
              `PROFILE_SETTINGS.FORM.AUDIO_NOTIFICATIONS_SECTION.ALERT_TYPES.${option.label.toUpperCase()}`
            )
          }}
        </span>
      </div>
      <p class="text-xs text-n-slate-9 px-1 mt-1">{{ $t(alertDescription) }}</p>
    </div>
  </div>
</template>
