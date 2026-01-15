<template>
  <div>
    <span class="mr-1">{{ power?.label }}</span>
    <span class="text-gray-400 text-[10px]">{{ minMaxValue }}</span>
  </div>
</template>

<script setup>
import { computed, onMounted } from "vue";

const props = defineProps({
  power: Object,
});

const minMaxValue = computed(() => {
  const min = props.power?.value.min;
  const max = props.power?.value.max;
  let valueType = "";

  if (!isValueTypeFlat()) {
    valueType = "%";
  }

  return `(${min}${valueType} ~ ${max}${valueType})`;
});

const isValueTypeFlat = () => {
  return props.power?.valueType === "FLAT";
};

onMounted(() => {});
</script>
