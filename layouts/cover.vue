<template>
  <div class="slidev-layout cover">
    <div class="my-auto w-full">
      <slot />
    </div>
  </div>
  <SlideContainer />
</template>

<script setup lang="ts">
import SlideContainer from "../components/SlideContainer.vue";
</script>
