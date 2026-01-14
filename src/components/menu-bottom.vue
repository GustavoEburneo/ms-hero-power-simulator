<template>
  <div
    class="text-white text-xs mt-1 p-2 bg-black rounded-xl flex justify-between items-center"
  >
    <div>
      <span>Ability Reconfiguration Level </span>
      <input
        type="number"
        min="1"
        max="20"
        class="bg-white w-10 text-gray-400 text-center rounded p-1"
        v-model="abilityLevel"
      />
    </div>
    <div>
      <p class="mb-1 pl-1">
        Honor cost:
        <span class="text-[#BED844]">
          {{ honorCost }}
        </span>
      </p>
      <button
        class="bg-lime-600 px-4 py-2 rounded-md cursor-pointer hover:bg-lime-700 disabled:bg-gray-800 disabled:cursor-default"
        :disabled="blockedCount === stage - 1"
        @mousedown.prevent="emit('start-hold')"
        @mouseup="emit('stop-hold')"
        @mouseleave="emit('stop-hold')"
        @touchstart.prevent="emit('start-hold')"
        @touchend="emit('stop-hold')"
      >
        Change Option
      </button>
    </div>
  </div>
</template>

<script setup>
import { computed, onBeforeUnmount } from "vue";
import { reconfigCost } from "../consts/reconfig-cost";

const abilityLevel = defineModel();

const emit = defineEmits(["start-hold", "stop-hold"]);

const props = defineProps({
  blockedCount: Number,
  stage: Number,
});

const honorCost = computed(() => {
  if (abilityLevel.value <= 0) {
    abilityLevel.value = 0;
    return reconfigCost[abilityLevel.value][props.blockedCount];
  }

  if (abilityLevel.value > 20) {
    abilityLevel.value = 20;
    return reconfigCost[abilityLevel.value - 1][props.blockedCount];
  }

  return reconfigCost[abilityLevel.value - 1][props.blockedCount];
});
</script>
