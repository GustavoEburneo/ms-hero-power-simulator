<template>
  <Teleport to="body">
    <div
      class="bg-[#000000aa] fixed inset-0 flex justify-center items-center w-full h-full"
    >
      <div
        class="bg-gray-600 flex flex-col justify-center rounded-xl p-1 pt-0 w-92 text-xs"
      >
        <div class="flex justify-between text-white font-bold p-1">
          <div></div>
          <span>OK</span>
          <button class="cursor-pointer" @click="onClose">X</button>
        </div>
        <div class="bg-white">
          <div
            class="p-2 flex flex-col justify-center items-center text-center font-bold text-gray-600"
          >
            <p class="mb-2">
              You have an option of a high rank obtainable at your current
              Ability Level.
            </p>
            <p>Reconfigure anyway?</p>
          </div>
          <div class="bg-gray-600 h-24 p-2 m-1 rounded">
            <Power
              v-for="(power, index) in powers"
              class="mb-1"
              hide-padlock
              :key="index"
              :power-random="power"
            />
          </div>
          <div
            class="flex justify-center items-center gap-8 text-white font-bold p-3"
          >
            <button
              class="bg-blue-400 p-2 rounded-xl w-24 border border-black cursor-pointer"
              @click="onClose"
            >
              Cancel
            </button>
            <button
              class="bg-lime-600 p-2 rounded-xl w-24 border border-black px-2 cursor-pointer"
              @click="onReconfigure"
            >
              Reconfigure
            </button>
          </div>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import Power from "./power.vue";

const emit = defineEmits(["close", "reconfigure"]);

const props = defineProps({
  powers: Array,
  show: Boolean,
});

const onClose = () => {
  emit("close");
};

const onReconfigure = () => {
  emit("reconfigure", true);
  emit("close");
};
</script>
