<template>
  <div class="slidev-layout cover">
    <div class="my-auto w-full">
      <SlideTitle />
    </div>
  </div>
  <SlideContainer />
</template>

<script setup lang="ts">
import SlideContainer from "../components/SlideContainer.vue";
import SlideTitle from "../components/SlideTitle.vue";
</script>
