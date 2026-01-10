<template>
  <div
    class="rounded-xl px-2 w-18.5 flex justify-center items-center text-white"
    :style="{ backgroundColor: rarityTagColor }"
  >
    <span class="font-medium">{{ rarityLabel }}</span>
  </div>
</template>

<script setup>
defineProps({
  rarityLabel: String,
  rarityTagColor: String,
});
</script>
