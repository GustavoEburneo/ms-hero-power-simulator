<template>
  <div class="text-xs">
    <div>
      <span>Stage: </span>
      <input
        v-model="model"
        type="number"
        min="2"
        max="7"
        class="bg-white w-8 text-gray-400 text-center rounded"
      />
    </div>
    <div class="mt-2">
      <span>Select one rarity to pause the reconfiguration: </span>
      <div class="flex gap-2 mt-1">
        <Tag
          v-for="(rarity, index) in rarities"
          class="cursor-pointer"
          :rarity-label="rarity.label"
          :rarity-tag-color="setTagColor(rarity, index)"
          @click="onRaritySelect(rarity.id)"
        />
        <button
          class="cursor-pointer bg-lime-600 rounded px-1 text-white hover:bg-lime-700"
          @click="onResetRaritiesSelected"
        >
          X
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { rarities } from "../consts/rarities";
import Tag from "./power/tag.vue";

const model = defineModel();
const raritiesSelected = defineModel("raritiesSelected");

const emit = defineEmits([
  "rarity-select",
  "reset-rarities-selected",
  "change-stage",
]);

const setTagColor = (rarity, index) => {
  if (raritiesSelected.value.some((rarity) => rarity === index)) {
    return rarity.tagColor;
  }

  return "#000";
};

const onRaritySelect = (id) => {
  raritiesSelected.value = Object.entries(rarities)
    .filter(([key, value]) => id <= value.id)
    .map(([key]) => key);
};

const onResetRaritiesSelected = () => {
  raritiesSelected.value = [];
};
</script>
