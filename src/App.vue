<template>
  <main class="h-screen bg-black flex items-center justify-center relative">
    <video
      src="https://mapleable-vod.dn.nexoncdn.co.kr/videos/intro_bg.mp4"
      autoplay
      loop
      muted
      playsinline
      preload="auto"
      class="relative pointer-events-none w-full h-full object-cover opacity-65"
    />
    <div class="absolute flex gap-2">
      <div class="w-100 bg-gray-600 rounded-xl p-3 text-[#BED844] font-bold">
        <p class="mb-2 pb-2 border-b border-white">ABILITY</p>
        <div class="text-xs">
          <div>
            <span>Stage: </span>
            <input
              type="number"
              min="2"
              max="7"
              class="bg-white w-8 text-gray-400 text-center rounded"
              v-model="stage"
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
                @click="onClickRaritySelect(rarity.id)"
              />
              <button
                class="cursor-pointer bg-lime-500 rounded px-1 text-white hover:bg-lime-600"
                @click="resetRaritiesSelected"
              >
                X
              </button>
            </div>
          </div>
        </div>
        <div class="flex flex-col gap-1 mt-3">
          <Power
            v-for="(rarity, index) in raritiesRandom"
            :key="index"
            :rarity="rarity"
            :power-random="powerRandom[index]"
          />
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
        </div>
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
                {{ reconfigCost[abilityLevel - 1][blockedCount] }}
              </span>
            </p>
            <button
              class="bg-lime-500 px-4 py-2 rounded-md cursor-pointer hover:bg-lime-600 disabled:bg-gray-800 disabled:cursor-default"
              :disabled="blockedCount === stage - 1"
              @mousedown="getRandomRarity()"
            >
              Change Option
            </button>
          </div>
        </div>
        <div
          class="text-white text-sm mt-2 pt-2 border-t flex justify-between items-center"
        >
          <div>
            <span>Total honor spent: </span>
            <span class="text-[#BED844]">{{ totalHonorSpent }}</span>
          </div>
          <button
            class="px-2 py-1 rounded cursor-pointer bg-lime-500 text-xs hover:bg-lime-600"
            @click="resetTotalHonorSpent"
          >
            RESET
          </button>
        </div>
      </div>
      <div class="text-xs p-4 bg-gray-600 text-white rounded-xl">
        <p class="mt-2">Upcoming features:</p>
        <p>- Set the rarity that will pause the reroll</p>
        <p>- Hold down the Change option button to reroll</p>
        <p>- Info about the %</p>
      </div>
    </div>
  </main>
  <Footer />
  <BaseModal
    v-if="powersBlockedBySelected.length > 0"
    :powers="powersBlockedBySelected"
    @close="onCloseModal"
    @reconfigure="onReconfigure"
  />
</template>

<script setup>
import { computed, onMounted, ref, watch } from "vue";
import { rarities } from "./consts/rarities.js";
import Power from "./components/power.vue";
import { abilityLevelPercentage } from "./consts/ability-level.js";
import { statsByRarity } from "./consts/power.js";
import { reconfigCost } from "./consts/reconfig-cost.js";
import Footer from "./components/footer.vue";
import Tag from "./components/tag.vue";
import BaseModal from "./components/base-modal.vue";

const MAX_LINES_LENGTH = 6;

const abilityLevel = ref(20);
const raritiesRandom = ref([]);
const stage = ref(7);
const powerRandom = ref([]);
const raritiesSelected = ref([]);
const powersBlockedBySelected = ref([]);

let totalHonorSpent = ref(0);

onMounted(() => {});

const blockedCount = computed(() => {
  return powerRandom.value.filter((power) => {
    return power.isBlocked;
  }).length;
});

watch(stage, (newValue, oldValue) => {
  if (newValue < oldValue) {
    raritiesRandom.value.splice(newValue - 1, oldValue - newValue);
    powerRandom.value.splice(newValue - 1, oldValue - newValue);
  }
});

const onReconfigure = (ignorePause) => {
  getRandomRarity(ignorePause);
};

const onCloseModal = () => {
  powersBlockedBySelected.value = [];
};

const setTagColor = (rarity, index) => {
  if (raritiesSelected.value.some((rarity) => rarity === index)) {
    return rarity.tagColor;
  }

  return "#000";
};

const getRandomRarity = (ignorePause = false) => {
  powersBlockedBySelected.value = [];

  for (const power of powerRandom.value) {
    if (
      raritiesSelected.value.some((rarity) => rarity === power.rarity.key) &&
      !power.isBlocked &&
      !ignorePause
    ) {
      powersBlockedBySelected.value.push(power);
    }
  }

  if (powersBlockedBySelected.value.length > 0) {
    return;
  }

  for (let i = 0; i < stage.value - 1; i++) {
    if (powerRandom.value[i]?.isBlocked) {
      continue;
    }

    const roll = Math.random() * 100;

    const getLevel = abilityLevelPercentage.find(
      (ability) => ability.level === abilityLevel.value
    );

    let acc = 0;

    for (const rarity of getLevel.rarities) {
      acc += rarity.chance;

      if (roll < acc) {
        const selectedRarity = {
          ...rarities[rarity.key],
          key: rarity.key,
        };

        raritiesRandom.value[i] = selectedRarity;

        const powers = statsByRarity
          .filter((stat) => stat.values[selectedRarity.key])
          .map((stat) => ({
            id: i,
            key: stat.key,
            label: stat.label,
            valueType: stat.valueType,
            value: stat.values[selectedRarity.key],
            isBlocked: powerRandom.value[i]?.isBlocked ?? false,
            rarity: selectedRarity,
          }));

        powerRandom.value[i] =
          powers[Math.floor(Math.random() * powers.length)];
        break;
      }
    }
  }

  totalHonorSpent.value =
    totalHonorSpent.value +
    reconfigCost[abilityLevel.value - 1][blockedCount.value];
};

const resetTotalHonorSpent = () => {
  totalHonorSpent.value = 0;
};

const onClickRaritySelect = (id) => {
  raritiesSelected.value = Object.entries(rarities)
    .filter(([key, value]) => id <= value.id)
    .map(([key]) => key);
};

const resetRaritiesSelected = () => {
  raritiesSelected.value = [];
};
</script>

<style scoped></style>
