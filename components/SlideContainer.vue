<template>
  <Teleport to="#page-root">
    <Transition name="shape-fade">
      <div
        id="custom-background"
        class="transition-background"
        :style="{ '--shape-seed': seedNormalized }"
        :key="currentNavState"
      >
        <slot />
      </div>
    </Transition>
  </Teleport>
</template>
<script setup lang="ts">
import { ref, computed, watch } from "vue";
import { useSlideContext } from "@slidev/client";

const { $nav } = useSlideContext();

const currentNavState = computed(() => {
  return $nav.value.currentPage;
});

/**
 * 背景用のランダムシェイプのシード値
 */
const backGroundShape = ref($nav.value.currentPage * Math.random());

watch(currentNavState, (newVal) => {
  backGroundShape.value = newVal * Math.random();
});

// 正規化した 0..1 のシード値を CSS に渡す
const seedNormalized = computed(() => {
  const v = Number(backGroundShape.value) || 0;
  return v - Math.floor(v);
});
</script>
<style scoped>
.transition-background {
  position: relative;
  overflow: hidden;
  /* basic CSS variables derived from the seed (0..1) */
  --shape-hue: calc(var(--shape-seed) * 360);
  --shape-pos-x: calc(25% + var(--shape-seed) * 50%);
  --shape-size: calc(30% + var(--shape-seed) * 50%);
  background-color: var(--color-bg, transparent);
  width: 150%;
  height: 150%;
}

/* floating soft shapes using pseudo elements */
.transition-background::before,
.transition-background::after {
  content: "";
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  mix-blend-mode: screen;
}

.transition-background::before {
  background-image: radial-gradient(
    circle at var(--shape-pos-x) 30%,
    hsl(var(--shape-hue) 70% 60% / 0.18) 0%,
    transparent 40%
  );
  transform: translate3d(-10%, -5%, 0) scale(1.1);
  transition:
    transform 1.5s ease,
    opacity 1.5s ease;
}

.transition-background::after {
  background-image: radial-gradient(
    circle at calc(100% - var(--shape-pos-x)) 80%,
    hsl(calc(var(--shape-hue) + 60) 65% 55% / 0.12) 0%,
    transparent 45%
  );
  transform: translate3d(8%, 12%, 0) scale(1.15) rotate(8deg);
  transition:
    transform 1.5s ease,
    opacity 1.5s ease;
}

/* ensure content sits above the background shapes */
.transition-background > * {
  position: relative;
  z-index: 1;
}

/* アニメーション中の状態 */
.shape-fade-enter-active,
.shape-fade-leave-active {
  /* opacityと、もしあれば transform をトランジションの対象に */
  transition:
    opacity 2s ease-in-out,
    transform 2s ease-in-out;
}

/* Enter (新しい要素が入ってくる) の開始状態 と Leave (古い要素が消える) の終了状態 */
.shape-fade-enter-from,
.shape-fade-leave-to {
  opacity: 0;
}

/* 古い要素と新しい要素が重ならないように、位置を調整（任意） */
.shape-fade-leave-active {
  position: absolute;
  inset: 0;
}
</style>
