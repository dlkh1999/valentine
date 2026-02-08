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

    <q-btn
      ref="btnRef"
      rounded
      flat
      class="fly-btn"
      label="No"
      @mouseenter="onEnter"
      @click="onClick"
    />

    <q-btn rounded flat class="yes-center" label="Yes" @click="nextRound" />

    <q-btn
      v-for="decoy in decoys"
      :key="decoy.id"
      class="fly-btn decoy"
      :style="{ left: decoy.x + 'px', top: decoy.y + 'px' }"
      label="No"
      rounded
      flat
      @click="removeDecoy(decoy.id)"
    />

    <q-dialog v-model="showWin">
      <q-card class="valentine-card">
        <q-card-section class="valentine-header" v-if="onClickCount >= 5">
          Will you be my Valentine ?
        </q-card-section>
        <q-card-section v-if="clickMessage" class="valentine-sub">
          {{ clickMessage }}
        </q-card-section>
        <q-card-actions align="center" class="valentine-actions">
          <q-btn
            color="pink-4"
            outline
            rounded
            label="Yes"
            @click="onNo"
            v-if="onClickCount >= 5"
          />
          <q-btn color="pink-4" outline rounded label="No" @click="onNo" v-if="onClickCount >= 5" />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <q-dialog v-model="showFollowup">
      <q-card class="valentine-card">
        <q-card-section class="valentine-header">See you on Valentine’s Day</q-card-section>
      </q-card>
    </q-dialog>
    <div v-if="showConfetti" class="confetti" aria-hidden="true">
      <span v-for="(c, i) in confetti" :key="i" class="confetti-heart" :style="confettiStyle(c)" />
    </div>
    <div v-if="complimentMessage" class="compliment">{{ complimentMessage }}</div>
    <div v-if="showNoOverlay" class="no-overlay" aria-live="assertive">
      <div v-for="(item, i) in noPositions" :key="i" class="no-msg" :style="noStyle(item)">YES</div>
    </div>

    <q-btn
      class="next-btn"
      color="pink-4"
      outline
      rounded
      label="Next"
      @click="nextRound"
      v-if="showNextRound"
    />
  </q-layout>
</template>

<script setup lang="ts">
import { ref, nextTick, computed } from 'vue';
import type { QBtn } from 'quasar';
import { useRouter } from 'vue-router';

type Decoy = { id: number; x: number; y: number };
const router = useRouter();

const btnRef = ref<QBtn | null>(null);
const moveCount = ref(0);
const maxMoves = 10;

const onClickCount = ref(0);
const maxOnClick = 5;

const decoys = ref<Decoy[]>([]);
let decoyId = 1;

let regenTimer: number | null = null;
const timeLimitMs = 60000;
const decoyCount = 15;

const showWin = ref(false);
const showFollowup = ref(false);
const showNextRound = ref(false);
let followupTimer: number | null = null;
type NoMsg = { top: number; left: number; rotate: number; delay: number };
const showNoOverlay = ref(false);
const noPositions = ref<NoMsg[]>([]);
const winClicksLeft = ref(0);
const showConfetti = ref(false);
const confetti = ref<{ size: number; dx: number; dy: number; delay: number; rotate: number }[]>([]);
let confettiTimer: number | null = null;
const complimentMessage = ref('');
let complimentTimer: number | null = null;
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

const clickMessages = [
  'Please don`t say no',
  'Don`t make me sad',
  'Are you sure???',
  'You are breaking my heart </3',
];
const clickMessage = computed(() => clickMessages[onClickCount.value - 1] ?? '');
const compliments = [
  'You light up the room.',
  'You have the best smile.',
  'You make everything better.',
  'You are genuinely adorable.',
  'You are my favorite person.',
  'You are pure sunshine.',
];

const sleep = (ms: number) => new Promise<void>((resolve) => setTimeout(resolve, ms));

const shrinkOnce = async (step: number, total: number) => {
  await nextTick();
  const el = btnRef.value?.$el as HTMLElement | undefined;
  if (!el) return;
  el.style.transformOrigin = 'center';
  el.style.transition = 'transform 220ms ease';
  const scale = 1 - (total - step + 1) * 0.22;
  el.style.transform = `scale(${scale})`;
  await sleep(260);
};

async function onClick() {
  onClickCount.value += 1;
  if (onClickCount.value >= 1 && onClickCount.value <= 4) {
    showWin.value = true;
    await moveRandom();
    return;
  }

  if (onClickCount.value >= maxOnClick && decoys.value.length === 0) {
    if (winClicksLeft.value === 0) {
      winClicksLeft.value = 5;
    }
    await shrinkOnce(winClicksLeft.value, 5);
    winClicksLeft.value -= 1;
    if (winClicksLeft.value <= 0) {
      showWin.value = true;
    }
    return;
  }

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
    showNextRound.value = true;
    showNoOverlay.value = false;
    const count = 32;
    const items = [];
    for (let i = 0; i < count; i += 1) {
      items.push({
        size: Math.floor(Math.random() * 14) + 10,
        dx: Math.random() * 320 - 160,
        dy: -(Math.random() * 260 + 80),
        delay: Math.random() * 0.2,
        rotate: Math.random() * 120 - 60,
      });
    }
    confetti.value = items;
    showConfetti.value = true;
    if (confettiTimer != null) {
      clearTimeout(confettiTimer);
    }
    confettiTimer = window.setTimeout(() => {
      showConfetti.value = false;
    }, 2400);
  }, 5000);
};

const noStyle = (item: NoMsg) => ({
  top: `${item.top}%`,
  left: `${item.left}%`,
  transform: `rotate(${item.rotate}deg)`,
  animationDelay: `${item.delay}s`,
});

const confettiStyle = (c: (typeof confetti.value)[number]) => ({
  '--size': `${c.size}px`,
  '--dx': `${c.dx}px`,
  '--dy': `${c.dy}px`,
  '--delay': `${c.delay}s`,
  '--rot': `${c.rotate}deg`,
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

  if (complimentTimer != null) {
    clearTimeout(complimentTimer);
  }

  complimentMessage.value = compliments[Math.floor(Math.random() * compliments.length)] ?? '';
  complimentTimer = window.setTimeout(() => {
    complimentMessage.value = '';
  }, 1400);
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

  if (moveCount.value === maxMoves) {
    await startRegenLoop();
  }
};

function nextRound() {
  void router.push('/first');
}
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
  background-size: 200% 200%;
  animation: bg-shift 14s ease-in-out infinite;
  position: relative;
}

.love-bg {
  position: fixed;
  inset: 0;
  overflow: hidden;
  pointer-events: none;
  z-index: 0;
}

.yes-center {
  position: fixed;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  z-index: 2;
}

.next-btn {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 2;
}

@keyframes heart-glow {
  0%,
  100% {
    opacity: 0.55;
    box-shadow:
      0 0 40px rgba(255, 90, 122, 0.35),
      0 0 110px rgba(255, 90, 122, 0.3),
      0 0 180px rgba(255, 90, 122, 0.2);
  }
  50% {
    opacity: 1;
    box-shadow:
      0 0 70px rgba(255, 90, 122, 0.55),
      0 0 150px rgba(255, 90, 122, 0.45),
      0 0 240px rgba(255, 90, 122, 0.35);
  }
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

.valentine-sub {
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

.confetti {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 6;
}

.confetti-heart {
  --size: 12px;
  --dx: 0px;
  --dy: -120px;
  --delay: 0s;
  --rot: 0deg;
  position: absolute;
  left: 50%;
  top: 50%;
  width: var(--size);
  height: var(--size);
  background: #ff5a7a;
  transform: translate(-50%, -50%) rotate(-45deg);
  opacity: 0;
  animation: confetti-burst 1.6s ease-out both;
  animation-delay: var(--delay);
}

.confetti-heart::before,
.confetti-heart::after {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background: #ff5a7a;
  border-radius: 50%;
}

.confetti-heart::before {
  top: -50%;
  left: 0;
}

.confetti-heart::after {
  left: 50%;
  top: 0;
}

.compliment {
  position: fixed;
  left: 50%;
  bottom: 36px;
  transform: translateX(-50%);
  padding: 8px 14px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.85);
  color: #b21b43;
  font-weight: 600;
  box-shadow: 0 10px 24px rgba(255, 90, 122, 0.2);
  z-index: 2;
  animation: compliment-fade 1.4s ease-out both;
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

@keyframes confetti-burst {
  0% {
    opacity: 0;
    transform: translate(-50%, -50%) rotate(-45deg) scale(0.7);
  }
  15% {
    opacity: 1;
  }
  100% {
    opacity: 0;
    transform: translate(calc(-50% + var(--dx)), calc(-50% + var(--dy)))
      rotate(calc(-45deg + var(--rot))) scale(1.05);
  }
}

@keyframes bg-shift {
  0%,
  100% {
    background-position: 0% 20%;
  }
  50% {
    background-position: 100% 80%;
  }
}

@keyframes compliment-fade {
  0% {
    opacity: 0;
    transform: translateX(-50%) translateY(6px);
  }
  15% {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
  100% {
    opacity: 0;
    transform: translateX(-50%) translateY(-6px);
  }
}
</style>
