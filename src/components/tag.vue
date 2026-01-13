<template>
  <div
    class="rounded-xl px-2 flex justify-center items-center text-white w-18.5"
    :class="{ 'w-fit': onlyInitialLetter }"
    :style="{ backgroundColor: rarityTagColor }"
  >
    <span class="font-medium">{{ formatedLabel }}</span>
  </div>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  rarityLabel: String,
  rarityTagColor: String,
  onlyInitialLetter: Boolean,
});

const formatedLabel = computed(() => {
  return props.onlyInitialLetter ? props.rarityLabel[0] : props.rarityLabel;
});
</script>
