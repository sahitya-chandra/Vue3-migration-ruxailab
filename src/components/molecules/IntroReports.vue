<template>
  <IntroComp
    :colors="['#ec6618', '#f54e42']"
    :title="'Reports'"
    :image="'IntroReports.svg'"
    :main="$t('descriptions.intro.reports')"
    :link="$t('descriptions.intro.invite')"
    :items="items"
    @link-clicked="goToCoops"
    @call-func="callFunc"
  />
</template>

<script setup>
import { computed } from 'vue'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import IntroComp from '@/components/atoms/IntrosComponent.vue'

const emit = defineEmits(['goToCoops'])
const router = useRouter()
const { t } = useI18n()

const items = computed(() => [
  {
    iconColor: '#ff6224',
    icon: 'mdi-file-document',
    title: t('pages.intros.docTitle'),
    subtitle: t('pages.intros.docSubtitle') + t('titles.reports'),
    func: 'goToDoc',
  },
])

const goToDoc = () => {
  router.push('/reports/documentation').catch(() => {})
}

const goToCoops = () => {
  emit('goToCoops')
}

const callFunc = (func) => {
  if (func === 'goToDoc') goToDoc()
}
</script>