<script setup>
import { computed } from 'vue';
import { useAccount } from 'dashboard/composables/useAccount';
import SectionLayout from './SectionLayout.vue';

const { currentAccount } = useAccount();
const getAccountId = computed(() => currentAccount.value.id.toString());
</script>

<template>
  <SectionLayout
    title="Workspace ID"
    description="Required for API-based integrations and third-party connections"
    section-number="—"
  >
    <div
      class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 w-fit min-w-[120px]"
    >
      <span
        class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
        >Account ID</span
      >
      <span class="text-sm font-mono text-[#4ade80]">{{ getAccountId }}</span>
    </div>
  </SectionLayout>
</template>
