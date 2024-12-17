<template>
  <div class="music-widget" ref="musicWidgetRef">
    <div class="overlay" ref="overlayRef"></div>
    <div
      class="disc"
      :style="{
        backgroundImage: 'url(' + props.songs[currentSong].thumbnail + ')',
      }"
      ref="discRef"
      @click="toggleDiscExpansion"
    >
      <span
        v-for="(item, index) in 3"
        :key="index"
        :class="`c${index + 1}`"
        ref="circleRefs"
      ></span>
    </div>
    <div class="info" ref="infoRef">
      <span class="soundbar">
        <div v-for="(item, index) in 4" :key="index" class="bar"></div>
      </span>
      <div class="text">
        <p class="artist" ref="artistRef">{{ props.songs[0].artist }}</p>
        <h3 class="song" ref="songRef">{{ props.songs[0].title }}</h3>
        <div
          class="progress-bar"
          ref="progressBarRef"
          @mouseover="scaleProgressBar(2)"
          @mouseleave="scaleProgressBar(1)"
          @click="handleProgressBarClick"
        >
          <div class="total-progress"></div>
          <span
            class="active-progress"
            ref="activeProgressRef"
            @click=""
          ></span>
        </div>
        <div class="timestamp" ref="timestampRef">
          <span class="current-time">
            <span class="minutes" ref="minutesRef">0</span>:<span
              class="tens-seconds"
              ref="tenSecondsRef"
              >0</span
            ><span class="seconds" ref="secondsRef">0</span>
          </span>
          /
          <span class="total-time" ref="totalTimeRef">{{
            props.songs[0].duration
          }}</span>
        </div>
      </div>
    </div>
    <div class="prev" @click="prevSong()">
      <img src="../assets/images/arrow.png" class="prevbtn" ref="prevBtnRef" />
    </div>
    <div class="next" @click="nextSong()">
      <img src="../assets/images/arrow.png" class="nextbtn" ref="nextBtnRef" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { gsap } from "gsap";

const props = defineProps({
  songs: {
    type: Array,
    required: true,
  },
});

const ROTATION_DURATION = 20;
const EXPAND_DURATION = 1;
const FADE_DURATION = 0.5;
const DISC_ANIMATION_DURATION = 0.5;

const musicWidgetRef = ref(null);
const discRef = ref(null);
const overlayRef = ref(null);
const circleRefs = ref([]);
const infoRef = ref(null);
const artistRef = ref(null);
const songRef = ref(null);
const progressBarRef = ref(null);
const activeProgressRef = ref(null);
const totalTimeRef = ref(null);
const timestampRef = ref(null);
const prevBtnRef = ref(null);
const nextBtnRef = ref(null);
const minutesRef = ref(null);
const tenSecondsRef = ref(null);
const secondsRef = ref(null);

const currentSong = ref(0);
const state = ref({
  isSpinning: true,
  isExpanded: false,
  progressBarScale: 1,
  previousTime: { minutes: 0, tensSeconds: 0, seconds: 0 },
  totalSeconds: 0,
  currentTime: 0,
});

let timerInterval;

const animations = {
  expand: () =>
    gsap.to(discRef.value, {
      width: "100%",
      height: "100%",
      rotation: 0,
      top: 0,
      left: 0,
      borderRadius: "0%",
      duration: EXPAND_DURATION,
      ease: "power2.inOut",
    }),
  collapse: () =>
    gsap.to(discRef.value, {
      width: "100%",
      height: "100%",
      top: "-45%",
      left: "0%",
      borderRadius: "100%",
      duration: EXPAND_DURATION,
      ease: "power2.inOut",
    }),
  fadeCircles: (fadeIn) =>
    gsap.to(circleRefs.value, {
      opacity: fadeIn ? 0.8 : 0,
      duration: FADE_DURATION,
      ease: fadeIn ? "power2.in" : "power2.out",
      onComplete: () => {
        if (!fadeIn)
          circleRefs.value.forEach((c) => (c.style.display = "none"));
      },
    }),
  toggleOverlay: (show) =>
    gsap.to(overlayRef.value, {
      opacity: show ? 0.1 : 0,
      display: show ? "block" : "none",
      duration: FADE_DURATION,
      ease: show ? "power2.out" : "power2.in",
    }),
  blurInfo: (blur) =>
    gsap.to(infoRef.value, {
      filter: `blur(${blur ? 2 : 0}px)`,
      duration: blur ? FADE_DURATION : EXPAND_DURATION,
      ease: "power2.out",
    }),
};

const playSong = (index = 0) => {
  updatePlayerTime(0);
  const song = props.songs[index];
  state.totalSeconds = convertToSeconds(song.duration);
  state.currentTime = 0;

  if (timerInterval) clearInterval(timerInterval);
  timerInterval = setInterval(() => {
    if (state.currentTime < state.totalSeconds) {
      state.currentTime++;
      const timeComponents = formatTimeComponents(state.currentTime);
      updateTimestamp(timeComponents);
      updateProgressBar(state.currentTime);
    } else {
      clearInterval(timerInterval);
    }
  }, 1000);

  animations.rotate.play();
  gsap.to(timestampRef.value, { opacity: 0, duration: -0.1 });

  gsap.to(infoRef.value, {
    y: 10,
    opacity: 0,
    duration: FADE_DURATION,
    ease: "power2.inOut",
    onComplete: () => {
      songRef.value.textContent = song.title;
      artistRef.value.textContent = song.artist;
      totalTimeRef.value.textContent = song.duration;

      gsap.to(infoRef.value, {
        opacity: 1,
        y: 0,
        duration: FADE_DURATION,
        ease: "power2.inOut",
      });
      gsap.to(timestampRef.value, { opacity: 1, duration: FADE_DURATION });
    },
  });

  gsap.to(discRef.value, {
    top: "-120%",
    duration: DISC_ANIMATION_DURATION,
    ease: "power2.inOut",
    onComplete: () => {
      gsap.to(discRef.value, {
        top: "-45%",
        duration: DISC_ANIMATION_DURATION,
        ease: "power2.inOut",
      });
    },
  });

  animations.blurInfo(true);
  gsap.to(infoRef.value, {
    opacity: 0,
    duration: FADE_DURATION,
    onComplete: () => {
      songRef.value.textContent = song.title;
      artistRef.value.textContent = song.artist;
      totalTimeRef.value.textContent = song.duration;
      animations.blurInfo(false);
      gsap.to(infoRef.value, {
        opacity: 1,
        duration: FADE_DURATION,
      });
    },
  });
};

const expandDisc = () => {
  animations.fadeCircles(false);
  animations.expand();
  animations.toggleOverlay(true);
  animations.blurInfo(true);
  animations.rotate.pause();
  state.isSpinning = false;
};

const collapseDisc = () => {
  circleRefs.value.forEach((c) => (c.style.display = "block"));
  animations.fadeCircles(true);
  animations.collapse();
  animations.toggleOverlay(false);
  animations.blurInfo(false);
  animations.rotate.restart();
  state.isSpinning = true;
};

const toggleDiscExpansion = () => {
  state.isExpanded = !state.isExpanded;

  if (state.isExpanded) {
    expandDisc();
  } else {
    collapseDisc();
  }
};

const nextSong = () => {
  currentSong.value = (currentSong.value + 1) % props.songs.length;
  playSong(currentSong.value);
};

const prevSong = () => {
  currentSong.value =
    (currentSong.value - 1 + props.songs.length) % props.songs.length;
  playSong(currentSong.value);
};

const updateProgressBar = (currentSeconds) => {
  const progressPercent = (currentSeconds / state.totalSeconds) * 100;

  if (activeProgressRef.value) {
    activeProgressRef.value.style.width = `${progressPercent}%`;
  }
};

const updatePlayerTime = (newTime) => {
  state.currentTime = newTime;
  const timeComponents = formatTimeComponents(state.currentTime);
  updateTimestamp(timeComponents);
  updateProgressBar(state.currentTime);
};

const animateDigit = (element, oldValue, newValue) => {
  gsap.to(element, {
    y: -5,
    duration: 0.3,
    ease: "power2.out",
    onComplete: () => {
      element.textContent = newValue;
      gsap.fromTo(
        element,
        { y: 5 },
        { y: 0, duration: 0.3, ease: "power2.in" }
      );
    },
  });
};

const convertToSeconds = (time) => {
  const [minutes, seconds] = time.split(":").map(Number);
  return minutes * 60 + seconds;
};

const formatTimeComponents = (seconds) => {
  const minutes = Math.floor(seconds / 60);
  const remainingSeconds = seconds % 60;
  return {
    minutes,
    tensSeconds: Math.floor(remainingSeconds / 10),
    seconds: remainingSeconds % 10,
  };
};

const updateTimestamp = (timeComponents) => {
  if (state.previousTime) {
    Object.entries(timeComponents).forEach(([key, value]) => {
      let refElement = null;

      if (key === "minutes") {
        refElement = minutesRef.value;
      } else if (key === "tensSeconds") {
        refElement = tenSecondsRef.value;
      } else if (key === "seconds") {
        refElement = secondsRef.value;
      }

      if (refElement && value !== state.previousTime[key]) {
        animateDigit(refElement, state.previousTime[key], value);
      }
    });
  }

  state.previousTime = timeComponents;
};

const scaleProgressBar = (scale) => {
  state.progressBarScale = scale;
  gsap.to(progressBarRef.value, { scale, duration: 0.3, ease: "power2.out" });
};

const handleProgressBarClick = (e) => {
  const rect = progressBarRef.value.getBoundingClientRect();
  const clickX = e.clientX - rect.left;
  const adjustedClickX = clickX / state.progressBarScale;
  const newProgress = adjustedClickX / progressBarRef.value.offsetWidth;
  const newTime = Math.floor(newProgress * state.totalSeconds);

  updatePlayerTime(newTime);
};

onMounted(() => {
  animations.rotate = gsap.to(discRef.value, {
    rotation: 360,
    duration: ROTATION_DURATION,
    repeat: -1,
    ease: "linear",
    paused: false,
  });

  playSong();
});
</script>

<style scoped>
.music-widget {
  width: 350px;
  height: 350px;
  background: linear-gradient(
    to bottom,
    var(--disc-color) 0%,
    var(--bg-color) 100%
  );
  box-shadow: rgba(99, 99, 99, 0.2) 0px 2px 8px 0px;
  border-radius: 50px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  position: relative;
  overflow: hidden;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgb(107, 107, 107);
  filter: blur(100px);
  opacity: 0;
  display: none;
  z-index: 3;
}

.disc {
  background-size: cover;
  height: 100%;
  width: 100%;
  border-radius: 100%;
  position: absolute;
  top: -45%;
  box-shadow: rgba(50, 50, 93, 0.25) 0px 30px 60px -12px,
    rgba(0, 0, 0, 0.3) 0px 18px 36px -18px;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 4;
  cursor: pointer;
}

.c1 {
  background-color: var(--inside-disc-2);
  width: 30%;
  height: 30%;
  z-index: 1;
  border-radius: 100%;
  border: 2px solid white;
  opacity: 0.8;
}

.c2 {
  background-color: var(--inside-disc-1);
  width: 20%;
  height: 20%;
  z-index: 2;
  border-radius: 100%;
  position: absolute;
}

.c3 {
  background-color: var(--disc-color);
  width: 15%;
  height: 15%;
  z-index: 3;
  border-radius: 100%;
  position: absolute;
  border: 2px solid white;
}

.soundbar {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 90%;
  height: 20px;
  padding-bottom: 0.5rem;
}
.bar {
  background: grey;
  height: 20px;
  position: relative;
  width: 3px;
  border-radius: 10px;
  animation: sound 0s -20000ms linear infinite alternate;
}

.bar:nth-child(1) {
  left: 1px;
  animation-duration: 474ms;
}
.bar:nth-child(2) {
  left: 5px;
  animation-duration: 433ms;
}
.bar:nth-child(3) {
  left: 9px;
  animation-duration: 407ms;
}
.bar:nth-child(4) {
  left: 13px;
  animation-duration: 458ms;
}

.text {
  width: 100%;
  text-align: center;
  align-items: center;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  position: relative;
  padding-bottom: 20%;
  z-index: 2;
}

.artist {
  color: grey;
  font-size: 0.9rem;
  margin-bottom: -0.25rem;
}

.song {
  color: var(--text-color);
  font-size: 1.5rem;
  letter-spacing: 1px;
}

.progress-bar {
  width: 30px;
  height: 3px;
  position: relative;
  cursor: pointer;
}

.total-progress {
  position: relative;
  width: 100%;
  height: 100%;
  background-color: var(--inside-disc-1);
  z-index: 1;
  border-radius: 10px;
}

.active-progress {
  width: 50%;
  height: 100%;
  position: absolute;
  left: 0;
  top: 0;
  background-color: var(--inside-disc-2);
  z-index: 2;
  border-radius: 10px 0 0 10px;
}

.timestamp {
  display: flex;
  color: grey;
  gap: 0.25rem;
  font-weight: 800;
  font-size: 1rem;
  margin-top: 0.1rem;
}

.current-time {
  color: var(--text-color);
}

.next,
.prev {
  position: absolute;
  top: 0;
  width: 20%;
  height: 100%;
  opacity: 0;
  z-index: 3;
  display: flex;
  align-items: flex-end;
  transition: opacity 0.3s;
  cursor: pointer;
  transition: 0.5s all ease;
}

.prev {
  left: 0;
  background: linear-gradient(
    to right,
    rgba(190, 190, 203, 0.3) 0%,
    transparent 100%
  );
}

.next {
  right: 0;
  background: linear-gradient(
    to left,
    rgba(190, 190, 203, 0.3) 0%,
    transparent 100%
  );
  justify-content: flex-end;
}

.prev:hover,
.next:hover {
  opacity: 1;
  transition: 0.5s all ease;
}

.prevbtn {
  transform: rotate(180deg);
  margin-bottom: 3rem;
  margin-left: 1rem;
  width: 15px;
  height: 15px;
  opacity: 0.3;
}

.nextbtn {
  margin-bottom: 3rem;
  margin-right: 1rem;
  width: 15px;
  height: 15px;
  opacity: 0.3;
}
</style>
