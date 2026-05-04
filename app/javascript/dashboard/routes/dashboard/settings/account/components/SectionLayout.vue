<script setup>
import { useI18n } from 'vue-i18n';

defineProps({
  title: { type: String, required: true },
  description: { type: String, required: true },
  withBorder: { type: Boolean, default: false },
  hideContent: { type: Boolean, default: false },
  beta: { type: Boolean, default: false },
  sectionNumber: { type: String, default: '' },
});
const { t } = useI18n();
</script>

<template>
  <section
    class="flex flex-col gap-4 [interpolate-size:allow-keywords]"
    :class="{
      'pt-8 border-t border-white/10': withBorder,
      'pt-6': !withBorder,
      'pb-6': !hideContent,
    }"
  >
    <header
      v-if="
        title ||
        $slots.title ||
        description ||
        $slots.description ||
        $slots.headerActions
      "
      class="flex flex-col gap-1"
    >
      <div class="flex items-center justify-between">
        <div class="flex items-center gap-2">
          <span
            v-if="sectionNumber"
            class="text-xs font-bold text-[#4ade80] tracking-widest"
            >{{ sectionNumber }}</span
          >
          <h4
            class="text-xs font-semibold tracking-[0.18em] text-n-slate-10 uppercase flex items-center gap-2"
          >
            <slot name="title">{{ title }}</slot>
            <div
              v-if="beta"
              v-tooltip.top="t('GENERAL.BETA_DESCRIPTION')"
              class="text-[10px] uppercase text-[#4ade80] border border-[#4ade80]/30 leading-none rounded-lg px-1 py-0.5"
            >
              {{ t('GENERAL.BETA') }}
            </div>
          </h4>
        </div>
        <slot name="headerActions" />
      </div>
      <p
        v-if="description || $slots.description"
        class="text-xs text-n-slate-9 max-w-xl"
      >
        <slot name="description">{{ description }}</slot>
      </p>
    </header>
    <div
      class="transition-[height] duration-300 ease-in-out"
      :class="{ 'overflow-hidden h-0': hideContent, 'h-auto': !hideContent }"
    >
      <slot />
    </div>
  </section>
</template>
