<template>
  <main>
    <mobile-menu v-if="isMobile && !authorizationNeeded"></mobile-menu>

    <splitpanes @resized="onResized($event)">
      <pane min-size="10" :size="menuWidth" v-if="!authorizationNeeded && !isMobile && !collapseNav">
        <side-menu @search="showFuzzySearch"></side-menu>
      </pane>
      <pane min-size="10">
        <splitpanes>
          <pane class="has-min-height router-view">
            <router-view></router-view>
          </pane>
          <template v-if="!isMobile">
            <pane v-for="other in activeContainers" :key="other.id">
              <log-container
                :id="other.id"
                show-title
                scrollable
                closable
                @close="containerStore.removeActiveContainer(other)"
              ></log-container>
            </pane>
          </template>
        </splitpanes>
      </pane>
    </splitpanes>
    <button
      @click="collapseNav = !collapseNav"
      class="button is-rounded"
      :class="{ collapsed: collapseNav }"
      id="hide-nav"
      v-if="!isMobile && !authorizationNeeded"
    >
      <span class="icon ml-2" v-if="collapseNav">
        <mdi-light-chevron-right />
      </span>
      <span class="icon" v-else>
        <mdi-light-chevron-left />
      </span>
    </button>
  </main>
</template>

<script lang="ts" setup>
import { Splitpanes, Pane } from "splitpanes";
import { ref, onMounted, watchEffect } from "vue";
import { storeToRefs } from "pinia";
import { useProgrammatic } from "@oruga-ui/oruga-next";
import hotkeys from "hotkeys-js";

import { setTitle } from "@/composables/title";
import { isMobile } from "@/composables/media";
import { smallerScrollbars, lightTheme, menuWidth } from "@/composables/settings";
import { useContainerStore } from "@/stores/container";
import config from "@/stores/config";

import FuzzySearchModal from "@/components/FuzzySearchModal.vue";
import LogContainer from "@/components/LogContainer.vue";
import SideMenu from "@/components/SideMenu.vue";
import MobileMenu from "@/components/MobileMenu.vue";

const collapseNav = ref(false);
const { oruga } = useProgrammatic();
const { authorizationNeeded } = config;

const containerStore = useContainerStore();

const { activeContainers, visibleContainers } = storeToRefs(containerStore);

onMounted(() => {
  if (smallerScrollbars.value) {
    document.documentElement.classList.add("has-custom-scrollbars");
  }
  switch (lightTheme.value) {
    case "dark":
      document.documentElement.setAttribute("data-theme", "dark");
      break;
    case "light":
      document.documentElement.setAttribute("data-theme", "light");
      break;
    default:
      document.documentElement.removeAttribute("data-theme");
  }

  hotkeys("command+k, ctrl+k", (event, handler) => {
    event.preventDefault();
    showFuzzySearch();
  });
});

watchEffect(() => {
  setTitle(`${visibleContainers.value.length} containers`);
});

watchEffect(() => {
  if (smallerScrollbars.value) {
    document.documentElement.classList.add("has-custom-scrollbars");
  } else {
    document.documentElement.classList.remove("has-custom-scrollbars");
  }

  switch (lightTheme.value) {
    case "dark":
      document.documentElement.setAttribute("data-theme", "dark");
      break;
    case "light":
      document.documentElement.setAttribute("data-theme", "light");
      break;
    default:
      document.documentElement.removeAttribute("data-theme");
  }
});

function showFuzzySearch() {
  oruga.modal.open({
    // parent: this,
    component: FuzzySearchModal,
    animation: "false",
    width: 600,
    active: true,
  });
}
function onResized(e) {
  if (e.length == 2) {
    menuWidth.value = e[0].size;
  }
}
</script>

<style scoped lang="scss">
:deep(.splitpanes--vertical > .splitpanes__splitter) {
  min-width: 3px;
  background: var(--border-color);
  &:hover {
    background: var(--border-hover-color);
  }
}

@media screen and (max-width: 768px) {
  .router-view {
    padding-top: 75px;
  }
}

.button.has-no-border {
  border-color: transparent !important;
}

.has-min-height {
  min-height: 100vh;
}

#hide-nav {
  position: fixed;
  left: 10px;
  bottom: 10px;
  &.collapsed {
    left: -40px;
    width: 60px;
    padding-left: 40px;
    background: rgba(0, 0, 0, 0.95);

    &:hover {
      left: -25px;
    }
  }
}
</style>
