<script>
import { mapGetters } from 'vuex';
import { useAlert } from 'dashboard/composables';
import { useUISettings } from 'dashboard/composables/useUISettings';
import { useFontSize } from 'dashboard/composables/useFontSize';
import { useBranding } from 'shared/composables/useBranding';
import { clearCookiesOnLogout } from 'dashboard/store/utils/api.js';
import { copyTextToClipboard } from 'shared/helpers/clipboard';
import { parseAPIErrorResponse } from 'dashboard/store/utils/api';
import { parseBoolean } from '@chatwoot/utils';
import UserProfilePicture from './UserProfilePicture.vue';
import UserBasicDetails from './UserBasicDetails.vue';
import FontSize from './FontSize.vue';
import UserLanguageSelect from './UserLanguageSelect.vue';
import HotKeyCard from './HotKeyCard.vue';
import ChangePassword from './ChangePassword.vue';
import NotificationPreferences from './NotificationPreferences.vue';
import AudioNotifications from './AudioNotifications.vue';
import SectionLayout from '../account/components/SectionLayout.vue';
// import AccessToken from './AccessToken.vue';
import MfaSettingsCard from './MfaSettingsCard.vue';
import Policy from 'dashboard/components/policy.vue';
import {
  ROLES,
  CONVERSATION_PERMISSIONS,
} from 'dashboard/constants/permissions.js';

export default {
  components: {
    SectionLayout,
    FontSize,
    UserLanguageSelect,
    UserProfilePicture,
    Policy,
    UserBasicDetails,
    HotKeyCard,
    ChangePassword,
    NotificationPreferences,
    AudioNotifications,
    // AccessToken,
    MfaSettingsCard,
  },
  setup() {
    const { isEditorHotKeyEnabled, updateUISettings } = useUISettings();
    const { currentFontSize, updateFontSize } = useFontSize();
    const { replaceInstallationName } = useBranding();
    return {
      currentFontSize,
      updateFontSize,
      isEditorHotKeyEnabled,
      updateUISettings,
      replaceInstallationName,
    };
  },
  data() {
    return {
      avatarFile: '',
      avatarUrl: '',
      name: '',
      displayName: '',
      email: '',
      hotKeys: [
        {
          key: 'enter',
          title: this.$t(
            'PROFILE_SETTINGS.FORM.SEND_MESSAGE.CARD.ENTER_KEY.HEADING'
          ),
          description: this.$t(
            'PROFILE_SETTINGS.FORM.SEND_MESSAGE.CARD.ENTER_KEY.CONTENT'
          ),
          lightImage: '/assets/images/dashboard/profile/hot-key-enter.svg',
          darkImage: '/assets/images/dashboard/profile/hot-key-enter-dark.svg',
        },
        {
          key: 'cmd_enter',
          title: this.$t(
            'PROFILE_SETTINGS.FORM.SEND_MESSAGE.CARD.CMD_ENTER_KEY.HEADING'
          ),
          description: this.$t(
            'PROFILE_SETTINGS.FORM.SEND_MESSAGE.CARD.CMD_ENTER_KEY.CONTENT'
          ),
          lightImage: '/assets/images/dashboard/profile/hot-key-ctrl-enter.svg',
          darkImage:
            '/assets/images/dashboard/profile/hot-key-ctrl-enter-dark.svg',
        },
      ],
      notificationPermissions: [...ROLES, ...CONVERSATION_PERMISSIONS],
      audioNotificationPermissions: [...ROLES, ...CONVERSATION_PERMISSIONS],
    };
  },
  computed: {
    ...mapGetters({
      currentUser: 'getCurrentUser',
      currentUserId: 'getCurrentUserID',
      globalConfig: 'globalConfig/get',
    }),
    isMfaEnabled() {
      return parseBoolean(window.chatwootConfig?.isMfaEnabled);
    },
    sections() {
      return [
        {
          id: 'profile',
          title: this.$t('PROFILE_SETTINGS.SECTIONS.PROFILE.TITLE'),
          description: this.$t('PROFILE_SETTINGS.SECTIONS.PROFILE.DESCRIPTION'),
          visible: true,
          withBorder: false,
        },
        {
          id: 'password',
          title: this.$t('PROFILE_SETTINGS.SECTIONS.PASSWORD.TITLE'),
          description: this.$t(
            'PROFILE_SETTINGS.SECTIONS.PASSWORD.DESCRIPTION'
          ),
          visible: !this.globalConfig.disableUserProfileUpdate,
          withBorder: true,
        },
        {
          id: 'mfa',
          title: this.$t('PROFILE_SETTINGS.SECTIONS.MFA.TITLE'),
          description: this.$t('PROFILE_SETTINGS.FORM.SECURITY_SECTION.NOTE'),
          visible: this.isMfaEnabled,
          withBorder: true,
        },
        {
          id: 'display',
          title: this.$t('PROFILE_SETTINGS.SECTIONS.DISPLAY.TITLE'),
          description: this.replaceInstallationName(
            this.$t('PROFILE_SETTINGS.FORM.INTERFACE_SECTION.NOTE')
          ),
          visible: true,
          withBorder: true,
        },
        {
          id: 'shortcut',
          title: this.$t('PROFILE_SETTINGS.SECTIONS.SHORTCUT.TITLE'),
          description: this.$t('PROFILE_SETTINGS.FORM.SEND_MESSAGE.NOTE'),
          visible: true,
          withBorder: true,
        },
        {
          id: 'notifications',
          title: this.$t('PROFILE_SETTINGS.SECTIONS.NOTIFICATIONS.TITLE'),
          description: this.$t(
            'PROFILE_SETTINGS.SECTIONS.NOTIFICATIONS.DESCRIPTION'
          ),
          visible: true,
          withBorder: true,
        },
        {
          id: 'audio',
          title: this.$t('PROFILE_SETTINGS.SECTIONS.AUDIO.TITLE'),
          description: this.$t(
            'PROFILE_SETTINGS.FORM.AUDIO_NOTIFICATIONS_SECTION.NOTE'
          ),
          visible: true,
          withBorder: true,
        },
        // {
        //   id: 'token',
        //   title: this.$t('PROFILE_SETTINGS.SECTIONS.TOKEN.TITLE'),
        //   description: this.replaceInstallationName(
        //     this.$t('PROFILE_SETTINGS.FORM.ACCESS_TOKEN.NOTE')
        //   ),
        //   visible: true,
        //   withBorder: true,
        // },
      ]
        .filter(s => s.visible)
        .map((s, i) => ({ ...s, number: String(i + 1).padStart(2, '0') }));
    },
    sectionMap() {
      return Object.fromEntries(this.sections.map(s => [s.id, s]));
    },
  },
  mounted() {
    if (this.currentUserId) this.initializeUser();
  },
  methods: {
    initializeUser() {
      this.name = this.currentUser.name;
      this.email = this.currentUser.email;
      this.avatarUrl = this.currentUser.avatar_url;
      this.displayName = this.currentUser.display_name;
    },
    async dispatchUpdate(payload, successMessage, errorMessage) {
      let alertMessage = '';
      try {
        await this.$store.dispatch('updateProfile', payload);
        alertMessage = successMessage;
        return true;
      } catch (error) {
        alertMessage = parseAPIErrorResponse(error) || errorMessage;
        return false;
      } finally {
        useAlert(alertMessage);
      }
    },
    async updateProfile(userAttributes) {
      const { name, email, displayName } = userAttributes;
      const hasEmailChanged = this.currentUser.email !== email;
      this.name = name || this.name;
      this.email = email || this.email;
      this.displayName = displayName || this.displayName;
      const success = await this.dispatchUpdate(
        {
          name: this.name,
          email: this.email,
          displayName: this.displayName,
          avatar: this.avatarFile,
        },
        hasEmailChanged
          ? this.$t('PROFILE_SETTINGS.AFTER_EMAIL_CHANGED')
          : this.$t('PROFILE_SETTINGS.UPDATE_SUCCESS'),
        this.$t('RESET_PASSWORD.API.ERROR_MESSAGE')
      );
      if (hasEmailChanged && success) clearCookiesOnLogout();
    },
    updateProfilePicture({ file, url }) {
      this.avatarFile = file;
      this.avatarUrl = url;
    },
    async deleteProfilePicture() {
      try {
        await this.$store.dispatch('deleteAvatar');
        this.avatarUrl = '';
        this.avatarFile = '';
        useAlert(this.$t('PROFILE_SETTINGS.AVATAR_DELETE_SUCCESS'));
      } catch {
        useAlert(this.$t('PROFILE_SETTINGS.AVATAR_DELETE_FAILED'));
      }
    },
    toggleHotKey(key) {
      this.hotKeys = this.hotKeys.map(h =>
        h.key === key ? { ...h, active: !h.active } : h
      );
      this.updateUISettings({ editor_message_key: key });
      useAlert(this.$t('PROFILE_SETTINGS.FORM.SEND_MESSAGE.UPDATE_SUCCESS'));
    },
    async onCopyToken(value) {
      await copyTextToClipboard(value);
      useAlert(this.$t('COMPONENTS.CODE.COPY_SUCCESSFUL'));
    },
    // async resetAccessToken() {
    //   const success = await this.$store.dispatch('resetAccessToken');
    //   useAlert(
    //     success
    //       ? this.$t('PROFILE_SETTINGS.FORM.ACCESS_TOKEN.RESET_SUCCESS')
    //       : this.$t('PROFILE_SETTINGS.FORM.ACCESS_TOKEN.RESET_ERROR')
    //   );
    // },
  },
};
</script>

<template>
  <div class="flex flex-col max-w-2xl ltr:mr-auto rtl:ml-auto">
    <div class="pb-6 border-b border-white/10 mb-2">
      <!-- FIX: no-bare-strings-in-template — заменён хардкод 'Account' на i18n-ключ -->
      <p
        class="text-xs font-semibold tracking-[0.2em] text-[#4ade80] uppercase mb-1"
      >
        {{ $t('PROFILE_SETTINGS.ACCOUNT_LABEL') }}
      </p>
      <h2 class="text-3xl font-black tracking-wide text-white uppercase">
        {{ $t('PROFILE_SETTINGS.TITLE') }}
      </h2>
    </div>

    <SectionLayout
      v-if="sectionMap.profile"
      :title="sectionMap.profile.title"
      :description="sectionMap.profile.description"
      :section-number="sectionMap.profile.number"
      :with-border="sectionMap.profile.withBorder"
    >
      <div class="flex flex-col gap-4">
        <UserProfilePicture
          :src="avatarUrl"
          :name="name"
          @change="updateProfilePicture"
          @delete="deleteProfilePicture"
        />
        <UserBasicDetails
          :name="name"
          :display-name="displayName"
          :email="email"
          :email-enabled="!globalConfig.disableUserProfileUpdate"
          @update-user="updateProfile"
        />
      </div>
    </SectionLayout>

    <SectionLayout
      v-if="sectionMap.password"
      :title="sectionMap.password.title"
      :description="sectionMap.password.description"
      :section-number="sectionMap.password.number"
      :with-border="sectionMap.password.withBorder"
    >
      <ChangePassword />
    </SectionLayout>

    <SectionLayout
      v-if="sectionMap.mfa"
      :title="sectionMap.mfa.title"
      :description="sectionMap.mfa.description"
      :section-number="sectionMap.mfa.number"
      :with-border="sectionMap.mfa.withBorder"
    >
      <MfaSettingsCard />
    </SectionLayout>

    <SectionLayout
      v-if="sectionMap.display"
      :title="sectionMap.display.title"
      :description="sectionMap.display.description"
      :section-number="sectionMap.display.number"
      :with-border="sectionMap.display.withBorder"
    >
      <div class="flex flex-col gap-3">
        <UserLanguageSelect
          :label="$t('PROFILE_SETTINGS.FORM.INTERFACE_SECTION.LANGUAGE.TITLE')"
          :description="
            $t('PROFILE_SETTINGS.FORM.INTERFACE_SECTION.LANGUAGE.NOTE')
          "
        />
        <FontSize
          :value="currentFontSize"
          :label="$t('PROFILE_SETTINGS.FORM.INTERFACE_SECTION.FONT_SIZE.TITLE')"
          :description="
            $t('PROFILE_SETTINGS.FORM.INTERFACE_SECTION.FONT_SIZE.NOTE')
          "
          @change="updateFontSize"
        />
      </div>
    </SectionLayout>

    <SectionLayout
      v-if="sectionMap.shortcut"
      :title="sectionMap.shortcut.title"
      :description="sectionMap.shortcut.description"
      :section-number="sectionMap.shortcut.number"
      :with-border="sectionMap.shortcut.withBorder"
    >
      <div class="grid grid-cols-2 gap-3">
        <HotKeyCard
          v-for="hotKey in hotKeys"
          :key="hotKey.key"
          :title="hotKey.title"
          :description="hotKey.description"
          :light-image="hotKey.lightImage"
          :dark-image="hotKey.darkImage"
          :active="isEditorHotKeyEnabled(hotKey.key)"
          @click="toggleHotKey(hotKey.key)"
        />
      </div>
    </SectionLayout>

    <Policy
      v-if="sectionMap.notifications"
      :permissions="notificationPermissions"
    >
      <SectionLayout
        :title="sectionMap.notifications.title"
        :description="sectionMap.notifications.description"
        :section-number="sectionMap.notifications.number"
        :with-border="sectionMap.notifications.withBorder"
      >
        <NotificationPreferences />
      </SectionLayout>
    </Policy>

    <Policy v-if="sectionMap.audio" :permissions="audioNotificationPermissions">
      <SectionLayout
        :title="sectionMap.audio.title"
        :description="sectionMap.audio.description"
        :section-number="sectionMap.audio.number"
        :with-border="sectionMap.audio.withBorder"
      >
        <AudioNotifications />
      </SectionLayout>
    </Policy>

    <!-- <SectionLayout
      v-if="sectionMap.token"
      :title="sectionMap.token.title"
      :description="sectionMap.token.description"
      :section-number="sectionMap.token.number"
      :with-border="sectionMap.token.withBorder"
    >
      <AccessToken
        :value="currentUser.access_token"
        @on-copy="onCopyToken"
        @on-reset="resetAccessToken"
      />
    </SectionLayout> -->
  </div>
</template>
