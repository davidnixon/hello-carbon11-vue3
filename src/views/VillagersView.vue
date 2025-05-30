<script setup>
import { onMounted, ref, nextTick } from "vue";
import { groupBy } from "lodash";
import { useTranslation } from "i18next-vue";
import {
  Sprout20 as NatureIcon,
  Music20 as MusicIcon,
  Education20 as EducationIcon,
  CameraAction20 as FashionIcon,
  Soccer20 as FitnessIcon,
  Basketball20 as PlayIcon,
} from "@carbon/icons-vue";
import { useVillagersStore } from "@/stores/villagers";
import VillagerHobby from "@/components/Villager/Hobby.vue";
import "@carbon/web-components/es/components/content-switcher/index.js";
import "@carbon/web-components/es/components/inline-loading/index.js";

const { t } = useTranslation();
const villagerStore = useVillagersStore();
const loading = ref(false);
const villagerHobbies = ref({});
const selected = ref("switcher-Education");
/**
 * Bug data
 * @typedef {Object} HobbyistData
 * @property {string} hobby
 * @property {Array<VillagerData>} villagers
 */
onMounted(() => {
  loading.value = true;
  try {
    villagerStore.loadVillagers().finally(() => {
      const groups = groupBy(villagerStore.villagers, "hobby");
      const keys = Object.keys(groups);
      villagerHobbies.value = keys.map((key) => {
        return { hobby: key, villagers: groups[key] };
      });

      // What's happening here? I don't want to default always to the first set of content, "Education".
      // So instead we set it based on what minute of the hour it is when we load. So for example
      // if the page is loaded between 15:30 and 15:40 the 4th content, "Nature", is shown.
      const minute = new Date().getMinutes();
      let which = 0;
      if (minute > 9) which = 1;
      if (minute > 19) which = 2;
      if (minute > 29) which = 3;
      if (minute > 39) which = 4;
      if (minute > 49) which = 5;

      // todo: I think this is a bug in content switcher
      // workaround:
      // 1. after loading of the data in complete, add the sections to the DOM that content switcher will control
      // 2. after the sections are available, add the content switcher to the DOM
      // 3. after the content switcher is available set the selected content
      nextTick(() => {
        loading.value = false;
        nextTick(() => {
          contentSwitcherReady.value = true;
          nextTick(() => selected.value = `switcher-${villagerHobbies.value[which].hobby}`);
        });
      });
    });
  }
  catch (e) {
    console.error("error loading bugs from API", e.message);
  }
});

/**
 * Get an icon for the given hobby
 * @param {string} hobby
 * @returns {Object} icon
 */
function hobbyIcon(hobby) {
  switch (hobby) {
    case "Education":
      return EducationIcon;
    case "Fashion":
      return FashionIcon;
    case "Fitness":
      return FitnessIcon;
    case "Music":
      return MusicIcon;
    case "Nature":
      return NatureIcon;
    case "Play":
      return PlayIcon;
    default:
      return NatureIcon;
  }
}

/**
 * Keep track of what is selected
 * @param {UIEvent} evt
 */
function onSelected(evt) {
  selected.value = evt.target?.value;
}

const contentSwitcherReady = ref(false);
</script>

<template>
  <div class="w-full pl-14 mt-12">
    <div class="text-xl! text-center mb-8">
      {{ t("villagers") }}
    </div>
    <div>
      <cds-inline-loading
        v-if="loading"
        status="active"
      />
      <div v-if="contentSwitcherReady">
        <cds-content-switcher
          @cds-content-switcher-selected="onSelected"
        >
          <cds-content-switcher-item
            v-for="group in villagerHobbies"
            :key="`switcher-${group.hobby}`"
            :target="`switcher-${group.hobby}`"
            :value="`switcher-${group.hobby}`"
            :selected="selected === `switcher-${group.hobby}`"
            class="w-full relative"
          >
            <span slot="tooltip-content"> {{ group.hobby }}</span>
            <div class="flex flex-row gap-4">
              <component :is="hobbyIcon(group.hobby)" />
              <div>{{ t(group.hobby) }}</div>
            </div>
          </cds-content-switcher-item>
        </cds-content-switcher>
      </div>

      <section v-if="!loading">
        <div
          v-for="group in villagerHobbies"
          :id="`switcher-${group.hobby}`"
          :key="`content-${group.hobby}`"
        >
          <VillagerHobby :hobbyists="group" />
        </div>
      </section>
    </div>
  </div>
</template>

<style scoped lang="scss">
.special-icon {
  position: relative;
  &--play {
    left: 16px;
    bottom: -2px;
  }
  &--grow {
    bottom: 0;
  }
  &--run,
  &--flash {
    left: 16px;
    bottom: 9px;
  }
  &--reading,
  &--music {
    left: 0;
    top: 0;
    height: 100%;
  }
}
</style>
