<template>
  <div class="hero is-halfheight">
    <div class="hero-body">
      <div class="container has-text-centered">
        <h1 class="title">
          Container not found.
          <small class="subtitle">It may have been removed.</small>
        </h1>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { setTitle } from "@/composables/title";

export default {
  name: "ContainerNotFound",
  setup() {
    setTitle("Container not found");
  },
};
</script>
