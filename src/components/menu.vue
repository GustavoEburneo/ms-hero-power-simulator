<template>
  <div class="text-xs">
    <span>Stage: </span>
    <input
      v-model="model"
      type="number"
      min="2"
      max="7"
      class="bg-white w-8 text-gray-400 text-center rounded"
    />
  </div>
</template>

<script setup>
const model = defineModel();
</script>
