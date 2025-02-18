<script setup lang="ts">
import { onMounted } from "vue"
import VEmojiAvatar from "~/components/VEmojiAvatar.vue"
import { useTrans } from "~/composable/locales"
import { languageOptions, modelOptions, sendKeyOptions, themeOptions, useSettingStore } from "~/composable/settings"
import SettingItem from "~/pages/chat/settings/SettingItem.vue"

const settingStore = useSettingStore()
const settings = settingStore.settings
onMounted(() => {
  settingStore.fetchRemoteLatestCommitDate()
})

const { t, setLocale } = useTrans()

const apiKeyShow = ref(false)
const usageReloading = ref(false)
const checkUsage = () => {
  if (usageReloading.value) {
    return
  }
  usageReloading.value = true
  setTimeout(() => {
    usageReloading.value = false
  }, 1000)
}

watch(
  () => settings.language,
  () => {
    setLocale(settings.language)
  }
)
</script>
<template>
  <ClientOnly>
    <div class="flex flex-1 flex-col">
      <div class="flex items-center justify-between border-b border-gray-200 p-3.5">
        <div class="truncate">
          <div class="truncate text-[1.25rem] font-bold">
            {{ t("Settings.Title") }}
          </div>
          <div class="mt-1 text-[0.88rem]">
            {{ t("Settings.SubTitle") }}
          </div>
        </div>
        <div class="flex">
          <div class="ml-3">
            <HeadIconButton icon="i-mdi-bin-outline" size="1.3em" />
          </div>
          <div class="ml-3">
            <HeadIconButton icon="i-mdi-reload" size="1.3em" />
          </div>
          <div class="ml-3">
            <HeadIconButton icon="i-mdi-close" size="1.3em" />
          </div>
        </div>
      </div>
      <div class="overflow-scroll p-5">
        <div class="mb-5 divide-y rounded-xl border shadow-sm">
          <SettingItem :title="t('Settings.Avatar')">
            <VEmojiAvatar v-model="settings.avatar" />
          </SettingItem>

          <SettingItem
            :title="t('Settings.Update.Version', { x: settings.latestCommitDate })"
            :subtitle="
              settings.hasNewVersion
                ? t('Settings.Update.FoundUpdate', { x: `${settings.remoteLatestCommitDate}` })
                : ``
            "
          >
            <NuxtLink
              target="_blank"
              href="https://github.com/hylarucoder/ChatGPT-Nuxt#keep-updated"
              class="cursor-pointer text-[0.75rem] text-emerald-400"
              >{{ settings.hasNewVersion ? t("Settings.Update.GoToUpdate") : t("Settings.Update.IsLatest") }}
            </NuxtLink>
          </SettingItem>

          <SettingItem :title="t('Settings.SendKey')">
            <USelect class="min-w-100" :options="sendKeyOptions" v-model="settings.sendKey" />
          </SettingItem>

          <SettingItem :title="t('Settings.Theme')">
            <USelect :options="themeOptions" v-model="settings.theme" />
          </SettingItem>

          <SettingItem :title="t('Settings.Lang.Name')">
            <USelect :options="languageOptions" v-model="settings.language" />
          </SettingItem>

          <SettingItem :title="t('Settings.FontSize.Title')" :subtitle="t('Settings.FontSize.SubTitle')">
            <div class="flex w-40 items-center justify-center">
              <span class="mr-2 w-6"> {{ settings.fontSize }}px </span>
              <URange step="1" max="18" min="10" v-model="settings.fontSize" />
            </div>
          </SettingItem>

          <SettingItem :title="t('Settings.Mask.Title')" :subtitle="t('Settings.Mask.SubTitle')">
            <UCheckbox v-model="settings.maskLaunchPage" />
          </SettingItem>
        </div>
        <div class="mb-5 divide-y rounded-xl border shadow-sm">
          <SettingItem :title="t('Settings.Endpoint.Title')" :subtitle="t('Settings.Endpoint.SubTitle')">
            <div class="flex justify-end">
              <UInput v-model="settings.serverUrl" />
            </div>
          </SettingItem>
          <SettingItem :title="t('Settings.Token.Title')" :subtitle="t('Settings.Token.SubTitle')">
            <div class="flex justify-end">
              <button
                class="mr-1 flex h-9 w-9 cursor-pointer items-center justify-center truncate rounded-xl p-3 text-center hover:bg-gray-200"
                @click="apiKeyShow = !apiKeyShow"
              >
                <div class="flex items-center justify-center">
                  <span class="i-mdi-eye-outline h-4 w-4" v-if="apiKeyShow" />
                  <span class="i-mdi-eye-off-outline h-4 w-5" v-if="!apiKeyShow" />
                </div>
              </button>
              <UInput
                v-model="settings.apiKey"
                :type="!apiKeyShow ? `password` : `text`"
                :placeholder="t('Settings.Token.Placeholder')"
              />
            </div>
          </SettingItem>

          <SettingItem
            :title="t('Settings.Usage.Title')"
            :subtitle="t('Settings.Usage.SubTitle', { used: `[?]`, total: `[?]` })"
          >
            <button
              @click="checkUsage()"
              class="flex h-10 w-24 cursor-pointer items-center justify-center truncate rounded-xl p-3 text-center hover:bg-gray-200"
            >
              <div :class="{ 'animate-spin': usageReloading }" class="flex items-center justify-center">
                <span class="i-mdi-reload h-4 w-4" />
              </div>
              <div class="ml-1 truncate text-[0.75rem]">{{ t("Settings.Usage.Check") }}</div>
            </button>
          </SettingItem>
        </div>
        <div class="mb-5 divide-y rounded-xl border shadow-sm">
          <SettingItem :title="t('Settings.Prompt.Disable.Title')" :subtitle="t('Settings.Prompt.Disable.SubTitle')">
            <UCheckbox v-model="settings.disableAutoCompletePrompt" />
          </SettingItem>

          <SettingItem
            :title="t('Settings.Prompt.List')"
            :subtitle="t('Settings.Prompt.ListCount', { builtin: 0, custom: 0 })"
          >
            <button
              class="flex h-10 cursor-pointer items-center justify-center truncate rounded-xl p-3 text-center hover:bg-gray-200"
            >
              <div class="flex items-center justify-center">
                <span class="i-mdi-pen h-4 w-4" />
              </div>
              <div class="ml-1 truncate text-[0.75rem]">{{ t("Settings.Prompt.Edit") }}</div>
            </button>
          </SettingItem>
        </div>
        <div class="mb-5 divide-y rounded-xl border shadow-sm">
          <SettingItem :title="t('Settings.Model')">
            <USelect searchable :options="modelOptions" v-model="settings.model" />
          </SettingItem>

          <SettingItem :title="t('Settings.Temperature.Title')" :subtitle="t('Settings.Temperature.SubTitle')">
            <div class="flex w-40 items-center justify-center">
              <span class="mr-2">{{ settings.temperature }}</span>
              <URange min="0.0" max="1.0" step="0.1" v-model="settings.temperature" />
            </div>
          </SettingItem>

          <SettingItem :title="t('Settings.MaxTokens.Title')" :subtitle="t('Settings.MaxTokens.SubTitle')">
            <div class="w-20">
              <UInput type="number" v-model="settings.maxTokens" />
            </div>
          </SettingItem>

          <SettingItem :title="t('Settings.PresencePenalty.Title')" :subtitle="t('Settings.PresencePenalty.SubTitle')">
            <div class="flex w-40 items-center justify-center">
              <span class="mr-2">
                {{ settings.presencePenalty }}
              </span>
              <URange :min="-2.0" :max="2.0" :step="0.1" v-model="settings.presencePenalty" />
            </div>
          </SettingItem>

          <SettingItem
            :title="t('Settings.FrequencyPenalty.Title')"
            :subtitle="t('Settings.FrequencyPenalty.SubTitle')"
          >
            <div class="flex w-40 items-center justify-center">
              <span class="mr-2">
                {{ settings.frequencyPenalty }}
              </span>
              <URange :min="-2.0" :max="2.0" :step="0.1" v-model="settings.frequencyPenalty" />
            </div>
          </SettingItem>

          <SettingItem :title="t('Settings.HistoryCount.Title')" :subtitle="t('Settings.HistoryCount.SubTitle')">
            <div class="flex w-40 items-center justify-center">
              <span class="mr-2">
                {{ settings.historyMessagesCount }}
              </span>
              <URange :min="0" :max="32" :step="1" v-model="settings.historyMessagesCount" />
            </div>
          </SettingItem>

          <SettingItem
            :title="t('Settings.CompressThreshold.Title')"
            :subtitle="t('Settings.CompressThreshold.SubTitle')"
          >
            <div class="w-20">
              <UInput type="number" v-model="settings.compressMessageLengthThreshold" />
            </div>
          </SettingItem>

          <SettingItem :title="t('Memory.Title')" :subtitle="t('Memory.Send')">
            <UCheckbox v-model="settings.historySummary" />
          </SettingItem>
        </div>
      </div>
    </div>
  </ClientOnly>
</template>
