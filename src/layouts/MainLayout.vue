<template>
  <q-layout view="lHh Lpr lFf" class="no-scroll love-layout">
    <div class="love-bg" aria-hidden="true">
      <span
        v-for="(heart, i) in hearts"
        :key="i"
        class="love"
        :style="{
          '--size': heart.size + 'px',
          '--left': heart.left + '%',
          '--duration': heart.duration + 's',
          '--delay': heart.delay + 's',
          '--drift': heart.drift + 'px',
        }"
      ></span>
    </div>

    <q-btn ref="btnRef" class="fly-btn" label="No" @mouseenter="onEnter" @click="onClick" />

    <q-btn
      v-for="decoy in decoys"
      :key="decoy.id"
      class="fly-btn decoy"
      :style="{ left: decoy.x + 'px', top: decoy.y + 'px' }"
      label="Catch me"
      @click="removeDecoy(decoy.id)"
    />

    <q-dialog v-model="showWin">
      <q-card class="valentine-card">
        <q-card-section class="valentine-header"> You must be my Valentine </q-card-section>
        <q-card-actions align="center" class="valentine-actions">
          <q-btn color="pink-4" outline rounded label="No" @click="onNo" />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <q-dialog v-model="showFollowup">
      <q-card class="valentine-card">
        <q-card-section class="valentine-header">See you on Valentine’s Day</q-card-section>
        <q-card-section class="valentine-sub">I’m excited already.</q-card-section>
      </q-card>
    </q-dialog>
    <div v-if="showNoOverlay" class="no-overlay" aria-live="assertive">
      <div v-for="(item, i) in noPositions" :key="i" class="no-msg" :style="noStyle(item)">YES</div>
    </div>
  </q-layout>
</template>

<script setup lang="ts">
import { ref, nextTick } from 'vue';
import type { QBtn } from 'quasar';

type Decoy = { id: number; x: number; y: number };

const btnRef = ref<QBtn | null>(null);
const moveCount = ref(4);
const maxMoves = 5;

const onClickCount = ref(5);
const maxOnClick = 5;

const decoys = ref<Decoy[]>([]);
let decoyId = 1;

let regenTimer: number | null = null;
const timeLimitMs = 5000;
const decoyCount = 2;

const showWin = ref(false);
const showFollowup = ref(false);
let followupTimer: number | null = null;
type NoMsg = { top: number; left: number; rotate: number; delay: number };
const showNoOverlay = ref(false);
const noPositions = ref<NoMsg[]>([]);
const hearts = [
  { size: 18, left: 8, duration: 18, delay: 0, drift: 24 },
  { size: 12, left: 16, duration: 14, delay: 3, drift: -18 },
  { size: 26, left: 28, duration: 22, delay: 6, drift: 36 },
  { size: 16, left: 38, duration: 16, delay: 1, drift: -28 },
  { size: 22, left: 52, duration: 20, delay: 4, drift: 30 },
  { size: 14, left: 60, duration: 15, delay: 7, drift: -22 },
  { size: 24, left: 70, duration: 21, delay: 2, drift: 34 },
  { size: 12, left: 78, duration: 13, delay: 5, drift: -20 },
  { size: 20, left: 86, duration: 19, delay: 8, drift: 26 },
  { size: 14, left: 92, duration: 17, delay: 9, drift: -24 },
  { size: 10, left: 45, duration: 12, delay: 10, drift: 16 },
  { size: 28, left: 24, duration: 24, delay: 11, drift: -40 },
];

async function onClick() {
  if (onClickCount.value >= maxOnClick && decoys.value.length === 0) {
    showWin.value = true;
    return;
  }
  onClickCount.value += 1;
  await moveRandom();
}

const onNo = () => {
  const count = 1000;
  const items: NoMsg[] = [];
  for (let i = 0; i < count; i += 1) {
    items.push({
      top: Math.random() * 94 + 3,
      left: Math.random() * 94 + 3,
      rotate: Math.random() * 60 - 30,
      delay: (i % 10) * 0.05,
    });
  }
  noPositions.value = items;
  showNoOverlay.value = true;
  showWin.value = false;
  if (followupTimer != null) {
    clearTimeout(followupTimer);
  }
  followupTimer = window.setTimeout(() => {
    showFollowup.value = true;
    showNoOverlay.value = false;
  }, 5000);
};

const noStyle = (item: NoMsg) => ({
  top: `${item.top}%`,
  left: `${item.left}%`,
  transform: `rotate(${item.rotate}deg)`,
  animationDelay: `${item.delay}s`,
});

const moveRandom = async () => {
  await nextTick();
  const el = btnRef.value?.$el as HTMLElement | undefined;
  if (!el) return;

  const btnW = el.offsetWidth;
  const btnH = el.offsetHeight;

  const maxX = Math.max(0, window.innerWidth - btnW);
  const maxY = Math.max(0, window.innerHeight - btnH);

  const x = Math.floor(Math.random() * (maxX + 1));
  const y = Math.floor(Math.random() * (maxY + 1));

  el.style.left = `${x}px`;
  el.style.top = `${y}px`;
};

const spawnDecoys = async (count: number) => {
  await nextTick();
  const el = btnRef.value?.$el as HTMLElement | undefined;
  if (!el) return;

  const btnW = el.offsetWidth;
  const btnH = el.offsetHeight;

  const maxX = Math.max(0, window.innerWidth - btnW);
  const maxY = Math.max(0, window.innerHeight - btnH);

  const items: Decoy[] = [];
  for (let i = 0; i < count; i += 1) {
    items.push({
      id: decoyId++,
      x: Math.floor(Math.random() * (maxX + 1)),
      y: Math.floor(Math.random() * (maxY + 1)),
    });
  }
  decoys.value = items;
};

const clearRegenTimer = () => {
  if (regenTimer != null) {
    clearTimeout(regenTimer);
    regenTimer = null;
  }
};

const startRegenLoop = async () => {
  clearRegenTimer();
  await spawnDecoys(decoyCount);

  regenTimer = window.setTimeout(() => {
    if (decoys.value.length === 0) {
      clearRegenTimer();
      return;
    }
    void startRegenLoop();
  }, timeLimitMs);
};

const removeDecoy = (id: number) => {
  decoys.value = decoys.value.filter((d) => d.id !== id);
  if (decoys.value.length === 0) {
    clearRegenTimer(); // stop regen if all clicked in time
  }
};

const onEnter = async () => {
  if (moveCount.value >= maxMoves) return;
  moveCount.value += 1;
  await moveRandom();

  if (moveCount.value === 5) {
    await startRegenLoop();
  }
};
</script>

<style scoped>
.yes-btn {
  width: 20px;
  height: 20px;
}

.no-scroll {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

.love-layout {
  background: radial-gradient(120% 120% at 10% 10%, #ffe9ef 0%, #fff7f9 40%, #fff 100%);
  position: relative;
}

.love-bg {
  position: fixed;
  inset: 0;
  overflow: hidden;
  pointer-events: none;
  z-index: 0;
}

.love {
  --size: 16px;
  --left: 50%;
  --duration: 18s;
  --delay: 0s;
  --drift: 24px;
  position: absolute;
  left: var(--left);
  bottom: -10%;
  width: var(--size);
  height: var(--size);
  background: #ff5a7a;
  transform: rotate(-45deg);
  animation:
    float-up var(--duration) linear infinite,
    drift var(--duration) ease-in-out infinite;
  animation-delay: var(--delay);
  opacity: 0.85;
  filter: drop-shadow(0 6px 10px rgba(255, 90, 122, 0.35));
}

.valentine-card {
  width: min(92vw, 420px);
  border-radius: 20px;
  background: linear-gradient(160deg, #ffffff 0%, #fff2f6 45%, #ffe1ea 100%);
  box-shadow: 0 18px 40px rgba(255, 90, 122, 0.25);
  padding: 8px 6px 12px;
}

.valentine-header {
  font-size: 22px;
  font-weight: 700;
  color: #c81f4a;
  text-align: center;
  padding: 20px 16px 8px;
}

.valentine-actions {
  gap: 10px;
  padding: 0 16px 18px;
}

.love::before,
.love::after {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background: #ff5a7a;
  border-radius: 50%;
}

.love::before {
  top: -50%;
  left: 0;
}

.love::after {
  left: 50%;
  top: 0;
}

.fly-btn {
  position: absolute;
  left: 16px;
  top: 16px;
}

.yes-btn,
.fly-btn,
.decoy,
.q-dialog {
  position: relative;
  z-index: 1;
}

.no-overlay {
  position: fixed;
  inset: 0;
  background: rgba(255, 70, 110, 0.2);
  backdrop-filter: blur(2px);
  z-index: 9999;
  pointer-events: none;
}

.no-msg {
  position: absolute;
  font-size: 28px;
  font-weight: 800;
  color: #c81f4a;
  text-shadow: 0 6px 16px rgba(200, 31, 74, 0.25);
  animation: pop 0.6s ease-out both;
}

@keyframes float-up {
  0% {
    transform: translateY(0) rotate(-45deg);
    opacity: 0;
  }
  10% {
    opacity: 0.9;
  }
  100% {
    transform: translateY(-120vh) rotate(-45deg);
    opacity: 0.2;
  }
}

@keyframes drift {
  0%,
  100% {
    margin-left: 0;
  }
  50% {
    margin-left: var(--drift);
  }
}

@keyframes pop {
  0% {
    transform: scale(0.6);
    opacity: 0;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}
</style>
