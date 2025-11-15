<template>
  <Teleport to="#page-root">
    <Transition name="shape-fade">
      <div
        id="custom-background"
        class="transition-background"
        :key="currentNavState"
        :style="cssVars"
      >
        <slot />
      </div>
    </Transition>
  </Teleport>
</template>
<script setup lang="ts">
import { ref, computed, watch } from "vue";
import { useSlideContext } from "@slidev/client";

const { $nav, $slidev } = useSlideContext();

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

const shapeHueColor = computed(() => $slidev.themeConfigs.hueColor ?? 0);

// カラーバイアスを適用したHue計算
const shapeHue = computed(() => {
  // colorBias が文字列（カラー値）の場合はそのまま返す
  if (typeof shapeHueColor.value === "string") {
    return shapeHueColor.value;
  }
  // colorBias が数値（Hueオフセット）の場合は計算
  const baseHue = seedNormalized.value * 360;
  return (baseHue + shapeHueColor.value) % 360;
});

// ランダム値に基づいて形状タイプを決定
const shapeType = computed(() => {
  const type = Math.floor(seedNormalized.value * 5);
  return type; // 0-4: circle, ellipse, blob, star, wavy
});

// 形状タイプに基づいて背景イメージを生成
// Provide a CSS-friendly color string; if shapeHue is numeric produce an HSL color, otherwise pass-through string
const shapeColorCss = computed(() => {
  const h = shapeHue.value;
  if (typeof h === "string") return h;
  return `hsl(${h} 65% 55%)`;
});

const shapeGradient = computed(() => {
  const shapeT = shapeType.value;

  // Use CSS variables for position and color so pseudo elements pick them up
  // color is provided through --shape-color (a full color string)
  if (shapeT === 0) {
    return `radial-gradient(circle at var(--shape-pos-x) var(--shape-pos-y), color-mix(in srgb, var(--shape-color) 50%, transparent) 0%, transparent 40%)`;
  }
  if (shapeT === 1) {
    return `radial-gradient(ellipse var(--shape-size) var(--shape-size) at var(--shape-pos-x) var(--shape-pos-y), color-mix(in srgb, var(--shape-color) 50%, transparent) 0%, transparent 40%)`;
  }
  if (shapeT === 2) {
    return `radial-gradient(ellipse var(--shape-size) var(--shape-size) at var(--shape-pos-x) var(--shape-pos-y), color-mix(in srgb, var(--shape-color) 50%, transparent) 0%, transparent 40%)`;
  }
  // fallback / blob-like
  return `radial-gradient(circle at var(--shape-pos-x) var(--shape-pos-y), color-mix(in srgb, var(--shape-color) 50%, transparent) 0%, transparent 40%)`;
});

// computed object of CSS variables to attach to the wrapper element via :style
const cssVars = computed(() => {
  // derive a second positional axis to vary Y differently from X
  const posYNormalized = (seedNormalized.value * 0.73) % 1;
  // small offsets for transforms to make motion less uniform
  const offsetX = `${(seedNormalized.value - 0.5) * 20}%`;
  const offsetY = `${(posYNormalized - 0.5) * 18}%`;
  const scale = `${1 + seedNormalized.value * 0.45}`;
  const blurPx = `${4 + seedNormalized.value * 12}px`;

  return {
    // gradient for ::before
    "--shape-gradient": shapeGradient.value,
    // a plain color (no alpha) usable with color-mix
    "--shape-color": shapeColorCss.value,
    // position and size (computed strings so CSS calc can evaluate)
    "--shape-pos-x": `calc(25% + ${seedNormalized.value} * 50%)`,
    "--shape-pos-y": `calc(20% + ${posYNormalized} * 60%)`,
    "--shape-size": `calc(30% + ${seedNormalized.value} * 20%)`,
    // type as number for calc-based rotation
    "--shape-type": `${shapeType.value}`,
    // small transform offsets, scale and blur for softer edges
    "--shape-offset-x": offsetX,
    "--shape-offset-y": offsetY,
    "--shape-scale": scale,
    "--shape-blur": blurPx,
  };
});
</script>
<style scoped>
.transition-background::before {
  background-image: var(--shape-gradient);
}

.transition-background {
  position: relative;
  overflow: hidden;
  /* fallbacks (overridden by inline --shape-pos-x/--shape-size from :style) */
  --shape-pos-x: 50%;
  --shape-size: 30%;
  background-color: var(--color-bg, transparent);
  width: 140%;
  height: 140%;
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
  transform: translate3d(var(--shape-offset-x), var(--shape-offset-y), 0) scale(var(--shape-scale))
    rotate(calc(var(--shape-type) * 15deg));
  transition:
    transform 1.5s ease,
    opacity 1.5s ease;
  filter: blur(var(--shape-blur));
  background-repeat: no-repeat;
  background-position: center;
  background-size: 200% 200%;
  will-change: transform, opacity;
}

.transition-background::after {
  /* a soft complementary glow using the same --shape-color via color-mix to add transparency */
  background-image: radial-gradient(
    circle at calc(100% - var(--shape-pos-x)) calc(100% - var(--shape-pos-y)),
    color-mix(in srgb, var(--shape-color) 15%, transparent) 0%,
    transparent 45%
  );
  transform: translate3d(calc(var(--shape-offset-x) * -0.6), calc(var(--shape-offset-y) * -0.6), 0) scale(calc(var(--shape-scale) * 1.15)) rotate(8deg);
  transition:
    transform 1.5s ease,
    opacity 1.5s ease;
  filter: blur(calc(var(--shape-blur) * 2));
  background-repeat: no-repeat;
  background-size: 220% 220%;
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
