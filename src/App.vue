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
    <div class="absolute">
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
              class="bg-white w-8 text-gray-400 text-center"
              v-model="abilityLevel"
            />
          </div>
          <button
            class="bg-lime-500 px-4 py-2 rounded-md cursor-pointer hover:bg-lime-600"
            @mousedown="getRandomRarity()"
          >
            Change Option
          </button>
        </div>
      </div>
      <div class="text-xs p-2 bg-gray-600 text-white rounded-xl mt-2">
        <p>
          Found a bug? Tell me:
          <a
            class="text-blue-400"
            href="https://github.com/GustavoEburneo/ms-hero-power-simulator/issues"
          >
            github issues
          </a>
        </p>
        <p>Created by: Botuca</p>
        <p>Nickname: Aipo</p>
        <p class="mt-2">Notices:</p>
        <p>- I'll add Honor Calc soon.</p>
      </div>
    </div>
  </main>
</template>

<script setup>
import { onMounted, ref, watch } from "vue";
import { rarities } from "./consts/rarities.js";
import Power from "./components/power.vue";
import { abilityLevelPercentage } from "./consts/ability-level.js";
import { statsByRarity } from "./consts/power.js";

const MAX_LINES_LENGTH = 6;

const abilityLevel = ref(20);
const raritiesRandom = ref([]);
const stage = ref(7);
const powerRandom = ref([]);

onMounted(() => {
  getRandomRarity();
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
};
</script>

<style scoped></style>
