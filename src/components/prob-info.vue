<template>
  <div class="h-fit fixed top-1 left-1">
    <button
      v-if="!showMenu"
      class="bg-gray-600 text-white text-xs rounded-xl w-fit h-7 px-2 text-center align-middle cursor-pointer"
      @click="showMenu = true"
    >
      <span>Prob info</span>
    </button>
    <div v-else class="text-xs text-white bg-gray-600 rounded-xl p-2">
      <button class="cursor-pointer" @click="showMenu = false">
        <span>X</span>
      </button>
      <div>
        <table>
          <caption class="mb-2">
            Prob. info.
          </caption>
          <thead>
            <tr>
              <th class="w-fit p-1 bg-black rounded-tl" scope="col">Level</th>
              <th
                v-for="(rarity, index) in abilityLevelPercentage[0].rarities"
                :key="index"
                class="w-fit px-4 place-items-center bg-black last:rounded-tr"
                scope="col"
              >
                <Tag
                  only-initial-letter
                  :rarity-label="rarity.label"
                  :rarity-tag-color="rarities[rarity.key].tagColor"
                />
              </th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(ability, index) in abilityLevelPercentage"
              :key="index"
              class="even:bg-gray-800"
              :class="{
                'bg-lime-600! font-bold ': ability.level === abilityLevel,
              }"
            >
              <th scope="row" class="">
                {{ ability.level }}
              </th>
              <td
                v-for="(rarity, index) in ability.rarities"
                :key="index"
                class="h-6"
                :class="{ 'bg-red-700': rarity.chance === 0 }"
              >
                {{ rarity.chance }}%
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { abilityLevelPercentage } from "../consts/ability-level";
import { rarities } from "../consts/rarities";
import Tag from "./tag.vue";

const showMenu = ref(false);

const props = defineProps({
  abilityLevel: Number,
});
</script>

<style lang="css">
td {
  @apply text-center;
}
</style>
