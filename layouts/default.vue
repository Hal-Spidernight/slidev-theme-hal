<template>
  <div class="slidev-layout default">
    <div class="my-auto h-full w-full">
      <slot />
    </div>
    <RenderWhen context="print">
      <SlideContainer
        style="position: fixed; top: 0; left: 0; overflow: visible"
      />
    </RenderWhen>
  </div>
</template>
