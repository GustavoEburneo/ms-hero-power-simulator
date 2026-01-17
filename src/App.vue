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

        <Menu v-model="stage" />

        <div class="flex flex-col gap-1 mt-3">
          <Power
            v-for="(rarity, index) in raritiesRandom"
            :key="index"
            :rarity="rarity"
            :power-random="powersRandom[index]"
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
        <Statistic v-model:total-honor-spent="totalHonorSpent" />
      </div>
      <ProbInfo :ability-level="abilityLevel" />
    </div>
  </main>
  <Footer />
  <ReconfigModal
    v-if="powersBlockedBySelected.length > 0"
    :powers="powersBlockedBySelected"
    @close="onCloseModal"
    @reconfigure="onReconfigure"
  />
  <PreferredOptionModal v-if="false" />
</template>

<script setup>
import { computed, onMounted, ref, watch, onBeforeUnmount } from "vue";
import { rarities } from "./consts/rarities.js";
import { abilityLevelPercentage } from "./consts/ability-level.js";
import { statsByRarity } from "./consts/power.js";
import { reconfigCost } from "./consts/reconfig-cost.js";
import {
  ReconfigModal,
  Footer,
  Menu,
  MenuBottom,
  ProbInfo,
  Power,
  UnlockLine,
  Statistic,
  PreferredOptionModal,
} from "./components";

const DELAY = 150;

const abilityLevel = ref(20);
const raritiesRandom = ref([]);
const stage = ref(7);
const powersRandom = ref([]);
const powersBlockedBySelected = ref([]);
const totalHonorSpent = ref(0);

let timer = null;

onMounted(() => {});

const blockedCount = computed(() => {
  return powersRandom.value.filter((power) => {
    return power.isBlocked;
  }).length;
});

watch(stage, (newValue, oldValue) => {
  if (newValue < oldValue) {
    raritiesRandom.value.splice(newValue - 1, oldValue - newValue);
    powersRandom.value.splice(newValue - 1, oldValue - newValue);
  }
});

const hasRaritySelectedInPower = (power, ignorePause) => {
  return !power.isBlocked && !ignorePause;
};

const onReconfigure = (ignorePause) => {
  getRandomRarity(ignorePause);
};

const onCloseModal = () => {
  powersBlockedBySelected.value = [];
};

const getRandomRarity = (ignorePause = false) => {
  const preferredOptionsStorage = getPreferredOptions();

  powersBlockedBySelected.value = [];

  collectPowerInPreferredOptionsStorage(preferredOptionsStorage, ignorePause);

  if (powersBlockedBySelected.value.length > 0) {
    return;
  }

  const levelConfig = findAbilityLevel();

  for (let i = 0; i < stage.value - 1; i++) {
    if (powersRandom.value[i]?.isBlocked) {
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
      isBlocked: powersRandom.value[i]?.isBlocked ?? false,
      rarity: selectedRarity,
    }));

    powersRandom.value[i] = powers[getRandomPowerIndex(powers.length)];
  }

  sumTotalHonorSpent();
};

const getPreferredOptions = () => {
  return JSON.parse(localStorage.getItem("preferredOptions"));
};

const collectPowerInPreferredOptionsStorage = (
  preferredOptionsStorage,
  ignorePause,
) => {
  for (const power of powersRandom.value) {
    const isPreferred = preferredOptionsStorage.some(
      (option) =>
        option.stat === power.key && option.index === power.rarity.key,
    );

    if (isPreferred && hasRaritySelectedInPower(power, ignorePause)) {
      powersBlockedBySelected.value.push(power);
    }
  }
};

const findAbilityLevel = () => {
  return abilityLevelPercentage.find(
    (ability) => ability.level === abilityLevel.value,
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
  return statsByRarity.filter((stat) => !stat.values[selectedRarity.key].void);
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

function stopHold() {
  if (!timer) return;
  clearInterval(timer);
  timer = null;
  window.removeEventListener("mouseup", stopHold);
}

function startHold() {
  if (timer) return;
  getRandomRarity();
  timer = setInterval(getRandomRarity, DELAY);
  window.addEventListener("mouseup", stopHold);
}

onBeforeUnmount(stopHold);
</script>
