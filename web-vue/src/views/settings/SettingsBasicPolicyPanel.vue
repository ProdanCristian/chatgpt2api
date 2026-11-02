<template>
  <div class="space-y-4">
    <FormSection title="Account policy">
      <div class="settings-check-grid settings-check-grid--single">
        <div class="settings-check-item">
          <div class="settings-check-control">
            <Checkbox
              v-model="settings.auto_remove_invalid_accounts"
              :disabled="fieldReadOnly('auto_remove_invalid_accounts')"
            >Automatically remove abnormal accounts</Checkbox>
            <HelpTip text="Accounts whose authentication is confirmed invalid enter abnormal handling. When enabled they are removed directly; when disabled they keep the abnormal status." />
          </div>
        </div>
        <div class="settings-check-item">
          <div class="settings-check-control">
            <Checkbox
              v-model="settings.auto_remove_rate_limited_accounts"
              :disabled="fieldReadOnly('auto_remove_rate_limited_accounts')"
            >Automatically remove accounts with exhausted quota</Checkbox>
            <HelpTip text="Only handled when the remote side explicitly confirms the image quota is 0. Proxy errors, dropped streams, or upstream 429 responses will not delete an account." />
          </div>
        </div>
        <div class="settings-check-item">
          <div class="settings-check-control">
            <Checkbox
              v-model="settings.image_account_retry_enabled"
              :disabled="fieldReadOnly('image_account_retry_enabled')"
            >Try other accounts after image generation fails</Checkbox>
            <HelpTip text="Except for text results (HTTP 400), when the current account fails to deliver an image, another account is tried immediately. The current request does not wait for background account verification." />
          </div>
        </div>
      </div>
      <FormField label="Maximum accounts to try">
        <template #label-extra>
          <HelpTip text="Includes the first account used. Defaults to 4, minimum 2, with no upper limit on the value you can enter." />
        </template>
        <SettingsNumberInput
          :field="imageMaxAccountAttemptsField"
          :disabled="!settings.image_account_retry_enabled"
        />
      </FormField>
    </FormSection>

    <FormSection title="Image confirmation">
      <div class="settings-check-grid settings-check-grid--single">
        <div class="settings-check-item">
          <div class="settings-check-control">
            <Checkbox
              v-model="settings.image_settle_enabled"
              :disabled="fieldReadOnly('image_settle_enabled')"
            >Image double-check mechanism</Checkbox>
            <HelpTip text="After an image result is found, wait the specified number of seconds and check once more, reducing early returns before the result has stabilized." />
          </div>
        </div>
        <div class="settings-check-item">
          <div class="settings-check-control">
            <Checkbox
              v-model="settings.image_remove_conversation_after_result"
              :disabled="fieldReadOnly('image_remove_conversation_after_result')"
            >Delete the official site conversation after image success</Checkbox>
            <HelpTip text="Off by default. Only after the image has been saved successfully does it try to hide the ChatGPT website conversation; failures are only logged and do not affect the image response. When off, the website conversation is kept for recovery and troubleshooting." />
          </div>
        </div>
      </div>
      <FormField label="Double-check wait (seconds)">
        <SettingsNumberInput
          :field="imageSettleSecondsField"
          :disabled="!settings.image_settle_enabled"
        />
      </FormField>
    </FormSection>

    <FormSection title="Image upscaling">
      <div class="settings-check-grid settings-check-grid--single">
        <div class="settings-check-item">
          <div class="settings-check-control">
            <Checkbox
              v-model="settings.image_upscale_enabled"
              :disabled="fieldReadOnly('image_upscale_enabled')"
            >Automatically upscale when the image is too small</Checkbox>
            <HelpTip text="Off by default. Only applies to results where a size was explicitly requested and the upstream image is smaller than the target size; auto-sized images and images that already reach the target size are left unchanged." />
          </div>
        </div>
      </div>
      <FormField label="Upscaling engine">
        <GroupedSelectMenu
          v-model="settings.image_upscale_engine"
          :options="imageUpscaleEngineOptions"
          :disabled="!settings.image_upscale_enabled || fieldReadOnly('image_upscale_engine')"
          selected-indicator="none"
          aria-label="Image upscaling engine"
          block
        />
      </FormField>
    </FormSection>

    <FormSection title="Console log level">
      <div class="settings-check-grid settings-check-grid--single mt-3">
        <div
          v-for="level in logLevelOptions"
          :key="level.value"
          class="settings-check-item"
        >
          <div class="settings-check-control">
            <Checkbox
              :model-value="settings.log_levels.includes(level.value)"
              :disabled="fieldReadOnly('log_levels')"
              @update:model-value="$emit('setLogLevel', level.value, Boolean($event))"
            >
              {{ level.label }}
            </Checkbox>
            <HelpTip v-if="level.value === 'debug'" text="When no level is selected, the default info / warning / error is used." />
          </div>
        </div>
      </div>
    </FormSection>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { Checkbox, FormField, FormSection, HelpTip } from 'nanocat-ui'
import { GroupedSelectMenu } from 'nanocat-ui'
import type { Settings } from '@/types/api'
import SettingsNumberInput from '@/views/settings/SettingsNumberInput.vue'
import {
  settingsFieldOptions,
  settingsFieldReadOnly,
  type SettingsFields,
} from '@/views/settings/settingsView'
import type { NumberSettingField } from '@/views/settings/useNumberSettingField'

const props = defineProps<{
  settings: Settings
  fields: SettingsFields
  imageMaxAccountAttemptsField: NumberSettingField
  imageSettleSecondsField: NumberSettingField
}>()

defineEmits<{
  setLogLevel: [level: string, enabled: boolean]
}>()

const fieldReadOnly = (path: string) => settingsFieldReadOnly(props.fields, path)
const imageUpscaleEngineOptions = computed(() => (
  settingsFieldOptions(props.fields, 'image_upscale_engine', props.settings.image_upscale_engine)
))
const logLevelOptions = computed(() => (
  settingsFieldOptions(props.fields, 'log_levels', props.settings.log_levels)
))
</script>

<style scoped>
.settings-check-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(13.5rem, 1fr));
  gap: 8px;
}

.settings-check-grid--single {
  grid-template-columns: minmax(0, 1fr);
}

.settings-check-item {
  min-height: 38px;
  border: 1px solid hsl(var(--border));
  border-radius: 14px;
  background: hsl(var(--background) / 0.72);
  transition:
    border-color 0.16s ease,
    background-color 0.16s ease;
}

.settings-check-item:hover {
  border-color: hsl(var(--foreground) / 0.18);
  background: hsl(var(--muted) / 0.24);
}

.settings-check-control {
  display: flex;
  min-height: 38px;
  align-items: center;
  gap: 8px;
  padding-right: 10px;
}

.settings-check-item :deep(label) {
  display: flex;
  width: 100%;
  flex: 1;
  min-height: 38px;
  align-items: center;
  gap: 10px;
  padding: 9px 11px;
}

.settings-check-item :deep(label > span:last-child) {
  color: hsl(var(--foreground) / 0.78);
  line-height: 1.35;
}
</style>
