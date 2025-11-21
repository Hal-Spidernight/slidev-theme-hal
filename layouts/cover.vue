<template>
  <div class="slidev-layout cover">
    <div class="my-auto w-full">
      <SlideTitle />
    </div>
  </div>
  <RenderWhen context="print">
    <SlideContainer
      style="position: fixed; top: 0; left: 0; overflow: visible"
    />
  </RenderWhen>
  <Teleport to="#page-root">
    <SlideContainer />
  </Teleport>
</template>

<script setup lang="ts">
import SlideContainer from "../components/SlideContainer.vue";
import SlideTitle from "../components/SlideTitle.vue";
</script>
