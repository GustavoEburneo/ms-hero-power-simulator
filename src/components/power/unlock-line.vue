<template>
  <div
    v-for="i in MAX_LINES_LENGTH - raritiesRandom.length"
    :key="i"
    class="bg-black rounded-xl p-2 flex items-center justify-center"
  >
    <span class="text-xs text-gray-400">
      Unlock upon reaching Hero Power Stage
      {{ i + raritiesRandom.length + 1 }}
    </span>
  </div>
</template>

<script setup>
const MAX_LINES_LENGTH = 6;

defineProps({
  raritiesRandom: Array,
});
</script>
