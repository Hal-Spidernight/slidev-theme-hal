<template>
  <Teleport to="#page-root">
    <div id="custom-background" class="transition-background" :style="cssVars">
      <slot />
    </div>
  </Teleport>
</template>
<script setup lang="ts">
import { ref, computed, watch } from "vue";
import { useSlideContext } from "@slidev/client";

const { $nav, $slidev } = useSlideContext();

const currentNavState = computed(() => $nav.value.currentPage);

/**
 * 背景用のシード値（既存挙動を踏襲しつつ、外部propsで上書き可能）
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
  if (typeof shapeHueColor.value === "string") {
    return shapeHueColor.value;
  }
  const baseHue = seedNormalized.value * 360;
  return (baseHue + shapeHueColor.value) % 360;
});

// Provide a CSS-friendly color string; if numeric produce an HSL color
const shapeColorCss = computed(() => {
  const h = shapeHue.value;
  if (typeof h === "string") return h;
  return `hsl(${h} 65% 55%)`;
});

// Props: animation control
const props = withDefaults(
  defineProps<{
    animate?: boolean;
    speed?: number;
    pointsCount?: number;
  }>(),
  {
    animate: true,
    speed: 1,
    pointsCount: 7,
  }
);

// --- clip-path ベースへ移行 -----------------------------------------------
// シード値に基づき決定論的な擬似乱数を生成し、clip-path: polygon(...) を作る
function mulberry32(a: number) {
  return function () {
    let t = (a += 0x6d2b79f5);
    t = Math.imul(t ^ (t >>> 15), t | 1);
    t ^= t + Math.imul(t ^ (t >>> 7), t | 61);
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}

// animation parameter: use page number (scaled) as the time/seed driver
// each page gets a deterministic animation phase based on currentNavState
const time = computed(() => currentNavState.value * (props.speed ?? 1));

// compute animated clip-path using deterministic per-page seed + time
const clipPathValue = computed(() => {
  const base = Math.floor((Number(backGroundShape.value) || 0) * 1e9);
  const rnd = mulberry32(base || 1);

  const pointsCount = props.pointsCount ?? 7;
  const cx = 50;
  const cy = 40;
  const minR = 15; // percent radius
  const maxR = 50;

  const t = time.value * (props.speed ?? 1);

  const pts: string[] = [];
  for (let i = 0; i < pointsCount; i++) {
    const baseAngle = (i / pointsCount) * Math.PI * 2;
    const angleJitter = (rnd() - 0.5) * 0.6;
    const baseR = minR + rnd() * (maxR - minR);

    const angle = baseAngle + angleJitter + Math.sin(t + i * 0.9) * 0.35;
    const r = baseR + Math.sin(t * 1.2 + i * 1.3) * ((maxR - minR) * 0.12);

    const x = cx + Math.cos(angle) * r;
    const y = cy + Math.sin(angle) * r;
    pts.push(`${x.toFixed(2)}% ${y.toFixed(2)}%`);
  }

  return `polygon(${pts.join(", ")})`;
});

const cssVars = computed(() => {
  return {
    "--clip-path": clipPathValue.value,
    "--shape-color": shapeColorCss.value,
    "--glow-blur": `28px`,
    "--glow-opacity": `0.35`,
    "--glow-scale": `1.06`,
    "--clip-blur": `10px`,
    "--clip-blur-scale": `1.03`,
  };
});
// -------------------------------------------------------------------------
</script>
<style scoped>
.transition-background::before {
  /* clip-path を使ってマスクされた単色の図形を描く */
  background: var(--shape-color);
  clip-path: var(--clip-path);
}

.transition-background {
  position: relative;
  overflow: hidden;
  /* fallbacks (overridden by inline vars from :style) */

  --clip-path: polygon(50% 20%, 80% 50%, 60% 80%, 40% 80%, 20% 50%);
  filter: blur(5rem) brightness(1.5) contrast(1.5) saturate(1.1);
  opacity: 0.4;
  --shape-color: hsl(220 15% 60%);
  background-color: var(--color-bg, transparent);
  width: 110%;
  height: 120%;
}

/* floating soft shapes using pseudo elements */
.transition-background::before,
.transition-background::after {
  content: "";
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  mix-blend-mode: normal;
}

.transition-background::before {
  /* no transform/blur by default for sharp clip-path shape */
  transform: none;
  transition:
    opacity 1s ease,
    clip-path 1.5s cubic-bezier(0.25, 0.46, 0.45, 0.94),
    background-color 1.5s ease;
  opacity: 0.85;
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;
  will-change: opacity, clip-path;
}

.transition-background::after {
  /* no secondary element for now (keep markup simpler) */
  display: none;
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
  transition: transform 2s ease-in-out;
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
