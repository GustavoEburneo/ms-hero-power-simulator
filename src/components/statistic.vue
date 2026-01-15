<template>
  <div
    class="text-white text-sm mt-2 pt-2 border-t flex justify-between items-center"
  >
    <div>
      <span>Total honor spent: </span>
      <span class="text-[#BED844]">{{ totalHonorSpent }}</span>
    </div>
    <button
      class="px-2 py-1 rounded cursor-pointer bg-lime-600 text-xs hover:bg-lime-700"
      @click="resetTotalHonorSpent"
    >
      RESET
    </button>
  </div>
</template>

<script setup>
const totalHonorSpent = defineModel("totalHonorSpent");

const resetTotalHonorSpent = () => {
  totalHonorSpent.value = 0;
};
</script>
