<script setup lang="ts">
import { gsap } from "gsap";
import { onMounted, onUnmounted, ref } from "vue";
import heroBlur from "../assets/hero-blur.png";
import heroImage from "../assets/hero.png";
import door from "../assets/door.jpeg";
import secret1 from "../assets/secret-1.jpeg";
import secret2 from "../assets/secret-2.jpeg";
import secret3 from "../assets/secret-3.jpeg";

const background = ref<HTMLElement | null>(null);
const heroSection = ref<HTMLElement | null>(null);
const darkOverlay = ref<HTMLElement | null>(null);
const videoOverlay = ref<HTMLElement | null>(null);
const heading = ref<HTMLElement | null>(null);
const scrollHeadingSection = ref<HTMLElement | null>(null);
const scrollHeading = ref<HTMLElement | null>(null);
const videoSection = ref<HTMLElement | null>(null);
const scrollVideo = ref<HTMLVideoElement | null>(null);
const doorFigure = ref<HTMLElement | null>(null);
const doorImage = ref<HTMLImageElement | null>(null);
const secretSection = ref<HTMLElement | null>(null);
const secretCards = ref<HTMLElement[]>([]);
const secretText = ref<HTMLElement[]>([]);
const secretImages = [secret1, secret2, secret3];

let zoomTimer: ReturnType<typeof setTimeout> | undefined;
let zoomAnimation: ReturnType<typeof gsap.to> | undefined;
let textAnimation: gsap.core.Tween | undefined;
let videoFrame: number | undefined;
let heroFrame: number | undefined;
let doorFrame: number | undefined;
let scrollHeadingFrame: number | undefined;
let secretFrame: number | undefined;
let secretDelayTimer: ReturnType<typeof setTimeout> | undefined;
let secretDelayUntil: number | undefined;
let videoCleanup: (() => void) | undefined;
let heroCleanup: (() => void) | undefined;
let doorCleanup: (() => void) | undefined;
let scrollHeadingCleanup: (() => void) | undefined;
let secretCleanup: (() => void) | undefined;
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

  const updateScrollHeading = () => {
    scrollHeadingFrame = undefined;

    const section = scrollHeadingSection.value;
    if (!section) return;

    const { top } = section.getBoundingClientRect();
    const start = window.innerHeight * 0.5;
    const end = -window.innerHeight * 0.5;
    const progress = Math.min(1, Math.max(0, (start - top) / (start - end)));

    scrollHeading.value?.style.setProperty(
      "--fill-progress",
      `${progress * 100}%`,
    );
  };

  const requestScrollHeadingUpdate = () => {
    if (scrollHeadingFrame === undefined) {
      scrollHeadingFrame = requestAnimationFrame(updateScrollHeading);
    }
  };

  const updateSecretCards = () => {
    secretFrame = undefined;

    const section = secretSection.value;
    if (!section) return;

    const sectionRect = section.getBoundingClientRect();
    const scrollRange = Math.max(1, sectionRect.height - window.innerHeight);
    const rawProgress = Math.min(
      1,
      Math.max(0, -sectionRect.top / scrollRange),
    );
    if (sectionRect.top > 0) {
      secretDelayUntil = undefined;
    } else if (secretDelayUntil === undefined) {
      secretDelayUntil = performance.now() + 200;
      secretDelayTimer = setTimeout(requestSecretUpdate, 200);
    }

    const progress =
      secretDelayUntil !== undefined && performance.now() < secretDelayUntil
        ? 0
        : rawProgress;
    const animationProgress = progress * secretCards.value.length;
    const stackProgress = secretCards.value.reduce(
      (cardProgress, _, index) =>
        cardProgress +
        (index === 0
          ? 0
          : Math.min(1, Math.max(0, animationProgress - index))),
      0,
    );

    secretCards.value.forEach((card, index) => {
      const distance = index - stackProgress;
      const behindProgress = Math.min(1, Math.max(0, -distance));
      const enteringProgress = Math.min(1, Math.max(0, 1 - distance));
      const translateY = distance > 0 ? (1 - enteringProgress) * 100 : -behindProgress * 4;
      const scale = distance <= 0 ? 1 - behindProgress * 0.16 : 1;

      card.style.transform = `translate3d(0, ${translateY}%, 0) scale(${scale})`;
      card.style.zIndex = String(index);
      card.style.opacity = distance > 1 ? "0" : "1";
    });

    secretText.value.forEach((text, index) => {
      const phaseProgress = Math.min(
        1,
        Math.max(0, animationProgress - index),
      );
      const fillProgress = phaseProgress * phaseProgress;
      text.style.setProperty("--fill-progress", `${fillProgress * 100}%`);
      text.style.opacity = "1";
    });
  };

  const requestSecretUpdate = () => {
    if (secretFrame === undefined) {
      secretFrame = requestAnimationFrame(updateSecretCards);
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

  window.addEventListener("scroll", requestScrollHeadingUpdate, { passive: true });
  window.addEventListener("resize", requestScrollHeadingUpdate);
  requestScrollHeadingUpdate();

  scrollHeadingCleanup = () => {
    window.removeEventListener("scroll", requestScrollHeadingUpdate);
    window.removeEventListener("resize", requestScrollHeadingUpdate);
    if (scrollHeadingFrame !== undefined) {
      cancelAnimationFrame(scrollHeadingFrame);
    }
  };

  window.addEventListener("scroll", requestSecretUpdate, { passive: true });
  window.addEventListener("resize", requestSecretUpdate);
  requestSecretUpdate();

  secretCleanup = () => {
    window.removeEventListener("scroll", requestSecretUpdate);
    window.removeEventListener("resize", requestSecretUpdate);
    if (secretFrame !== undefined) cancelAnimationFrame(secretFrame);
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
  if (secretDelayTimer) clearTimeout(secretDelayTimer);
  zoomAnimation?.kill();
  textAnimation?.kill();
  heroCleanup?.();
  doorCleanup?.();
  scrollHeadingCleanup?.();
  secretCleanup?.();
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
<div ref="scrollHeadingSection" class="min-h-[150vh] px-40">
  <h2
    ref="scrollHeading"
    class="scroll-fill-heading sticky top-0 flex h-screen items-center justify-center text-center text-4xl"
  >
    Achter gesloten deuren begon een andere wereld.
  </h2>
</div>
<div ref="secretSection" class="relative h-[240vh]">
  <div class="sticky top-0 grid h-screen grid-cols-1 items-center gap-10 px-6 py-12 lg:grid-cols-[minmax(0,1fr)_minmax(16rem,0.7fr)] lg:gap-16 lg:px-40">
    <div class="relative mx-auto aspect-square w-full max-w-2xl overflow-hidden">
      <img
        v-for="(image, index) in secretImages"
        :key="image"
        :ref="(element) => { if (element) secretCards[index] = element as HTMLElement }"
        :src="image"
        :alt="`De verborgen ruimte van Secreto, beeld ${index + 1}`"
        class="absolute inset-0 h-full w-full origin-center object-cover will-change-transform"
      />
    </div>
    <div class="flex flex-col">
      <p
        v-for="(text, index) in ['Een andere kaart.', 'Bijzondere wijnen.', 'Een intiemere sfeer.']"
        :key="text"
        :ref="(element) => { if (element) secretText[index] = element as HTMLElement }"
        class="scroll-fill-heading transition-opacity duration-300"
      >
        {{ text }}
      </p>
    </div>
  </div>
</div>
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

<style scoped>
.scroll-fill-heading {
  --fill-progress: 0%;
  color: transparent;
  background: linear-gradient(
    to right,
    var(--color-heading) 0%,
    var(--color-heading) var(--fill-progress),
    color-mix(in srgb, var(--color-heading) 40%, transparent) var(--fill-progress),
    color-mix(in srgb, var(--color-heading) 40%, transparent) 100%
  );
  background-clip: text;
  -webkit-background-clip: text;
}
</style>
