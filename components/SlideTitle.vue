<template>
  <h1
    class="title-text font-bold"
    :style="{ fontSize: ($slidev.themeConfigs.titleSize || '3rem') + ' !important' }"
  >
    {{ $slidev.configs.title }}
  </h1>
  <div class="flex mt-12">
    <div class="justify-center">
      <p>
        {{ $slidev.themeConfigs.company }}
      </p>
      <p
        class="author-text"
        :style="{ fontSize: ($slidev.themeConfigs.authorSize || '2.5rem') + ' !important' }"
      >
        {{ $slidev.themeConfigs.author }}
      </p>
    </div>
    <img
      class="ml-auto mr-8 items-center my-auto"
      :style="{ width: $slidev.themeConfigs.logoWidth || '150px' }"
      :src="$slidev.themeConfigs.logo"
    />
  </div>
  <RenderWhen context="print">
    <SlideContainer
      style="position: fixed; top: 0; left: 0; overflow: visible"
    />
  </RenderWhen>
</template>
<script setup lang="ts">
import { useSlideContext } from "@slidev/client";
import SlideContainer from "./SlideContainer.vue";

const { $slidev } = useSlideContext();
</script>
<style scoped>
.title-text {
  white-space: pre-wrap;
}
.author-text {
  opacity: 0.8 !important;
  white-space: pre-wrap;
  line-height: 100% !important;
}
</style>
