<script setup>
import Avatar from 'dashboard/components-next/avatar/Avatar.vue';

defineProps({
  src: { type: String, default: '' },
  name: { type: String, default: '' },
});

const emit = defineEmits(['change', 'delete']);
</script>

<template>
  <div
    class="flex items-center gap-4 px-4 py-3 rounded-xl border border-white/10 bg-white/5"
  >
    <Avatar
      :src="src || ''"
      :name="name || ''"
      :size="64"
      allow-upload
      @upload="e => emit('change', e)"
      @delete="() => emit('delete')"
    />
    <div class="flex flex-col gap-0.5">
      <span
        class="text-[10px] font-semibold tracking-[0.15em] text-[#4ade80] uppercase"
      >
        {{ $t('PROFILE_SETTINGS.FORM.PICTURE') }}
      </span>
      <p class="text-xs text-n-slate-9">
        {{ $t('PROFILE_SETTINGS.FORM.AVATAR') }}
      </p>
    </div>
  </div>
</template>
