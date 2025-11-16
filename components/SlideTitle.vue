<template>
  <h1 class="title-text">
    {{ $slidev.configs.title }}
  </h1>
  <div class="mt-8">
    <p>
      {{ $slidev.themeConfigs.company }}
    </p>
    <p class="author-text">
      {{ $slidev.themeConfigs.author }}
    </p>
  </div>
</template>
<script setup lang="ts">
import { useSlideContext } from "@slidev/client";
const { $slidev } = useSlideContext();
</script>
<style scoped>
.title-text {
  font-size: 3rem !important;
  white-space: pre-wrap;
}
.author-text {
  opacity: 0.8 !important;
  font-size: 2.5rem !important;
  white-space: pre-wrap;
  line-height: 100% !important;
}
</style>
