<template>
  <div class="space-y-4">
    <FormSection title="Basic settings">
      <div class="grid grid-cols-1 gap-3 md:grid-cols-2">
        <FormField label="Background maintenance interval">
          <template #label-extra>
            <HelpTip text="In minutes. Controls how often background processing runs for accounts pending verification, rate-limited accounts, and AT tokens that are about to expire." />
          </template>
          <SettingsNumberInput :field="refreshAccountIntervalField" />
        </FormField>

        <FormField label="Image access URL">
          <template #label-extra>
            <HelpTip text="URL prefix used to build access links for generated images." />
          </template>
          <Input
            v-model.trim="settings.base_url"
            block
            :disabled="fieldReadOnly('base_url')"
            placeholder="https://example.com"
          />
        </FormField>

        <FormField label="Image auto-cleanup">
          <template #label-extra>
            <HelpTip text="Automatically delete local images older than this many hours." />
          </template>
          <SettingsNumberInput :field="imageRetentionHoursField" />
        </FormField>

        <FormField label="Log auto-cleanup">
          <template #label-extra>
            <HelpTip text="Automatically delete console call logs older than this many hours." />
          </template>
          <SettingsNumberInput :field="logRetentionHoursField" />
        </FormField>

        <FormField label="Image concurrency per account">
          <template #label-extra>
            <HelpTip text="Limits how many image requests each account can process at the same time. Defaults to 1; can be set to 1–3." />
          </template>
          <SettingsNumberInput :field="imageAccountConcurrencyField" />
        </FormField>

        <FormField label="Account batch task concurrency">
          <template #label-extra>
            <HelpTip text="Controls the maximum concurrency of account batch tasks. Refreshing AT, syncing accounts and quotas, import verification, and background rechecks use one concurrency slot per account; enabling, disabling, resetting, deleting, and batch saving use one slot for the whole batch. Defaults to 30; can be set to 1–100. Image generation concurrency is configured separately." />
          </template>
          <SettingsNumberInput :field="accountProcessingConcurrencyField" />
        </FormField>
      </div>
    </FormSection>

    <FormSection title="Requests and tasks">
      <div class="grid grid-cols-1 gap-3 md:grid-cols-2">
        <FormField label="Console request timeout">
          <template #label-extra>
            <HelpTip text="In seconds. The longest the console waits for a backend response on requests such as logs, images, accounts, and settings." />
          </template>
          <SettingsNumberInput :field="consoleRequestTimeoutField" />
        </FormField>

      </div>
    </FormSection>

    <FormSection title="Image generation">
      <div class="grid grid-cols-1 gap-3 md:grid-cols-2">
        <FormField label="Image stream wait limit">
          <template #label-extra>
            <HelpTip text="In seconds. Limits how long to wait on the ChatGPT image-generation SSE stream." />
          </template>
          <SettingsNumberInput :field="imageStreamTimeoutField" />
        </FormField>

        <FormField label="Image result wait limit">
          <template #label-extra>
            <HelpTip text="In seconds. The longest to keep waiting for the upstream image result after the SSE stream ends." />
          </template>
          <SettingsNumberInput :field="imagePollTimeoutField" />
        </FormField>

        <FormField label="Initial image polling delay">
          <template #label-extra>
            <HelpTip text="In seconds. How long to wait before starting to query for the image result." />
          </template>
          <SettingsNumberInput :field="imagePollInitialWaitField" />
        </FormField>

        <FormField label="Image polling interval">
          <template #label-extra>
            <HelpTip text="In seconds. The wait time between two image result queries." />
          </template>
          <SettingsNumberInput :field="imagePollIntervalField" />
        </FormField>
      </div>
    </FormSection>
  </div>
</template>

<script setup lang="ts">
import { FormField, FormSection, HelpTip, Input } from 'nanocat-ui'
import type { Settings } from '@/types/api'
import SettingsNumberInput from '@/views/settings/SettingsNumberInput.vue'
import { settingsFieldReadOnly, type SettingsFields } from '@/views/settings/settingsView'
import type { NumberSettingField } from '@/views/settings/useNumberSettingField'

const props = defineProps<{
  settings: Settings
  fields: SettingsFields
  refreshAccountIntervalField: NumberSettingField
  imageRetentionHoursField: NumberSettingField
  logRetentionHoursField: NumberSettingField
  consoleRequestTimeoutField: NumberSettingField
  imagePollTimeoutField: NumberSettingField
  imageStreamTimeoutField: NumberSettingField
  imagePollInitialWaitField: NumberSettingField
  imagePollIntervalField: NumberSettingField
  imageAccountConcurrencyField: NumberSettingField
  accountProcessingConcurrencyField: NumberSettingField
}>()

const fieldReadOnly = (path: string) => settingsFieldReadOnly(props.fields, path)
</script>
