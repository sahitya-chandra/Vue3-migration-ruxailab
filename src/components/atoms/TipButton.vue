<template>
  <v-dialog
    v-model="dialog"
    width="500"
  >
    <template #activator="{ props }">
      <v-btn
        color="green"
        class="ml-4 my-2 mr-auto text-white"
        variant="tonal"
        icon
        v-bind="props"
      >
        <v-icon>mdi-chat-question</v-icon>
      </v-btn>
    </template>

    <v-card>
      <v-card-title
        class="text-h5 bg-orange-lighten-1 text-center"
        style="color: white;"
        primary-title
      >
        {{ task.taskName }} - TIP
      </v-card-title>

      <v-card-text
        class="pa-3"
        style="text-align: center; color: black;"
      >
        {{ task.taskTip }}
      </v-card-text>

      <v-divider />

      <v-card-actions>
        <v-spacer />
        <v-btn
          color="orange"
          variant="text"
          @click="dialog = false"
        >
          Ok
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  value: {
    type: Boolean,
    default: false
  },
  task: {
    type: Object,
    default: () => ({
      taskName: 'Task',
      taskTip: 'No tip available'
    }),
    validator: (task) => {
      return typeof task.taskName === 'string' && 
             typeof task.taskTip === 'string'
    }
  }
})

const emit = defineEmits(['input'])

const dialog = ref(false)

watch(() => dialog.value, (newVal) => {
  emit('input', newVal)
})

watch(() => props.value, (newVal) => {
  dialog.value = newVal
}, { immediate: true })
</script>