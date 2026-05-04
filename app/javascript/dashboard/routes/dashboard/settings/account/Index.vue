<script>
import { useVuelidate } from '@vuelidate/core';
import { required } from '@vuelidate/validators';
import { mapGetters } from 'vuex';
import { useAlert } from 'dashboard/composables';
import { useUISettings } from 'dashboard/composables/useUISettings';
import { useConfig } from 'dashboard/composables/useConfig';
import { useAccount } from 'dashboard/composables/useAccount';
import NextButton from 'dashboard/components-next/button/Button.vue';
import AccountId from './components/AccountId.vue';
import AccountDelete from './components/AccountDelete.vue';
import SectionLayout from './components/SectionLayout.vue';

export default {
  components: {
    NextButton,
    AccountId,
    AccountDelete,
    SectionLayout,
  },
  setup() {
    const { updateUISettings, uiSettings } = useUISettings();
    const { enabledLanguages } = useConfig();
    const { accountId } = useAccount();
    const v$ = useVuelidate();
    return { updateUISettings, uiSettings, v$, enabledLanguages, accountId };
  },
  data() {
    return {
      id: '',
      name: '',
      locale: 'en',
      domain: '',
      supportEmail: '',
      features: {},
      activeChatLimitEnabled: false,
      activeChatLimitValue: null,
      queueEnabled: false,
      queueMessage: '',
    };
  },
  validations: {
    name: { required },
    locale: { required },
  },
  computed: {
    ...mapGetters({
      getAccount: 'accounts/getAccount',
      uiFlags: 'accounts/getUIFlags',
      isOnChatwootCloud: 'globalConfig/isOnChatwootCloud',
    }),
    languagesSortedByCode() {
      return [...this.enabledLanguages].sort((l1, l2) =>
        l1.iso_639_1_code.localeCompare(l2.iso_639_1_code)
      );
    },
    isUpdating() {
      return this.uiFlags.isUpdating;
    },
    featureInboundEmailEnabled() {
      return !!this.features?.inbound_emails;
    },
    featureCustomReplyDomainEnabled() {
      return this.featureInboundEmailEnabled && !!this.features.custom_reply_domain;
    },
    featureCustomReplyEmailEnabled() {
      return this.featureInboundEmailEnabled && !!this.features.custom_reply_email;
    },
    currentAccount() {
      return this.getAccount(this.accountId) || {};
    },
  },
  watch: {
    activeChatLimitValue(val) {
      if (val !== null && val < 0) {
        this.activeChatLimitValue = 0;
      }
    },
  },
  mounted() {
    this.initializeAccount();
  },
  methods: {
    async initializeAccount() {
      try {
        const {
          name, locale, id, domain, support_email, features,
          queue_enabled, queue_message, active_chat_limit_enabled, active_chat_limit_value,
        } = this.getAccount(this.accountId);
        const effectiveLocale = this.uiSettings?.locale || locale;
        if (effectiveLocale) this.$root.$i18n.locale = effectiveLocale;
        this.name = name;
        this.locale = locale;
        this.id = id;
        this.domain = domain;
        this.supportEmail = support_email;
        this.features = features;
        this.queueEnabled = queue_enabled;
        this.queueMessage = queue_message;
        this.activeChatLimitEnabled = active_chat_limit_enabled;
        this.activeChatLimitValue = active_chat_limit_value;
      } catch (error) {
        // Ignore error
      }
    },
    handleLimitKeydown(event) {
      const blockedKeys = ['-', 'e', 'E', '+'];
      if (blockedKeys.includes(event.key)) {
        event.preventDefault();
        return;
      }
      if (event.key === 'ArrowDown' && (this.activeChatLimitValue ?? 0) <= 0) {
        event.preventDefault();
      }
    },
    async updateAccount() {
      this.v$.$touch();
      if (this.v$.$invalid) {
        useAlert(this.$t('GENERAL_SETTINGS.FORM.ERROR'));
        return;
      }
      try {
        await this.$store.dispatch('accounts/update', {
          locale: this.locale,
          name: this.name,
          domain: this.domain,
          support_email: this.supportEmail,
          queue_enabled: this.queueEnabled,
          queue_message: this.queueMessage,
          active_chat_limit_enabled: this.activeChatLimitEnabled,
          active_chat_limit_value: this.activeChatLimitValue,
        });
        const updatedLocale = this.uiSettings?.locale || this.locale;
        if (updatedLocale) this.$root.$i18n.locale = updatedLocale;
        this.getAccount(this.id).locale = this.locale;
        useAlert(this.$t('GENERAL_SETTINGS.UPDATE.SUCCESS'));
      } catch (error) {
        useAlert(this.$t('GENERAL_SETTINGS.UPDATE.ERROR'));
      }
    },
  },
};
</script>

<template>
  <div class="flex flex-col w-full max-w-2xl ltr:mr-auto rtl:ml-auto">
    <div class="pb-6 border-b border-white/10 mb-2">
      <p class="text-xs font-semibold tracking-[0.2em] text-[#4ade80] uppercase mb-1">
        {{ $t('GENERAL_SETTINGS.FORM.WORKSPACE_LABEL') }}
      </p>
      <h2 class="text-3xl font-black tracking-wide text-white uppercase">
        {{ $t('GENERAL_SETTINGS.TITLE') }}
      </h2>
    </div>

    <AccountId />

    <div class="flex-grow flex-shrink min-w-0">
      <SectionLayout
        :title="$t('GENERAL_SETTINGS.FORM.GENERAL_SECTION.TITLE')"
        :description="$t('GENERAL_SETTINGS.FORM.GENERAL_SECTION.NOTE')"
        section-number="01">
        <form
          v-if="!uiFlags.isFetchingItem"
          id="general-settings-form"
          class="flex flex-col gap-4"
          @submit.prevent="updateAccount">
          <div class="grid grid-cols-2 gap-3">
            <div
              class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)] hover:shadow-[0_0_12px_rgba(74,222,128,0.15)] focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]"
              :class="{ 'border-red-500/50': v$.name.$error }">
              <span class="text-[10px] font-semibold tracking-[0.15em] text-[#4ade80] uppercase">
                {{ $t('GENERAL_SETTINGS.FORM.NAME.LABEL') }}
              </span>
              <input
                v-model="name"
                type="text"
                :placeholder="$t('GENERAL_SETTINGS.FORM.NAME.PLACEHOLDER')"
                class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0"
                @blur="v$.name.$touch" />
            </div>

            <div
              class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)] hover:shadow-[0_0_12px_rgba(74,222,128,0.15)] focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]"
              :class="{ 'border-red-500/50': v$.locale.$error }">
              <span class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase">
                {{ $t('GENERAL_SETTINGS.FORM.LANGUAGE.LABEL') }}
              </span>
              <select
                v-model="locale"
                class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 p-0 appearance-none cursor-pointer">
                <option
                  v-for="lang in languagesSortedByCode"
                  :key="lang.iso_639_1_code"
                  :value="lang.iso_639_1_code"
                  class="bg-n-solid-3">
                  {{ lang.name }}
                </option>
              </select>
            </div>

            <div
              v-if="featureCustomReplyDomainEnabled"
              class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 col-span-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)] hover:shadow-[0_0_12px_rgba(74,222,128,0.15)] focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]">
              <span class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase">
                {{ $t('GENERAL_SETTINGS.FORM.DOMAIN.LABEL') }}
              </span>
              <input
                v-model="domain"
                type="text"
                :placeholder="$t('GENERAL_SETTINGS.FORM.DOMAIN.PLACEHOLDER')"
                class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0" />
              <span v-if="featureInboundEmailEnabled" class="text-[10px] text-n-slate-9 mt-0.5">
                {{ $t('GENERAL_SETTINGS.FORM.FEATURES.INBOUND_EMAIL_ENABLED') }}
              </span>
            </div>

            <!-- <div
              v-if="featureCustomReplyEmailEnabled"
              class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 col-span-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)] hover:shadow-[0_0_12px_rgba(74,222,128,0.15)] focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]">
              <span class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase">
                {{ $t('GENERAL_SETTINGS.FORM.SUPPORT_EMAIL.LABEL') }}
              </span>
              <input
                v-model="supportEmail"
                type="text"
                :placeholder="$t('GENERAL_SETTINGS.FORM.SUPPORT_EMAIL.PLACEHOLDER')"
                class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0" />
            </div> -->
          </div>
        </form>
        <woot-loading-state v-if="uiFlags.isFetchingItem" />
      </SectionLayout>

      <SectionLayout
        :title="$t('GENERAL_SETTINGS.FORM.LIMIT_ENABLED')"
        :description="$t('GENERAL_SETTINGS.FORM.AGENT_LIMIT.DESCRIPTION')"
        with-border
        section-number="02">
        <div class="flex flex-col gap-3">
          <div class="grid grid-cols-2 gap-3">
            <button
              type="button"
              class="flex flex-col gap-1 rounded-xl border px-4 py-3 text-left transition-all duration-200 cursor-pointer"
              :class="!activeChatLimitEnabled
                ? 'border-[rgba(74,222,128,0.5)] bg-[rgba(74,222,128,0.05)] shadow-[0_0_16px_rgba(74,222,128,0.15)]'
                : 'border-white/10 bg-white/5 hover:border-white/20'"
              @click="activeChatLimitEnabled = false">
              <span
                class="text-[10px] font-semibold tracking-[0.15em] uppercase"
                :class="!activeChatLimitEnabled ? 'text-[#4ade80]' : 'text-n-slate-10'">
                {{ $t('GENERAL_SETTINGS.FORM.AGENT_LIMIT.UNLIMITED_LABEL') }}
              </span>
              <span class="text-xs text-n-slate-9">
                {{ $t('GENERAL_SETTINGS.FORM.AGENT_LIMIT.UNLIMITED_NOTE') }}
              </span>
            </button>
            <button
              type="button"
              class="flex flex-col gap-1 rounded-xl border px-4 py-3 text-left transition-all duration-200 cursor-pointer"
              :class="activeChatLimitEnabled
                ? 'border-[rgba(74,222,128,0.5)] bg-[rgba(74,222,128,0.05)] shadow-[0_0_16px_rgba(74,222,128,0.15)]'
                : 'border-white/10 bg-white/5 hover:border-white/20'"
              @click="activeChatLimitEnabled = true">
              <span
                class="text-[10px] font-semibold tracking-[0.15em] uppercase"
                :class="activeChatLimitEnabled ? 'text-[#4ade80]' : 'text-n-slate-10'">
                {{ $t('GENERAL_SETTINGS.FORM.AGENT_LIMIT.CUSTOM_LABEL') }}
              </span>
              <span class="text-xs text-n-slate-9">
                {{ $t('GENERAL_SETTINGS.FORM.AGENT_LIMIT.CUSTOM_NOTE') }}
              </span>
            </button>
          </div>

          <div
            v-if="activeChatLimitEnabled"
            class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 transition-all duration-200 focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]">
            <span class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase">
              {{ $t('GENERAL_SETTINGS.FORM.LIMIT_VALUE') }}
            </span>
            <input
              v-model.number="activeChatLimitValue"
              type="number"
              :min="0"
              :placeholder="$t('GENERAL_SETTINGS.FORM.LIMIT_VALUE')"
              class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0"
              @keydown="handleLimitKeydown" />
          </div>
        </div>
      </SectionLayout>

      <SectionLayout
        :title="$t('GENERAL_SETTINGS.FORM.QUEUE_ENABLED')"
        :description="$t('GENERAL_SETTINGS.FORM.QUEUE_MODE.DESCRIPTION')"
        with-border
        section-number="03">
        <div class="flex flex-col gap-3">
          <div class="grid grid-cols-2 gap-3">
            <button
              type="button"
              class="flex flex-col gap-1 rounded-xl border px-4 py-3 text-left transition-all duration-200 cursor-pointer"
              :class="!queueEnabled
                ? 'border-[rgba(74,222,128,0.5)] bg-[rgba(74,222,128,0.05)] shadow-[0_0_16px_rgba(74,222,128,0.15)]'
                : 'border-white/10 bg-white/5 hover:border-white/20'"
              @click="queueEnabled = false">
              <span
                class="text-[10px] font-semibold tracking-[0.15em] uppercase"
                :class="!queueEnabled ? 'text-[#4ade80]' : 'text-n-slate-10'">
                {{ $t('GENERAL_SETTINGS.FORM.QUEUE_MODE.DISABLED_LABEL') }}
              </span>
              <span class="text-xs text-n-slate-9">
                {{ $t('GENERAL_SETTINGS.FORM.QUEUE_MODE.DISABLED_NOTE') }}
              </span>
            </button>
            <button
              type="button"
              class="flex flex-col gap-1 rounded-xl border px-4 py-3 text-left transition-all duration-200 cursor-pointer"
              :class="queueEnabled
                ? 'border-[rgba(74,222,128,0.5)] bg-[rgba(74,222,128,0.05)] shadow-[0_0_16px_rgba(74,222,128,0.15)]'
                : 'border-white/10 bg-white/5 hover:border-white/20'"
              @click="queueEnabled = true">
              <span
                class="text-[10px] font-semibold tracking-[0.15em] uppercase"
                :class="queueEnabled ? 'text-[#4ade80]' : 'text-n-slate-10'">
                {{ $t('GENERAL_SETTINGS.FORM.QUEUE_MODE.ENABLED_LABEL') }}
              </span>
              <span class="text-xs text-n-slate-9">
                {{ $t('GENERAL_SETTINGS.FORM.QUEUE_MODE.ENABLED_NOTE') }}
              </span>
            </button>
          </div>

          <div
            v-if="queueEnabled"
            class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-3 transition-all duration-200 focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]">
            <span class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase">
              {{ $t('GENERAL_SETTINGS.FORM.QUEUE_MESSAGE.LABEL') }}
            </span>
            <textarea
              v-model="queueMessage"
              :placeholder="$t('GENERAL_SETTINGS.FORM.QUEUE_MESSAGE.PLACEHOLDER')"
              rows="3"
              class="bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0 resize-none mt-1" />
          </div>
        </div>
      </SectionLayout>

      <div v-if="!uiFlags.isFetchingItem && isOnChatwootCloud">
        <AccountDelete />
      </div>

      <div class="flex justify-end pt-4 mt-2 border-t border-white/10">
        <NextButton
          teal
          :is-loading="isUpdating"
          type="submit"
          form="general-settings-form"
          @click="updateAccount">
          {{ $t('GENERAL_SETTINGS.SUBMIT') }}
        </NextButton>
      </div>
    </div>
  </div>
</template>
