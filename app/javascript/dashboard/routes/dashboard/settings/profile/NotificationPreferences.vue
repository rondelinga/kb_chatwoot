<script>
import { mapGetters } from 'vuex';
import { useAlert } from 'dashboard/composables';
import {
  hasPushPermissions,
  requestPushPermissions,
  verifyServiceWorkerExistence,
} from 'dashboard/helper/pushHelper.js';
import { FEATURE_FLAGS } from 'dashboard/featureFlags';
import ToggleSwitch from 'dashboard/components-next/switch/Switch.vue';
import { NOTIFICATION_TYPES } from './constants';

export default {
  components: { ToggleSwitch },
  data() {
    return {
      selectedEmailFlags: [],
      selectedPushFlags: [],
      hasEnabledPushPermissions: false,
      notificationTypes: NOTIFICATION_TYPES,
    };
  },
  computed: {
    ...mapGetters({
      accountId: 'getCurrentAccountId',
      emailFlags: 'userNotificationSettings/getSelectedEmailFlags',
      pushFlags: 'userNotificationSettings/getSelectedPushFlags',
      isFeatureEnabledonAccount: 'accounts/isFeatureEnabledonAccount',
    }),
    isSLAEnabled() {
      return this.isFeatureEnabledonAccount(this.accountId, FEATURE_FLAGS.SLA);
    },
    filteredNotificationTypes() {
      return this.notificationTypes.filter(n =>
        this.isSLAEnabled
          ? true
          : !['sla_missed_first_response', 'sla_missed_next_response', 'sla_missed_resolution'].includes(n.value)
      );
    },
  },
  watch: {
    emailFlags(value) {
      this.selectedEmailFlags = value;
    },
    pushFlags(value) {
      this.selectedPushFlags = value;
    },
  },
  mounted() {
    if (hasPushPermissions()) this.getPushSubscription();
    this.$store.dispatch('userNotificationSettings/get');
  },
  methods: {
    // FIX: no-dynamic-keys — обёртка для динамических ключей i18n
    translateLabel(label) {
      return this.$t(label);
    },
    checkFlagStatus(type, flagType) {
      const flags = type === 'email' ? this.selectedEmailFlags : this.selectedPushFlags;
      return flags.includes(`${type}_${flagType}`);
    },
    onRequestPermissions(value) {
      if (value) {
        requestPushPermissions({
          onSuccess: () => {
            this.hasEnabledPushPermissions = true;
          },
        });
      } else {
        this.disablePushPermissions();
      }
    },
    disablePushPermissions() {
      verifyServiceWorkerExistence(registration =>
        registration.pushManager
          .getSubscription()
          // FIX: no-unused-expressions — явный вызов через then
          .then(sub => {
            if (sub) {
              return sub.unsubscribe();
            }
            return Promise.resolve();
          })
          .finally(() => {
            this.hasEnabledPushPermissions = false;
          })
          // FIX: no-console — пустой обработчик вместо console.log
          .catch(() => {})
      );
    },
    getPushSubscription() {
      verifyServiceWorkerExistence(registration =>
        registration.pushManager
          .getSubscription()
          .then(sub => {
            this.hasEnabledPushPermissions = !!sub;
          })
          // FIX: no-console — убран console.log
          .catch(() => {})
      );
    },
    async updateNotificationSettings() {
      try {
        await this.$store.dispatch('userNotificationSettings/update', {
          selectedEmailFlags: this.selectedEmailFlags,
          selectedPushFlags: this.selectedPushFlags,
        });
        useAlert(this.$t('PROFILE_SETTINGS.FORM.API.UPDATE_SUCCESS'));
      } catch {
        useAlert(this.$t('PROFILE_SETTINGS.FORM.API.UPDATE_ERROR'));
      }
    },
    toggleInput(selected, current) {
      return selected.includes(current)
        ? selected.filter(f => f !== current)
        : [...selected, current];
    },
    handleEmailInput(id) {
      this.selectedEmailFlags = this.toggleInput(this.selectedEmailFlags, id);
      this.updateNotificationSettings();
    },
    handlePushInput(id) {
      this.selectedPushFlags = this.toggleInput(this.selectedPushFlags, id);
      this.updateNotificationSettings();
    },
    handleInput(type, id) {
      if (type === 'email') {
        this.handleEmailInput(id);
      } else {
        this.handlePushInput(id);
      }
    },
  },
};
</script>

<template>
  <div class="flex flex-col gap-3">
    <!-- Desktop -->
    <div class="hidden sm:flex flex-col gap-1">
      <!-- Header -->
      <div class="grid grid-cols-12 gap-4 px-4 py-2">
        <div class="col-span-7">
          <span class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase">
            {{ $t('PROFILE_SETTINGS.FORM.NOTIFICATIONS.TYPE_TITLE') }}
          </span>
        </div>
        <div class="col-span-2 flex justify-center">
          <span class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase">
            {{ $t('PROFILE_SETTINGS.FORM.NOTIFICATIONS.EMAIL') }}
          </span>
        </div>
        <div class="col-span-3 flex justify-center">
          <span class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase">
            {{ $t('PROFILE_SETTINGS.FORM.NOTIFICATIONS.PUSH') }}
          </span>
        </div>
      </div>

      <!-- Rows -->
      <div
        v-for="(notification, index) in filteredNotificationTypes"
        :key="index"
        class="grid grid-cols-12 gap-4 px-4 py-2.5 rounded-xl border border-white/10 bg-white/5 transition-all duration-200 hover:border-[rgba(74,222,128,0.2)]"
      >
        <div class="col-span-7 flex items-center">
          <!-- FIX: no-dynamic-keys — используем метод translateLabel -->
          <span class="text-sm text-n-slate-9">{{ translateLabel(notification.label) }}</span>
        </div>
        <div
          v-for="type in ['email', 'push']"
          :key="type"
          class="flex items-center justify-center"
          :class="type === 'push' ? 'col-span-3' : 'col-span-2'"
        >
          <button
            type="button"
            class="w-5 h-5 rounded-md border transition-all duration-200 flex items-center justify-center shrink-0"
            :class="checkFlagStatus(type, notification.value)
              ? 'bg-[rgba(74,222,128,0.15)] border-[rgba(74,222,128,0.6)] shadow-[0_0_8px_rgba(74,222,128,0.2)]'
              : 'border-white/20 bg-white/5 hover:border-[rgba(74,222,128,0.4)]'"
            @click="handleInput(type, `${type}_${notification.value}`)"
          >
            <fluent-icon
              v-if="checkFlagStatus(type, notification.value)"
              icon="checkmark"
              size="10"
              class="text-[#4ade80]"
            />
          </button>
        </div>
      </div>
    </div>

    <!-- Mobile -->
    <div class="flex flex-col gap-4 sm:hidden">
      <span class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase">
        {{ $t('PROFILE_SETTINGS.FORM.EMAIL_NOTIFICATIONS_SECTION.TITLE') }}
      </span>
      <div class="flex flex-col gap-2">
        <div
          v-for="(notification, index) in filteredNotificationTypes"
          :key="index"
          class="flex flex-row items-center gap-3 px-4 py-2.5 rounded-xl border border-white/10 bg-white/5 transition-all duration-200 hover:border-[rgba(74,222,128,0.2)]"
          @click="handleEmailInput(`email_${notification.value}`)"
        >
          <button
            type="button"
            class="w-5 h-5 rounded-md border transition-all duration-200 flex items-center justify-center shrink-0"
            :class="checkFlagStatus('email', notification.value)
              ? 'bg-[rgba(74,222,128,0.15)] border-[rgba(74,222,128,0.6)]'
              : 'border-white/20 bg-white/5'"
          >
            <fluent-icon
              v-if="checkFlagStatus('email', notification.value)"
              icon="checkmark"
              size="10"
              class="text-[#4ade80]"
            />
          </button>
          <!-- FIX: no-dynamic-keys — используем метод translateLabel -->
          <span class="text-sm text-n-slate-9">{{ translateLabel(notification.label) }}</span>
        </div>
      </div>

      <span class="text-[10px] font-semibold tracking-[0.15em] text-n-slate-10 uppercase mt-2">
        {{ $t('PROFILE_SETTINGS.FORM.PUSH_NOTIFICATIONS_SECTION.TITLE') }}
      </span>
      <div class="flex flex-col gap-2">
        <div
          v-for="(notification, index) in filteredNotificationTypes"
          :key="index"
          class="flex flex-row items-center gap-3 px-4 py-2.5 rounded-xl border border-white/10 bg-white/5 transition-all duration-200 hover:border-[rgba(74,222,128,0.2)]"
          @click="handlePushInput(`push_${notification.value}`)"
        >
          <button
            type="button"
            class="w-5 h-5 rounded-md border transition-all duration-200 flex items-center justify-center shrink-0"
            :class="checkFlagStatus('push', notification.value)
              ? 'bg-[rgba(74,222,128,0.15)] border-[rgba(74,222,128,0.6)]'
              : 'border-white/20 bg-white/5'"
          >
            <fluent-icon
              v-if="checkFlagStatus('push', notification.value)"
              icon="checkmark"
              size="10"
              class="text-[#4ade80]"
            />
          </button>
          <!-- FIX: no-dynamic-keys — используем метод translateLabel -->
          <span class="text-sm text-n-slate-9">{{ translateLabel(notification.label) }}</span>
        </div>
      </div>
    </div>

    <!-- Browser permission toggle -->
    <div class="flex items-center justify-between w-full gap-2 px-4 py-3 rounded-xl border border-white/10 bg-white/5 transition-all duration-200 hover:border-[rgba(74,222,128,0.2)] mt-1">
      <div class="flex flex-row items-center gap-2">
        <fluent-icon icon="alert" class="shrink-0 text-n-slate-10" size="16" />
        <span class="text-sm text-n-slate-9">
          {{ $t('PROFILE_SETTINGS.FORM.NOTIFICATIONS.BROWSER_PERMISSION') }}
        </span>
      </div>
      <ToggleSwitch v-model="hasEnabledPushPermissions" @change="onRequestPermissions" />
    </div>
  </div>
</template>
