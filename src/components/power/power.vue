<template>
  <div
    class="rounded-xl bg-black flex items-center justify-between text-xs text-white px-2 h-8"
    :style="{ backgroundColor: powerRandom.rarity.bgColor }"
  >
    <div class="flex gap-1">
      <Tag
        :rarity-label="powerRandom.rarity.label"
        :rarity-tag-color="powerRandom.rarity.tagColor"
      />
      <Stats :power="powerRandom"></Stats>
    </div>
    <div class="flex items-center">
      <span class="text-[#4cc5c7] mr-2">
        {{ powerValueLabel }}
      </span>
      <img
        v-if="!hidePadlock"
        class="w-5 cursor-pointer"
        :src="togglePadlock"
        @click="toggleBlock"
      />
    </div>
  </div>
</template>

<script setup>
import Tag from "./tag.vue";
import Stats from "./stats.vue";
import PadlockLocked from "../../assets/padlock-locked.svg";
import PadlockUnlocked from "../../assets/padlock-unlocked.svg";
import { computed } from "vue";

const props = defineProps({
  powerRandom: Object,
  hidePadlock: Boolean,
});

const powerValue = computed(() => {
  return new Intl.NumberFormat("pt-BR").format(getRandomPowerValue());
});

const togglePadlock = computed(() => {
  if (props.powerRandom) {
    return props.powerRandom.isBlocked ? PadlockLocked : PadlockUnlocked;
  }
});

const powerValueLabel = computed(() => {
  if (!props.powerRandom.flat) {
    return `${props.powerRandom.value.roll}%`;
  } else {
    return props.powerRandom.value.roll;
  }
});

const toggleBlock = () => {
  if (props.powerRandom) {
    props.powerRandom.isBlocked = !props.powerRandom.isBlocked;
  }
};
</script>
