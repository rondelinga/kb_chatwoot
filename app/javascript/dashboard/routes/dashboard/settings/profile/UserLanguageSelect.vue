<script setup>
import { computed } from 'vue';
import { useI18n } from 'vue-i18n';
import { useAlert } from 'dashboard/composables';
import { useConfig } from 'dashboard/composables/useConfig';
import { useAccount } from 'dashboard/composables/useAccount';
import { useUISettings } from 'dashboard/composables/useUISettings';

defineProps({
  label: { type: String, default: '' },
  description: { type: String, default: '' },
});

const { t, locale } = useI18n();
const { updateUISettings, uiSettings } = useUISettings();
const { enabledLanguages } = useConfig();
const { currentAccount } = useAccount();

const currentLanguage = computed(() => uiSettings.value?.locale ?? '');

const languageOptions = computed(() => [
  {
    name: t(
      'PROFILE_SETTINGS.FORM.INTERFACE_SECTION.LANGUAGE.USE_ACCOUNT_DEFAULT'
    ),
    iso_639_1_code: '',
  },
  ...(enabledLanguages ?? []),
]);

const updateLanguage = async languageCode => {
  try {
    if (!languageCode) {
      await updateUISettings({ locale: null });
      locale.value = currentAccount.value.locale;
      useAlert(
        t('PROFILE_SETTINGS.FORM.INTERFACE_SECTION.LANGUAGE.UPDATE_SUCCESS')
      );
      return;
    }
    const valid = (enabledLanguages || []).some(
      l => l.iso_639_1_code === languageCode
    );
    if (!valid) throw new Error(`Invalid language code: ${languageCode}`);
    await updateUISettings({ locale: languageCode });
    locale.value = languageCode;
    useAlert(
      t('PROFILE_SETTINGS.FORM.INTERFACE_SECTION.LANGUAGE.UPDATE_SUCCESS')
    );
  } catch {
    useAlert(
      t('PROFILE_SETTINGS.FORM.INTERFACE_SECTION.LANGUAGE.UPDATE_ERROR')
    );
  }
};

const selectedValue = computed({
  get: () => currentLanguage.value,
  set: value => updateLanguage(value),
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
        v-for="option in languageOptions"
        :key="option.iso_639_1_code || 'default'"
        :value="option.iso_639_1_code"
        class="bg-n-solid-3"
      >
        {{ option.name }}
      </option>
    </select>
  </div>
</template>
