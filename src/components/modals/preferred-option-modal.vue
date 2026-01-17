<template>
  <Teleport to="body">
    <div
      class="bg-[#000000aa] fixed inset-0 flex justify-center items-center w-full"
    >
      <div
        class="bg-gray-600 flex flex-col justify-center rounded-xl p-1 pt-0 w-180 text-xs"
      >
        <div class="flex justify-between text-white font-bold p-1">
          <div></div>

          <span>Select Option</span>

          <button class="cursor-pointer" @click="onClosePreferredOptionModal">
            X
          </button>
        </div>

        <div class="bg-white max-h-[60vh] overflow-y-auto pb-3">
          <div
            class="p-2 flex flex-col justify-center items-center text-center font-bold text-gray-600"
          >
            <p class="mb-2">
              When a selected option appears, the reconfigure button will stop,
              and a popup will appear where you can choose whether to
              reconfigure.
            </p>
          </div>

          <div class="flex justify-center">
            <table
              class="border-separate p-2 bg-gray-300 border-spacing-1 border-spacing-x-0 rounded"
            >
              <thead>
                <tr class="bg-gray-800 text-white">
                  <th class="rounded-l-lg pl-2 w-fit">Options</th>
                  <th
                    v-for="(rarity, rarityKey) in rarities"
                    :key="rarityKey"
                    class="py-2 mx-2 last:rounded-r-lg w-22"
                  >
                    <div class="flex items-center gap-1 mr-2">
                      <input
                        type="checkbox"
                        :checked="isColumnSelected(rarityKey)"
                        :id="`rarity-${rarityKey}`"
                        @change="
                          onToggleColumn(rarityKey, $event.target.checked)
                        "
                      />
                      <label
                        class="text-left"
                        :for="`rarity-${rarityKey}`"
                        :style="{
                          color: rarity.tagColor,
                        }"
                      >
                        {{ rarity.label }}
                      </label>
                    </div>
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="(stat, index) in statsByRarity"
                  :key="index"
                  class="rounded bg-white"
                >
                  <th class="text-left text-[10px] p-2 bg-white">
                    {{ stat.label }}
                  </th>
                  <td v-for="(value, index) in stat.values" :key="index">
                    <div class="flex gap-1">
                      <input
                        v-if="!value.void"
                        v-model="selected"
                        type="checkbox"
                        :id="`stat-${stat.key}-${index}`"
                        :name="`stat-${stat.key}-${index}`"
                        :value="{ stat: stat.key, index, value }"
                      />
                      <label
                        class="text-[8px] text-left"
                        :for="`stat-${stat.key}-${index}`"
                      >
                        {{ minMaxValue(value, stat.valueType) }}
                      </label>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
        <div
          class="flex justify-between items-center gap-8 text-white font-bold p-3 px-8 bg-white border-t-2 border-gray-400"
        >
          <button
            class="bg-blue-400 p-2 rounded-xl w-24 border border-gray-500 cursor-pointer"
            @click="onClickDeselectAll"
          >
            Deselect All
          </button>

          <div class="flex gap-2">
            <button
              class="bg-lime-600 p-2 rounded-xl w-24 border border-gray-500 px-2 cursor-pointer hover:bg-lime-700"
              @click="onClickSaveOption"
            >
              Save Option
            </button>
          </div>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { ref } from "vue";
import { statsByRarity } from "../../consts/power";
import { rarities } from "../../consts/rarities";

const loadSelected = () => {
  const preferredOptionsStorage = JSON.parse(
    localStorage.getItem("preferredOptions"),
  );

  if (preferredOptionsStorage.length > 0) {
    return preferredOptionsStorage;
  } else {
    return [];
  }
};

const selected = ref(loadSelected());
const showPreferredOptionModal = defineModel("showPreferredOptionModal");

const isSelectedValue = (statKey, rarityKey) => {
  return selected.value?.some(
    (item) => item.stat === statKey && item.index === rarityKey,
  );
};

const selectColumn = (rarityKey) => {
  statsByRarity.forEach((stat) => {
    const value = stat.values[rarityKey];
    if (!value || value.void) return;
    if (isSelectedValue(stat.key, rarityKey)) return;
    selected.value.push({ stat: stat.key, index: rarityKey, value });
  });
};

const deselectColumn = (rarityKey) => {
  selected.value = selected.value.filter((item) => item.index !== rarityKey);
};

const isColumnSelected = (rarityKey) => {
  return statsByRarity.every((stat) => {
    const value = stat.values[rarityKey];
    if (!value || value.void) return true;
    return isSelectedValue(stat.key, rarityKey);
  });
};

const onToggleColumn = (rarityKey, checked) => {
  if (checked) {
    selectColumn(rarityKey);
    return;
  }

  deselectColumn(rarityKey);
};

const onClosePreferredOptionModal = () => {
  showPreferredOptionModal.value = false;
};

const minMaxValue = (value, valueType) => {
  if (value.void) return value.void;

  const min = value.min;
  const max = value.max;
  let valueTypeString = "";

  if (!isValueTypeFlat(valueType)) {
    valueTypeString = "%";
  }

  return `(${min}${valueTypeString} ~ ${max}${valueTypeString})`;
};

const isValueTypeFlat = (valueType) => {
  return valueType === "FLAT";
};

const onClickDeselectAll = () => {
  selected.value = [];
};

const onClickSaveOption = () => {
  localStorage.setItem(
    "preferredOptions",
    JSON.stringify(selected.value || []),
  );
  onClosePreferredOptionModal();
};
</script>
