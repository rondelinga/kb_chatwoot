<script setup>
import { computed } from 'vue';
import Icon from 'next/icon/Icon.vue';
import * as Sentry from '@sentry/vue';

const props = defineProps({
  value: { type: String, required: true },
  label: { type: String, default: '' },
});

const emit = defineEmits(['change']);

const alertTones = [
  { value: 'ping', label: 'Beep' },
  { value: 'bell', label: 'Bell' },
  { value: 'ding', label: 'Classic' },
  { value: 'chime', label: 'Chime' },
  { value: 'magic', label: 'Soft' },
];

const selectedValue = computed({
  get: () => props.value,
  set: value => emit('change', value),
});

const audio = new Audio();
const playAudio = async () => {
  try {
    audio.src = `/audio/dashboard/${selectedValue.value}.mp3`;
    await audio.play();
  } catch (error) {
    Sentry.captureException(error);
  }
};
</script>

<template>
  <div class="flex flex-col gap-2">
    <span
      class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
      >{{ label }}</span
    >
    <div class="flex items-center gap-2">
      <div class="flex-1 flex flex-wrap gap-2">
        <button
          v-for="tone in alertTones"
          :key="tone.value"
          type="button"
          class="px-3 py-1.5 rounded-lg border text-sm transition-all duration-200"
          :class="
            selectedValue === tone.value
              ? 'border-[rgba(74,222,128,0.6)] bg-[rgba(74,222,128,0.1)] text-[#4ade80] shadow-[0_0_8px_rgba(74,222,128,0.15)]'
              : 'border-white/10 bg-white/5 text-n-slate-9 hover:border-[rgba(74,222,128,0.4)]'
          "
          @click="selectedValue = tone.value"
        >
          {{ tone.label }}
        </button>
      </div>
      <button
        v-tooltip.top="
          $t('PROFILE_SETTINGS.FORM.AUDIO_NOTIFICATIONS_SECTION.PLAY')
        "
        type="button"
        class="flex items-center justify-center w-8 h-8 rounded-lg border border-white/10 bg-white/5 text-n-slate-9 hover:border-[rgba(74,222,128,0.4)] hover:text-[#4ade80] transition-all duration-200 shrink-0"
        @click="playAudio"
      >
        <Icon icon="i-lucide-volume-2" class="w-4 h-4" />
      </button>
    </div>
  </div>
</template>
