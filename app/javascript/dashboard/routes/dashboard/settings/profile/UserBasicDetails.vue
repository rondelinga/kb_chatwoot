<script>
import { useAlert } from 'dashboard/composables';
import NextButton from 'dashboard/components-next/button/Button.vue';
import { useVuelidate } from '@vuelidate/core';
import { required, minLength, email } from '@vuelidate/validators';

export default {
  components: { NextButton },
  props: {
    name: { type: String, default: '' },
    email: { type: String, default: '' },
    displayName: { type: String, default: '' },
    emailEnabled: { type: Boolean, default: false },
  },
  emits: ['updateUser'],
  setup() {
    return { v$: useVuelidate() };
  },
  data() {
    return {
      userName: this.name,
      userDisplayName: this.displayName,
      userEmail: this.email,
    };
  },
  validations: {
    userName: { required, minLength: minLength(1) },
    userDisplayName: {},
    userEmail: { required, email },
  },
  watch: {
    name: {
      handler(v) {
        this.userName = v;
      },
      immediate: true,
    },
    displayName: {
      handler(v) {
        this.userDisplayName = v;
      },
      immediate: true,
    },
    email: {
      handler(v) {
        this.userEmail = v;
      },
      immediate: true,
    },
  },
  methods: {
    async updateUser() {
      this.v$.$touch();
      if (this.v$.$invalid) {
        useAlert(this.$t('PROFILE_SETTINGS.FORM.ERROR'));
        return;
      }
      this.$emit('updateUser', {
        name: this.userName,
        displayName: this.userDisplayName,
        email: this.userEmail,
      });
    },
  },
};
</script>

<template>
  <form class="flex flex-col gap-3" @submit.prevent="updateUser">
    <div class="grid grid-cols-2 gap-3">
      <div
        class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)] hover:shadow-[0_0_12px_rgba(74,222,128,0.15)] focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]"
        :class="{ 'border-red-500/50': v$.userName.$error }"
      >
        <span
          class="text-[10px] font-semibold tracking-[0.15em] text-[#4ade80] uppercase"
        >
          {{ $t('PROFILE_SETTINGS.FORM.NAME.LABEL') }}
        </span>
        <input
          v-model="userName"
          type="text"
          :placeholder="$t('PROFILE_SETTINGS.FORM.NAME.PLACEHOLDER')"
          class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0"
          @blur="v$.userName.$touch"
        />
      </div>

      <div
        class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)] hover:shadow-[0_0_12px_rgba(74,222,128,0.15)] focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]"
        :class="{ 'border-red-500/50': v$.userDisplayName.$error }"
      >
        <span
          class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
        >
          {{ $t('PROFILE_SETTINGS.FORM.DISPLAY_NAME.LABEL') }}
        </span>
        <input
          v-model="userDisplayName"
          type="text"
          :placeholder="$t('PROFILE_SETTINGS.FORM.DISPLAY_NAME.PLACEHOLDER')"
          class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0"
          @blur="v$.userDisplayName.$touch"
        />
      </div>

      <div
        v-if="emailEnabled"
        class="flex flex-col gap-1 rounded-xl border border-white/10 bg-white/5 px-4 pt-1.5 pb-2 col-span-2 transition-all duration-200 hover:border-[rgba(74,222,128,0.4)] hover:shadow-[0_0_12px_rgba(74,222,128,0.15)] focus-within:border-[rgba(74,222,128,0.5)] focus-within:shadow-[0_0_16px_rgba(74,222,128,0.2)]"
        :class="{ 'border-red-500/50': v$.userEmail.$error }"
      >
        <span
          class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase"
        >
          {{ $t('PROFILE_SETTINGS.FORM.EMAIL.LABEL') }}
        </span>
        <input
          v-model="userEmail"
          type="email"
          :placeholder="$t('PROFILE_SETTINGS.FORM.EMAIL.PLACEHOLDER')"
          class="h-6 bg-transparent border-0 outline-none text-sm text-n-slate-9 placeholder:text-n-slate-8 p-0"
          @blur="v$.userEmail.$touch"
        />
      </div>
    </div>

    <div class="flex justify-end pt-2 border-t border-white/10">
      <NextButton teal type="submit" :label="$t('PROFILE_SETTINGS.BTN_TEXT')" />
    </div>
  </form>
</template>
