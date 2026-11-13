<!--
  Copyright (C) 2022 Nethesis S.r.l.
  SPDX-License-Identifier: GPL-3.0-or-later
-->
<template>
  <cv-grid fullWidth>
    <cv-row>
      <cv-column class="page-title">
        <h2>{{ $t("settings.title") }}</h2>
      </cv-column>
    </cv-row>
    <cv-row v-if="error.getConfiguration">
      <cv-column>
        <NsInlineNotification
          kind="error"
          :title="$t('action.get-configuration')"
          :description="error.getConfiguration"
          :showCloseButton="false"
        />
      </cv-column>
    </cv-row>
    <cv-row>
      <cv-column>
        <cv-tile light>
          <cv-form @submit.prevent="configureModule">
            <cv-text-input
              :label="$t('settings.gitea_fqdn')"
              placeholder="gitea.example.org"
              v-model.trim="host"
              class="mg-bottom"
              :invalid-message="$t(error.host)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="host"
            >
            </cv-text-input>
            <cv-toggle
              value="letsEncrypt"
              :label="$t('settings.lets_encrypt')"
              v-model="isLetsEncryptEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
            <cv-toggle
              value="httpToHttps"
              :label="$t('settings.http_to_https')"
              v-model="isHttpToHttpsEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
            <cv-select
              :label="$t('settings.setup_mode')"
              v-model="setupMode"
              class="mg-bottom"
              :disabled="
                setupModeLocked ||
                loading.getConfiguration ||
                loading.configureModule
              "
              :invalid-message="$t(error.setup_mode)"
              ref="setup_mode"
            >
              <cv-select-option value="" disabled>{{
                $t("settings.choose_setup_mode")
              }}</cv-select-option>
              <cv-select-option value="managed">{{
                $t("settings.setup_mode_managed")
              }}</cv-select-option>
              <cv-select-option value="manual">{{
                $t("settings.setup_mode_manual")
              }}</cv-select-option>
            </cv-select>
            <p v-if="setupModeLocked" class="mg-bottom">
              {{ $t("settings.setup_mode_locked") }}
            </p>
            <cv-accordion ref="accordion" class="maxwidth mg-bottom">
              <cv-accordion-item :open="false">
                <template slot="title">{{
                  $t("settings.active_directory")
                }}</template>
                <template slot="content">
                  <cv-toggle
                    value="adEnabled"
                    :label="$t('settings.ad_enabled')"
                    v-model="adEnabled"
                    :disabled="
                      setupMode !== 'managed' ||
                      loading.getConfiguration ||
                      loading.configureModule
                    "
                    class="mg-bottom"
                  >
                    <template slot="text-left">{{
                      $t("settings.disabled")
                    }}</template>
                    <template slot="text-right">{{
                      $t("settings.enabled")
                    }}</template>
                  </cv-toggle>
                  <cv-text-input
                    :label="$t('settings.ad_domain')"
                    placeholder="ad.example.org"
                    v-model.trim="adDomain"
                    class="mg-bottom"
                    :invalid-message="$t(error.ad_domain)"
                    :disabled="
                      !adEnabled ||
                      loading.getConfiguration ||
                      loading.configureModule
                    "
                    ref="ad_domain"
                  ></cv-text-input>
                  <cv-text-input
                    :label="$t('settings.ad_user_group')"
                    v-model.trim="adUserGroup"
                    class="mg-bottom"
                    :disabled="
                      !adEnabled ||
                      loading.getConfiguration ||
                      loading.configureModule
                    "
                  ></cv-text-input>
                  <cv-text-input
                    :label="$t('settings.ad_admin_group')"
                    v-model.trim="adAdminGroup"
                    class="mg-bottom"
                    :disabled="
                      !adEnabled ||
                      loading.getConfiguration ||
                      loading.configureModule
                    "
                  ></cv-text-input>
                  <cv-text-input
                    :label="$t('settings.ad_user_search_base')"
                    placeholder="CN=Users,DC=ad,DC=example,DC=org"
                    v-model.trim="adUserSearchBase"
                    class="mg-bottom"
                    :disabled="
                      !adEnabled ||
                      loading.getConfiguration ||
                      loading.configureModule
                    "
                  ></cv-text-input>
                  <cv-toggle
                    value="adNested"
                    :label="$t('settings.ad_nested_groups')"
                    v-model="adNestedGroups"
                    :disabled="
                      !adEnabled ||
                      loading.getConfiguration ||
                      loading.configureModule
                    "
                    class="mg-bottom"
                  >
                    <template slot="text-left">{{
                      $t("settings.disabled")
                    }}</template>
                    <template slot="text-right">{{
                      $t("settings.enabled")
                    }}</template>
                  </cv-toggle>
                </template>
              </cv-accordion-item>
            </cv-accordion>
            <cv-row v-if="error.configureModule">
              <cv-column>
                <NsInlineNotification
                  kind="error"
                  :title="$t('action.configure-module')"
                  :description="error.configureModule"
                  :showCloseButton="false"
                />
              </cv-column>
            </cv-row>
            <NsButton
              kind="primary"
              :icon="Save20"
              :loading="loading.configureModule"
              :disabled="loading.getConfiguration || loading.configureModule"
              >{{ $t("settings.save") }}</NsButton
            >
          </cv-form>
        </cv-tile>
      </cv-column>
    </cv-row>
  </cv-grid>
</template>

<script>
import to from "await-to-js";
import { mapState } from "vuex";
import {
  QueryParamService,
  UtilService,
  TaskService,
  IconService,
  PageTitleService,
} from "@nethserver/ns8-ui-lib";

export default {
  name: "Settings",
  mixins: [
    TaskService,
    IconService,
    UtilService,
    QueryParamService,
    PageTitleService,
  ],
  pageTitle() {
    return this.$t("settings.title") + " - " + this.appName;
  },
  data() {
    return {
      q: {
        page: "settings",
      },
      urlCheckInterval: null,
      host: "",
      isLetsEncryptEnabled: false,
      isHttpToHttpsEnabled: true,
      setupMode: "",
      setupModeLocked: false,
      adEnabled: false,
      adDomain: "",
      adUserGroup: "gitea-user",
      adAdminGroup: "gitea-admin",
      adUserSearchBase: "",
      adNestedGroups: false,
      loading: {
        getConfiguration: false,
        configureModule: false,
      },
      error: {
        getConfiguration: "",
        configureModule: "",
        host: "",
        lets_encrypt: "",
        http2https: "",
        setup_mode: "",
        ad_domain: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
  },
  created() {
    this.getConfiguration();
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.watchQueryData(vm);
      vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
    });
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.urlCheckInterval);
    next();
  },
  methods: {
    async getConfiguration() {
      this.loading.getConfiguration = true;
      this.error.getConfiguration = "";
      const taskAction = "get-configuration";
      const eventId = this.getUuid();

      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.getConfigurationAborted
      );

      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.getConfigurationCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.getConfiguration = this.getErrorMessage(err);
        this.loading.getConfiguration = false;
        return;
      }
    },
    getConfigurationAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.getConfiguration = this.$t("error.generic_error");
      this.loading.getConfiguration = false;
    },
    getConfigurationCompleted(taskContext, taskResult) {
      const config = taskResult.output;
      this.host = config.host;
      this.isLetsEncryptEnabled = config.lets_encrypt;
      this.isHttpToHttpsEnabled = config.http2https;
      this.setupMode =
        config.setup_mode === "managed" || config.setup_mode === "manual"
          ? config.setup_mode
          : "";
      // the mode cannot be changed once the instance has been configured
      this.setupModeLocked = this.setupMode !== "";
      this.adEnabled = config.ad_enabled;
      this.adDomain = config.ad_domain;
      this.adUserGroup = config.ad_user_group;
      this.adAdminGroup = config.ad_admin_group;
      this.adUserSearchBase = config.ad_user_search_base;
      this.adNestedGroups = config.ad_nested_groups;

      this.loading.getConfiguration = false;
      this.focusElement("host");
    },
    validateConfigureModule() {
      this.clearErrors(this);

      let isValidationOk = true;
      if (!this.host) {
        this.error.host = "common.required";

        if (isValidationOk) {
          this.focusElement("host");
        }
        isValidationOk = false;
      }
      if (!this.setupMode) {
        this.error.setup_mode = "common.required";
        if (isValidationOk) {
          this.focusElement("setup_mode");
        }
        isValidationOk = false;
      }
      if (this.adEnabled && !this.adDomain) {
        this.error.ad_domain = "common.required";
        if (isValidationOk) {
          this.focusElement("ad_domain");
        }
        isValidationOk = false;
      }
      return isValidationOk;
    },
    configureModuleValidationFailed(validationErrors) {
      this.loading.configureModule = false;
      let focusAlreadySet = false;

      for (const validationError of validationErrors) {
        const param = validationError.parameter;
        // set i18n error message
        this.error[param] = this.$t("settings." + validationError.error);

        if (!focusAlreadySet) {
          this.focusElement(param);
          focusAlreadySet = true;
        }
      }
    },
    async configureModule() {
      const isValidationOk = this.validateConfigureModule();
      if (!isValidationOk) {
        return;
      }

      this.loading.configureModule = true;
      const taskAction = "configure-module";
      const eventId = this.getUuid();

      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.configureModuleAborted
      );

      // register to task validation
      this.core.$root.$once(
        `${taskAction}-validation-failed-${eventId}`,
        this.configureModuleValidationFailed
      );

      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.configureModuleCompleted
      );
      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            host: this.host,
            lets_encrypt: this.isLetsEncryptEnabled,
            http2https: this.isHttpToHttpsEnabled,
            setup_mode: this.setupMode,
            ad_enabled: this.adEnabled,
            ad_domain: this.adDomain,
            ad_user_group: this.adUserGroup,
            ad_admin_group: this.adAdminGroup,
            ad_user_search_base: this.adUserSearchBase,
            ad_nested_groups: this.adNestedGroups,
          },
          extra: {
            title: this.$t("settings.instance_configuration", {
              instance: this.instanceName,
            }),
            description: this.$t("settings.configuring"),
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.configureModule = this.getErrorMessage(err);
        this.loading.configureModule = false;
        return;
      }
    },
    configureModuleAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.configureModule = this.$t("error.generic_error");
      this.loading.configureModule = false;
    },
    configureModuleCompleted() {
      this.loading.configureModule = false;

      // reload configuration
      this.getConfiguration();
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";
.mg-bottom {
  margin-bottom: $spacing-06;
}

.maxwidth {
  max-width: 38rem;
}
</style>
