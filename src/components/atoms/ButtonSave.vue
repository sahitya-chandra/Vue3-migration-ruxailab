<template>
  <v-container>
    <!-- Leave Alert Dialog -->
    <LeaveAlert />

    <v-tooltip
      v-if="visible"
      location="left"
    >
      <template #activator="{ props }">
        <v-btn
          size="large"
          icon
          fixed
          location="bottom right"
          color="#F9A826"
          :disabled="disabled"
         
          v-bind="props"
          @click="$emit('click')"
        >
          <v-icon size="large">
            mdi-content-save
          </v-icon>
        </v-btn>
      </template>

      <span>{{ $t('buttons.save') }}</span>
    </v-tooltip>
  </v-container>
</template>

<script>
import LeaveAlert from '@/components/atoms/LeaveAlert'

export default {
  components: {
    LeaveAlert,
  },

  props: {
    visible: {
      type: Boolean,
      default: false,
      required: true,
    },

    disabled: {
      type: Boolean,
      default: false,
      required: false,
    },
  },

  emits: ['click'],

  beforeMount() {
    window.addEventListener('beforeunload', this.preventNav)
  },

  beforeUnmount() {
    window.removeEventListener('beforeunload', this.preventNav)
  },

  methods: {
    preventNav(event) {
      if (!this.visible) return
      event.preventDefault()
    },
  },
}
</script>