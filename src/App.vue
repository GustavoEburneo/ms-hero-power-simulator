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
        <div
          class="flex border-b border-white justify-between mb-2 items-center"
        >
          <p class="mb-2 pb-2">ABILITY</p>
          <div>
            <label class="text-[10px] text-white">PRESETS</label>
            <div class="grid grid-cols-5 grid-rows-2 mb-2 gap-1">
              <div
                v-for="i in 10"
                class="flex justify-center items-center gap-1"
              >
                <input
                  v-model="preset"
                  type="radio"
                  name="preset"
                  class="bg-black rounded hover:bg-lime-600 active:bg-red-500"
                  :id="`preset-${i}`"
                  :value="i"
                  @change="onChangePreset($event.target.value)"
                />
                <label :for="`preset-${i}`" class="text-white text-xs">
                  {{ i }}
                </label>
              </div>
            </div>
          </div>
        </div>

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
        {{ powersRandom.value }}
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
const preset = ref(1);
const presets = ref([]);

let timer = null;

const initActivePreset = () => {
  if (!localStorage.getItem("activePreset")) {
    localStorage.setItem("activePreset", 1);
  } else {
    preset.value = localStorage.getItem("activePreset");
  }
};

const initPresets = () => {
  if (!localStorage.getItem("presets")) {
    localStorage.setItem(
      "presets",
      JSON.stringify(Array.from({ length: 10 }, () => [])),
    );
  } else {
    presets.value = localStorage.getItem("presets");
  }
};

const initPowerRandom = () => {
  const presets = JSON.parse(localStorage.getItem("presets"));

  if (presets[preset.value - 1]?.length > 0) {
    powersRandom.value = presets[preset.value - 1];
    console.log(presets[preset.value - 1]);

    for (let i = 0; i < powersRandom.value.length; i++) {
      raritiesRandom.value[i] = presets[preset.value - 1][i].rarity;
    }
  }
};

onMounted(() => {
  initActivePreset();
  initPresets();
  initPowerRandom();
});

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

const onChangePreset = (value) => {
  raritiesRandom.value = [];
  powersRandom.value = [];

  localStorage.setItem("activePreset", value);
  const presets = JSON.parse(localStorage.getItem("presets"));

  for (let i = 0; i < presets[value - 1].length; i++) {
    if (presets[preset.value - 1].length > 0) {
      powersRandom.value[i] = presets[preset.value - 1][i];
      raritiesRandom.value[i] = presets[preset.value - 1][i]?.rarity;
    }
  }
};

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
      flat: stat.flat,
      value: stat.values[selectedRarity.key],
      isBlocked: powersRandom.value[i]?.isBlocked ?? false,
      rarity: selectedRarity,
    }));

    powersRandom.value[i] = powers[getRandomPowerIndex(powers.length)];
    powersRandom.value[i].value.roll = new Intl.NumberFormat("pt-BR").format(
      getRandomPowerValue(powersRandom.value[i]),
    );
  }

  setPowersRandomInPreset();
  sumTotalHonorSpent();
};

const isValueTypeFlat = (powerRandom) => {
  if (powerRandom) {
    return powerRandom.flat;
  }
};

const getRandomPowerValue = (powerRandom) => {
  const min = powerRandom?.value.min;
  const max = powerRandom?.value.max;

  let randomMinMaxPercentage = null;
  let randomMinMaxFlat = null;

  if (isValueTypeFlat(powerRandom)) {
    randomMinMaxFlat = Math.floor(Math.random() * (max - min + 1)) + min;

    return randomMinMaxFlat;
  } else {
    randomMinMaxPercentage =
      Math.round((Math.random() * (max - min) + min) * 10) / 10;

    return randomMinMaxPercentage;
  }
};

const setPowersRandomInPreset = () => {
  let presets = JSON.parse(localStorage.getItem("presets"));
  presets[preset.value - 1] = powersRandom.value;
  localStorage.setItem("presets", JSON.stringify(presets));
};

const getPreferredOptions = () => {
  return JSON.parse(localStorage.getItem("preferredOptions"));
};

const collectPowerInPreferredOptionsStorage = (
  preferredOptionsStorage,
  ignorePause,
) => {
  for (const power of powersRandom.value) {
    const isPreferred = preferredOptionsStorage?.some(
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
