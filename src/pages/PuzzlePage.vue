<template>
  <div class="game-page">
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
      />
    </div>

    <div class="heart-grid">
      <div v-for="(slot, i) in slots" :key="i" class="cell">
        <div
          v-if="slot !== null"
          class="card"
          :class="{
            flipped: isFlipped(slot),
            wrong: incorrect.includes(slot),
            matched: matched.includes(slot),
          }"
          @click="onCardClick(slot)"
        >
          <div class="card-face card-back"></div>
          <div class="card-face card-front">
            <img :src="cards[slot]" :alt="`Photo ${slot + 1}`" />
          </div>
        </div>
        <div v-else class="spacer"></div>
      </div>
    </div>

    <!-- <q-btn class="next-btn" color="pink-4" outline rounded label="Next" @click="nextRound" /> -->

    <q-dialog v-model="showWin">
      <q-card class="win-card">
        <q-card-section class="win-title">You did it!</q-card-section>
        <q-card-section class="win-sub">All pairs matched.</q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
// import { useRouter } from 'vue-router';

const hearts = [
  { size: 14, left: 8, duration: 18, delay: 0, drift: 24 },
  { size: 10, left: 16, duration: 14, delay: 3, drift: -18 },
  { size: 20, left: 28, duration: 22, delay: 6, drift: 36 },
  { size: 12, left: 38, duration: 16, delay: 1, drift: -28 },
  { size: 18, left: 52, duration: 20, delay: 4, drift: 30 },
  { size: 10, left: 60, duration: 15, delay: 7, drift: -22 },
  { size: 20, left: 70, duration: 21, delay: 2, drift: 34 },
  { size: 10, left: 78, duration: 13, delay: 5, drift: -20 },
  { size: 16, left: 86, duration: 19, delay: 8, drift: 26 },
  { size: 12, left: 92, duration: 17, delay: 9, drift: -24 },
];

const images = [
  new URL('../assets/1.jpeg', import.meta.url).href,
  new URL('../assets/2.jpeg', import.meta.url).href,
  new URL('../assets/3.jpeg', import.meta.url).href,
  new URL('../assets/4.jpeg', import.meta.url).href,
  new URL('../assets/5.jpeg', import.meta.url).href,
  new URL('../assets/6.jpeg', import.meta.url).href,
  new URL('../assets/7.jpeg', import.meta.url).href,
  new URL('../assets/8.jpeg', import.meta.url).href,
  new URL('../assets/9.jpeg', import.meta.url).href,
  new URL('../assets/10.jpeg', import.meta.url).href,
  new URL('../assets/11.jpeg', import.meta.url).href,
  new URL('../assets/12.jpeg', import.meta.url).href,
  new URL('../assets/13.jpeg', import.meta.url).href,
  new URL('../assets/14.jpeg', import.meta.url).href,
  new URL('../assets/15.jpeg', import.meta.url).href,
  new URL('../assets/16.jpeg', import.meta.url).href,
  new URL('../assets/17.jpeg', import.meta.url).href,
  new URL('../assets/18.jpeg', import.meta.url).href,
];

const imagePairs = images.flatMap((image) => [image, image]);

const shuffleArray = <T,>(array: T[]) => {
  for (let i = array.length - 1; i > 0; i -= 1) {
    const j = Math.floor(Math.random() * (i + 1));
    const temp = array[i] as T;
    array[i] = array[j] as T;
    array[j] = temp;
  }
  return array;
};

const heartLayout: Array<Array<number | null>> = [
  [null, null, 0, 1, null, 2, 3, null, null],
  [null, 4, 5, 6, 7, 8, 9, 10, null],
  [11, 12, 13, 14, 15, 16, 17, 18, 19],
  [null, 20, 21, 22, 23, 24, 25, 26, null],
  [null, null, 27, 28, 29, 30, 31, null, null],
  [null, null, null, 32, 33, 34, null, null, null],
  [null, null, null, null, 35, null, null, null, null],
];

const slots = computed(() => heartLayout.flat());
const cards = ref(shuffleArray([...imagePairs]));
const selected = ref<number[]>([]);
const matched = ref<number[]>([]);
const incorrect = ref<number[]>([]);
const locked = ref(false);
const showWin = ref(false);
// const router = useRouter();

const isFlipped = (index: number) =>
  selected.value.includes(index) || matched.value.includes(index);

const onCardClick = (index: number) => {
  if (locked.value) return;
  if (selected.value.length === 2) return;
  if (matched.value.includes(index) || selected.value.includes(index)) return;

  if (selected.value.length === 0) {
    selected.value = [index];
    return;
  }

  const firstIndex = selected.value[0];
  if (firstIndex == null) return;
  selected.value = [firstIndex, index];

  if (cards.value[firstIndex] === cards.value[index]) {
    matched.value = [...matched.value, firstIndex, index];
    selected.value = [];
    if (matched.value.length === cards.value.length) {
      showWin.value = true;
    }
    return;
  }

  locked.value = true;
  incorrect.value = [firstIndex, index];
  window.setTimeout(() => {
    incorrect.value = [];
    selected.value = [];
    locked.value = false;
  }, 900);
};

// function nextRound() {
//   void router.push('/second');
// }
</script>

<style scoped>
.game-page {
  min-height: 100vh;
  display: grid;
  place-items: center;
  background: radial-gradient(120% 120% at 10% 10%, #ffe9ef 0%, #fff7f9 40%, #fff 100%);
  overflow: hidden;
  position: relative;
}

.love-bg {
  position: absolute;
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
  opacity: 0.7;
  filter: drop-shadow(0 6px 10px rgba(255, 90, 122, 0.3));
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

.heart-grid {
  display: grid;
  grid-template-columns: repeat(9, var(--cell-size));
  gap: 8px;
  justify-content: center;
  align-items: center;
  --cell-size: clamp(40px, 9.5vh, 80px);
  max-width: 95vw;
  position: relative;
  z-index: 1;
}

.next-btn {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 2;
}

.cell {
  width: var(--cell-size);
  height: var(--cell-size);
  display: grid;
  place-items: center;
}

.spacer {
  width: 100%;
  height: 100%;
}

.card {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.5s ease;
  cursor: pointer;
}

.card.flipped {
  transform: rotateY(180deg);
}

.card-face {
  position: absolute;
  inset: 0;
  backface-visibility: hidden;
  border-radius: 8px;
}

.card-back {
  background: linear-gradient(135deg, #ffd1dc 0%, #ffb3c7 100%);
  box-shadow: 0 8px 18px rgba(255, 90, 122, 0.25);
}

.card-front {
  transform: rotateY(180deg);
  overflow: hidden;
}

.card-front img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.card.wrong::after {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: 8px;
  background: rgba(255, 60, 80, 0.45);
  animation: wrong-flash 0.5s ease;
}

.win-card {
  width: min(90vw, 360px);
  border-radius: 18px;
  background: linear-gradient(160deg, #ffffff 0%, #fff2f6 45%, #ffe1ea 100%);
  box-shadow: 0 18px 40px rgba(255, 90, 122, 0.25);
  padding: 6px 4px 10px;
}

.win-title {
  font-size: 22px;
  font-weight: 700;
  color: #c81f4a;
  text-align: center;
  padding: 18px 12px 6px;
}

.win-sub {
  font-size: 14px;
  color: #7b2a3e;
  text-align: center;
  padding: 0 16px 14px;
}

@keyframes wrong-flash {
  0%,
  100% {
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
}

@keyframes float-up {
  0% {
    transform: translateY(0) rotate(-45deg);
    opacity: 0;
  }
  10% {
    opacity: 0.8;
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
</style>
