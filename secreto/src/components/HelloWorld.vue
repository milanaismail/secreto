<script setup lang="ts">
import { gsap } from "gsap";
import { onMounted, onUnmounted, ref } from "vue";
import heroImage from "../assets/hero.png";

const background = ref<HTMLElement | null>(null);
const heading = ref<HTMLElement | null>(null);
const headingWords = [
  "Niet",
  "alles",
  "is",
  "wat",
  "het",
  "lijkt.",
  '<br />',
  "Sommige",
  "ervaringen",
  "moet",
  "je",
  "zelf",
  "ontdekken.",
];
let zoomTimer: ReturnType<typeof setTimeout> | undefined;
let zoomAnimation: ReturnType<typeof gsap.to> | undefined;
let textAnimation: gsap.core.Timeline | undefined;

onMounted(() => {
  const words = heading.value?.querySelectorAll(".heading-word");

  if (words) {
    gsap.set(words, { opacity: 0 });
  }

  zoomTimer = setTimeout(() => {
    if (background.value) {
      zoomAnimation = gsap.to(background.value, {
        width: "100vw",
        height: "100vh",
        duration: 2.5,
        ease: "power3.inOut",
      });
    }

    if (words) {
      textAnimation = gsap.timeline();
      textAnimation
        .to(words, {
          opacity: 0.12,
          ease: "none",
        })
        .to(words, {
          opacity: 1,
          duration: 1.2,
          ease: "none",
        });
    }
  }, 3500);
});

onUnmounted(() => {
  if (zoomTimer) clearTimeout(zoomTimer);
  zoomAnimation?.kill();
  textAnimation?.kill();
});
</script>

<template>
  <div class="relative h-screen w-full">
  <!-- <img alt="Vue logo" src="../assets/hero-blur.png" class="block h-full w-full object-cover" /> -->
  <div class="block h-full w-full object-cover bg-background"></div>
  <div
      class="absolute left-1/2 top-1/2 aspect-[9/16] h-3/4 -translate-x-1/2 -translate-y-1/2"
    >
      <div
        ref="background"
        class="absolute left-1/2 top-1/2 h-full w-full -translate-x-1/2 -translate-y-1/2 bg-cover bg-center"
        :style="{ backgroundImage: `url(${heroImage})` }"
      />
      <video
        class="absolute inset-0 z-10 h-full w-full object-cover"
        playsinline
        autoplay
        muted
      >
        <source src="../assets/secreto_transparent_fixed.webm" type="video/webm" />
        Your browser does not support the video tag.
      </video>
    </div>
    <h1
      ref="heading"
      class="absolute bottom-8 left-0 z-20 w-full p-6 text-center text-white whitespace-pre-wrap"
    >
      <template v-for="(word, index) in headingWords" :key="`${word}-${index}`">
        <br v-if="word === '<br />'" />
        <span v-else class="heading-word">
          {{ word }}{{ index < headingWords.length - 1 ? " " : "" }}
        </span>
      </template>
    </h1>
  </div>
</template>
