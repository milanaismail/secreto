<script setup lang="ts">
import { gsap } from "gsap";
import { onMounted, onUnmounted, ref } from "vue";
import heroBlur from "../assets/hero-blur.png";
import heroImage from "../assets/hero.png";
import door from "../assets/door.jpeg";

const background = ref<HTMLElement | null>(null);
const heroSection = ref<HTMLElement | null>(null);
const darkOverlay = ref<HTMLElement | null>(null);
const videoOverlay = ref<HTMLElement | null>(null);
const heading = ref<HTMLElement | null>(null);
const videoSection = ref<HTMLElement | null>(null);
const scrollVideo = ref<HTMLVideoElement | null>(null);
const doorFigure = ref<HTMLElement | null>(null);
const doorImage = ref<HTMLImageElement | null>(null);

let zoomTimer: ReturnType<typeof setTimeout> | undefined;
let zoomAnimation: ReturnType<typeof gsap.to> | undefined;
let textAnimation: gsap.core.Tween | undefined;
let videoFrame: number | undefined;
let heroFrame: number | undefined;
let doorFrame: number | undefined;
let videoCleanup: (() => void) | undefined;
let heroCleanup: (() => void) | undefined;
let doorCleanup: (() => void) | undefined;
let videoReady = false;

onMounted(() => {
  if (heading.value) {
    gsap.set(heading.value, {
      autoAlpha: 0,
      y: 35,
      filter: "blur(12px)",
    });
  }

  zoomTimer = setTimeout(() => {
    const clipTargets = [background.value, videoOverlay.value].filter(
      (element): element is HTMLElement => element !== null,
    );

    if (clipTargets.length > 0) {
      zoomAnimation = gsap.to(clipTargets, {
        clipPath: "inset(0% 0% round 0vw)",
        duration: 2.5,
        ease: "power4.inOut",
      });
    }

    if (heading.value) {
      textAnimation = gsap.fromTo(
        heading.value,
        {
          autoAlpha: 0,
          y: 35,
          filter: "blur(12px)",
        },
        {
          autoAlpha: 1,
          y: 0,
          filter: "blur(0px)",
          duration: 1.2,
      delay: 0.3,

          ease: "power3.out",
        },
      );
    }
  }, 3500);

  const updateVideo = () => {
    videoFrame = undefined;

    const section = videoSection.value;
    const video = scrollVideo.value;
    if (
      !section ||
      !video ||
      !videoReady ||
      video.readyState < HTMLMediaElement.HAVE_CURRENT_DATA
    ) {
      return;
    }

    const sectionRect = section.getBoundingClientRect();
    const videoRect = video.getBoundingClientRect();
    const scrollRange = Math.max(
      1,
      sectionRect.height - videoRect.height,
    );
    const progress = Math.min(
      1,
      Math.max(0, -sectionRect.top / scrollRange),
    );
    video.currentTime = progress * video.duration;
  };

  const updateHeroBackground = () => {
    heroFrame = undefined;

    const section = heroSection.value;
    const overlay = darkOverlay.value;
    if (!section || !overlay) return;

    const sectionRect = section.getBoundingClientRect();
    const fadeStart = sectionRect.height / 2;
    const progress = Math.min(
      1,
      Math.max(0, -sectionRect.top / fadeStart - 1),
    );
    overlay.style.opacity = String(progress);
  };

  const requestVideoUpdate = () => {
    if (videoFrame === undefined) {
      videoFrame = requestAnimationFrame(updateVideo);
    }
  };

  const requestHeroUpdate = () => {
    if (heroFrame === undefined) {
      heroFrame = requestAnimationFrame(updateHeroBackground);
    }
  };

  const updateDoorImage = () => {
    doorFrame = undefined;

    const figure = doorFigure.value;
    const image = doorImage.value;
    if (!figure || !image) return;

    const figureRect = figure.getBoundingClientRect();
    const scrollRange = window.innerHeight + figureRect.height;
    const progress = Math.min(
      1,
      Math.max(0, (window.innerHeight - figureRect.top) / scrollRange),
    );
    const scale = 1.2 - progress * 0.2;
    image.style.transform = `scale(${scale})`;
  };

  const requestDoorUpdate = () => {
    if (doorFrame === undefined) {
      doorFrame = requestAnimationFrame(updateDoorImage);
    }
  };

  window.addEventListener("scroll", requestHeroUpdate, { passive: true });
  window.addEventListener("resize", requestHeroUpdate);
  requestHeroUpdate();

  heroCleanup = () => {
    window.removeEventListener("scroll", requestHeroUpdate);
    window.removeEventListener("resize", requestHeroUpdate);
    if (heroFrame !== undefined) cancelAnimationFrame(heroFrame);
  };

  window.addEventListener("scroll", requestDoorUpdate, { passive: true });
  window.addEventListener("resize", requestDoorUpdate);
  requestDoorUpdate();

  doorCleanup = () => {
    window.removeEventListener("scroll", requestDoorUpdate);
    window.removeEventListener("resize", requestDoorUpdate);
    if (doorFrame !== undefined) cancelAnimationFrame(doorFrame);
  };

  if (scrollVideo.value) {
    const video = scrollVideo.value;
    video.pause();

    const markVideoReady = () => {
      videoReady = true;
      requestVideoUpdate();
    };

    video.addEventListener("loadeddata", markVideoReady);
    video.addEventListener("canplay", markVideoReady);
    window.addEventListener("scroll", requestVideoUpdate, { passive: true });
    window.addEventListener("resize", requestVideoUpdate);
    requestVideoUpdate();

    videoCleanup = () => {
      video.removeEventListener("loadeddata", markVideoReady);
      video.removeEventListener("canplay", markVideoReady);
      window.removeEventListener("scroll", requestVideoUpdate);
      window.removeEventListener("resize", requestVideoUpdate);
      if (videoFrame !== undefined) cancelAnimationFrame(videoFrame);
    };
  }
});

onUnmounted(() => {
  if (zoomTimer) clearTimeout(zoomTimer);
  zoomAnimation?.kill();
  textAnimation?.kill();
  heroCleanup?.();
  doorCleanup?.();
  videoCleanup?.();
});
</script>

<template>
  <div ref="heroSection" class="relative h-screen w-full">
    <!-- <img
      alt=""
      :src="heroBlur"
      class="absolute inset-0 block h-full w-full object-cover"
    /> -->
    <div  class="bg-background absolute inset-0 block h-full w-full object-cover">

    </div>
    <div
      ref="background"
      class="absolute inset-0 bg-cover bg-center"
      :style="{
        backgroundImage: `url(${heroImage})`,
        clipPath: 'inset(22% 41% round 23vw)',
      }"
    >
    </div>
    <div
      ref="videoOverlay"
      class="absolute inset-0"
      :style="{ clipPath: 'inset(22% 41% round 23vw)' }"
    >
      <div
        class="absolute left-1/2 top-1/2 aspect-9/16 h-3/4 -translate-x-1/2 -translate-y-1/2"
      >
        <video
          class="absolute inset-0 z-10 h-full w-full object-cover"
          playsinline
          autoplay
          muted
        >
          <source src="../assets/secreto_exit.webm" type="video/webm" />
          Your browser does not support the video tag.
        </video>
      </div>
    </div>
    <div
      ref="darkOverlay"
      class="pointer-events-none absolute inset-0 z-20"
      :style="{
        backgroundColor: 'var(--color-background)',
        opacity: 0,
      }"
    ></div>
    <h1
      ref="heading"
      class="absolute left-1/2 top-1/2 z-30 w-full -translate-x-1/2 -translate-y-1/2 p-6 text-center text-white opacity-0 whitespace-pre-wrap [filter:blur(12px)]"
    >
Niet alles is wat het lijkt.<br>  Sommige ervaringen moet je zelf ontdekken.
  </h1>
  </div>
 <div class="grid w-full  grid-cols-2 items-center gap-16 px-40 py-30">
  <div class="flex w-full flex-col gap-6 text-left">
    <h2>
      Het begon met een deur<br />die niemand echt zag
    </h2>

    <p class="text-lg leading-[1.5]">
      In ons eerste restaurant was er een verborgen plek.<br />
      Officieel het appartement van mijn broer.<br />
      Onofficieel een geheime bovenkamer voor wie ons écht kende.
    </p>
  </div>

  <figure
    ref="doorFigure"
    class="relative aspect-[4/5] w-full overflow-hidden rounded-t-full bg-black"
  >
    <img
      ref="doorImage"
      :src="door"
      alt="De verborgen ruimte van Secreto"
      class="h-full w-full object-cover will-change-transform"
    />
  </figure>
</div>
<div class="py-30 px-40">
  <h2 class="text-center text-4xl ">Achter gesloten deuren begon een andere wereld.</h2> </div>
  <div ref="videoSection" class="relative h-screen w-full">
      <video
        ref="scrollVideo"
        class="sticky top-0 ml-auto block h-1/2 w-auto max-w-[calc(100%-3rem)] rotate-180 object-cover sm:mr-10 sm:h-3/5"
        playsinline
        muted
        preload="auto"
      >
        <source src="../assets/secreto_entrance.webm" type="video/webm" />
        Your browser does not support the video tag.
      </video>
    </div>
    <div class="h-1000"></div>
</template>
