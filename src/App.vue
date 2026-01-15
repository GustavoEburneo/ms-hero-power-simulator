<template>
  <main class="bg-black flex items-center justify-center relative">
    <video
      src="https://mapleable-vod.dn.nexoncdn.co.kr/videos/intro_bg.mp4"
      autoplay
      loop
      muted
      playsinline
      preload="auto"
      class="relative pointer-events-none w-full h-screen object-cover opacity-65"
    />

    <div class="absolute">
      <div class="w-100 bg-gray-600 rounded-xl p-3 text-[#BED844] font-bold">
        <p class="mb-2 pb-2 border-b border-white">ABILITY</p>

        <Menu v-model="stage" v-model:rarities-selected="raritiesSelected" />

        <div class="flex flex-col gap-1 mt-3">
          <Power
            v-for="(rarity, index) in raritiesRandom"
            :key="index"
            :rarity="rarity"
            :power-random="powerRandom[index]"
          />

          <UnlockLine :rarities-random="raritiesRandom" />
        </div>
        <MenuBottom
          v-model="abilityLevel"
          :blocked-count="blockedCount"
          :stage="stage"
          @stop-hold="stopHold"
          @start-hold="startHold"
        />
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
      </div>
      <ProbInfo :ability-level="abilityLevel" />
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
import { computed, onMounted, ref, watch, onBeforeUnmount } from "vue";
import { rarities } from "./consts/rarities.js";
import { abilityLevelPercentage } from "./consts/ability-level.js";
import { statsByRarity } from "./consts/power.js";
import { reconfigCost } from "./consts/reconfig-cost.js";
import {
  BaseModal,
  Footer,
  Menu,
  MenuBottom,
  ProbInfo,
  Power,
  UnlockLine,
} from "./components";

const abilityLevel = ref(20);
const raritiesRandom = ref([]);
const stage = ref(7);
const powerRandom = ref([]);
const raritiesSelected = ref([]);
const powersBlockedBySelected = ref([]);

let totalHonorSpent = ref(0);
let timer = null;

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

const hasRaritySelectedInPower = (power, ignorePause) => {
  return (
    raritiesSelected.value.some((rarity) => rarity === power.rarity.key) &&
    !power.isBlocked &&
    !ignorePause
  );
};

const onReconfigure = (ignorePause) => {
  getRandomRarity(ignorePause);
};

const onCloseModal = () => {
  powersBlockedBySelected.value = [];
};

const getRandomRarity = (ignorePause = false) => {
  powersBlockedBySelected.value = [];

  collectPowersBlockedBySelectedRarity(ignorePause);

  if (powersBlockedBySelected.value.length > 0) {
    return;
  }

  const levelConfig = findAbilityLevel();

  for (let i = 0; i < stage.value - 1; i++) {
    if (powerRandom.value[i]?.isBlocked) {
      continue;
    }

    const selectedRarity = pickRarityForRoll(levelConfig);

    if (!selectedRarity) {
      continue;
    }

    raritiesRandom.value[i] = selectedRarity;

    const powers = getPowersByRarity(selectedRarity).map((stat) => ({
      id: i,
      key: stat.key,
      label: stat.label,
      valueType: stat.valueType,
      value: stat.values[selectedRarity.key],
      isBlocked: powerRandom.value[i]?.isBlocked ?? false,
      rarity: selectedRarity,
    }));

    powerRandom.value[i] = powers[getRandomPowerIndex(powers.length)];
  }

  sumTotalHonorSpent();
};

const findAbilityLevel = () => {
  return abilityLevelPercentage.find(
    (ability) => ability.level === abilityLevel.value
  );
};

const getRandomPowerIndex = (powersLength) => {
  return Math.floor(Math.random() * powersLength);
};

const sumTotalHonorSpent = () => {
  totalHonorSpent.value =
    totalHonorSpent.value +
    reconfigCost[abilityLevel.value - 1][blockedCount.value];
};

const getPowersByRarity = (selectedRarity) => {
  return statsByRarity.filter((stat) => stat.values[selectedRarity.key]);
};

const collectPowersBlockedBySelectedRarity = (ignorePause) => {
  for (const power of powerRandom.value) {
    if (hasRaritySelectedInPower(power, ignorePause)) {
      powersBlockedBySelected.value.push(power);
    }
  }
};

const pickRarityForRoll = (levelConfig) => {
  const roll = Math.random() * 100;
  let acc = 0;

  for (const rarity of levelConfig.rarities) {
    acc += rarity.chance;

    if (roll < acc) {
      return {
        ...rarities[rarity.key],
        key: rarity.key,
      };
    }
  }

  return null;
};

const resetTotalHonorSpent = () => {
  totalHonorSpent.value = 0;
};

function stopHold() {
  if (!timer) return;
  clearInterval(timer);
  timer = null;
  window.removeEventListener("mouseup", stopHold);
}

function startHold() {
  if (timer) return;
  getRandomRarity();
  timer = setInterval(getRandomRarity, 300);
  window.addEventListener("mouseup", stopHold);
}

onBeforeUnmount(stopHold);
</script>
