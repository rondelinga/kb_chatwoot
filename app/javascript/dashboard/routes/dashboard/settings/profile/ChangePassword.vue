<script>
import { useVuelidate } from '@vuelidate/core';
import { required, minLength } from '@vuelidate/validators';
import { useAlert } from 'dashboard/composables';
import { parseAPIErrorResponse } from 'dashboard/store/utils/api';
import NextButton from 'dashboard/components-next/button/Button.vue';

export default {
  components: { NextButton },
  setup() {
    return { v$: useVuelidate() };
  },
  data() {
    return {
      currentPassword: '',
      password: '',
      passwordConfirmation: '',
    };
  },
  validations: {
    currentPassword: { required },
    password: { minLength: minLength(6) },
    passwordConfirmation: {
      minLength: minLength(6),
      isEqPassword(value) {
        return value === this.password;
      },
    },
  },
  computed: {
    isButtonDisabled() {
      return (
        !this.currentPassword ||
        !this.passwordConfirmation ||
        !this.v$.passwordConfirmation.isEqPassword
      );
    },
  },
  methods: {
    async changePassword() {
      this.v$.$touch();
      if (this.v$.$invalid) {
        useAlert(this.$t('PROFILE_SETTINGS.FORM.ERROR'));
        return;
      }
      let alertMessage = this.$t('PROFILE_SETTINGS.PASSWORD_UPDATE_SUCCESS');
      try {
        await this.$store.dispatch('updatePassword', {
          password: this.password,
          passwordConfirmation: this.passwordConfirmation,
          currentPassword: this.currentPassword,
        });
      } catch (error) {
        alertMessage =
          parseAPIErrorResponse(error) ||
          this.$t('RESET_PASSWORD.API.ERROR_MESSAGE');
      } finally {
        useAlert(alertMessage);
      }
    },
  },
};
</script>

<template>
  <form class="flex flex-col gap-3" @submit.prevent="changePassword">
    <div
      class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)] focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]"
      :class="{ 'border-red-500/50': v$.currentPassword.$error }"
    >
      <span
        class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
      >
        {{ $t('PROFILE_SETTINGS.FORM.CURRENT_PASSWORD.LABEL') }}
      </span>
      <input
        v-model="currentPassword"
        type="password"
        :placeholder="$t('PROFILE_SETTINGS.FORM.CURRENT_PASSWORD.PLACEHOLDER')"
        class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0"
        @blur="v$.currentPassword.$touch"
      />
    </div>

    <div class="grid grid-cols-2 gap-3">
      <div
        class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)] focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]"
        :class="{ 'border-red-500/50': v$.password.$error }"
      >
        <span
          class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
        >
          {{ $t('PROFILE_SETTINGS.FORM.PASSWORD.LABEL') }}
        </span>
        <input
          v-model="password"
          type="password"
          :placeholder="$t('PROFILE_SETTINGS.FORM.PASSWORD.PLACEHOLDER')"
          class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0"
          @blur="v$.password.$touch"
        />
      </div>

      <div
        class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)] focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]"
        :class="{ 'border-red-500/50': v$.passwordConfirmation.$error }"
      >
        <span
          class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
        >
          {{ $t('PROFILE_SETTINGS.FORM.PASSWORD_CONFIRMATION.LABEL') }}
        </span>
        <input
          v-model="passwordConfirmation"
          type="password"
          :placeholder="
            $t('PROFILE_SETTINGS.FORM.PASSWORD_CONFIRMATION.PLACEHOLDER')
          "
          class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0"
          @blur="v$.passwordConfirmation.$touch"
        />
      </div>
    </div>

    <div class="flex justify-end pt-2 border-t border-white/10">
      <NextButton
        teal
        type="submit"
        :label="$t('PROFILE_SETTINGS.FORM.PASSWORD_SECTION.BTN_TEXT')"
        :disabled="isButtonDisabled"
      />
    </div>
  </form>
</template>
