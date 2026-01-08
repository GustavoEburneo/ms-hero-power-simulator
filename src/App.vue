<template>
  <main class="h-screen bg-black flex items-center justify-center">
    <video
      src="https://mapleable-vod.dn.nexoncdn.co.kr/videos/intro_bg.mp4"
      autoplay
      loop
      muted
      playsinline
      preload="auto"
      class="relative pointer-events-none w-full h-full object-cover opacity-65"
    ></video>
    <div class="absolute flex gap-2">
      <div class="w-100 bg-gray-600 rounded-xl p-3 text-[#BED844] font-bold">
        <p class="mb-1">ABILITY</p>
        <span>Stage: </span>
        <input
          type="number"
          min="2"
          max="7"
          class="bg-white w-8 text-gray-400 text-center"
          v-model="stage"
        />
        <div class="flex flex-col gap-1 mt-3">
          <Power
            v-for="(rarity, index) in raritiesRandom"
            :key="rarity.id"
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
              :disabled="blockedCount === 6"
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
            class="px-2 py-1 bg-black rounded cursor-pointer"
            @click="resetTotalHonorSpent"
          >
            reset
          </button>
        </div>
      </div>
      <div class="text-xs p-4 bg-gray-600 text-white rounded-xl">
        <p class="mt-2">Upcoming features:</p>
        <p>- Honor calc</p>
        <p>- Set the rarity that will pause the reroll</p>
        <p>- Hold down the Change option button to reroll</p>
        <p>- Info about the %</p>
      </div>
    </div>
  </main>
  <Footer />
</template>

<script setup>
import { computed, onMounted, ref, watch } from "vue";
import { rarities } from "./consts/rarities.js";
import Power from "./components/power.vue";
import { abilityLevelPercentage } from "./consts/ability-level.js";
import { statsByRarity } from "./consts/power.js";
import { reconfigCost } from "./consts/reconfig-cost.js";
import Footer from "./components/footer.vue";

const MAX_LINES_LENGTH = 6;

const abilityLevel = ref(20);
const raritiesRandom = ref([]);
const stage = ref(7);
const powerRandom = ref([]);
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

const getRandomRarity = () => {
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
</script>

<style scoped></style>
