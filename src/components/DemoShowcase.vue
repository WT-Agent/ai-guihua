<template>
  <NomadsShowcase @apply-template="handleApply" />
</template>

<script setup lang="ts">
import NomadsShowcase from './NomadsShowcase.vue';

const emit = defineEmits<{
  (e: 'apply-template', payload: any): void;
}>();

function handleApply(payload: any) {
  emit('apply-template', payload);
}
</script>
