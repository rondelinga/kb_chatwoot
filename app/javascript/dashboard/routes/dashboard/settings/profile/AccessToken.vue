<script setup>
import { ref, computed } from 'vue';
import NextButton from 'dashboard/components-next/button/Button.vue';
import ConfirmButton from 'dashboard/components-next/button/ConfirmButton.vue';

const props = defineProps({
  value: { type: String, default: '' },
  showResetButton: { type: Boolean, default: true },
});

const emit = defineEmits(['onCopy', 'onReset']);

const isVisible = ref(false);

const displayValue = computed(() =>
  isVisible.value ? props.value : '•'.repeat(40)
);

const onClick = () => emit('onCopy', props.value);
const onReset = () => emit('onReset');
</script>

<template>
  <div class="flex flex-col gap-3">
    <div class="flex flex-row gap-3 items-center">
      <div
        class="flex flex-1 flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)]"
      >
        <span
          class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
          >Token</span
        >
        <span class="text-sm font-mono text-n-slate-9 truncate select-all">{{
          displayValue
        }}</span>
      </div>
      <div class="flex flex-row gap-2 shrink-0">
        <button
          type="button"
          class="flex items-center justify-center w-8 h-8 rounded-lg border border-white/10 bg-white/5 text-n-slate-9 hover:border-[rgba(74,222,128,0.4)] hover:text-[#4ade80] transition-all duration-200"
          @click="isVisible = !isVisible"
        >
          <fluent-icon :icon="isVisible ? 'eye-hide' : 'eye-show'" :size="15" />
        </button>
        <NextButton
          :label="$t('PROFILE_SETTINGS.FORM.ACCESS_TOKEN.COPY')"
          slate
          outline
          type="button"
          icon="i-lucide-copy"
          class="rounded-xl"
          @click="onClick"
        />
        <ConfirmButton
          v-if="showResetButton"
          :label="$t('PROFILE_SETTINGS.FORM.ACCESS_TOKEN.RESET')"
          :confirm-label="
            $t('PROFILE_SETTINGS.FORM.ACCESS_TOKEN.CONFIRM_RESET')
          "
          :confirm-hint="$t('PROFILE_SETTINGS.FORM.ACCESS_TOKEN.CONFIRM_HINT')"
          color="slate"
          confirm-color="ruby"
          variant="outline"
          icon="i-lucide-key-round"
          class="rounded-xl"
          @click="onReset"
        />
      </div>
    </div>
  </div>
</template>
