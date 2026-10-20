<script setup lang="ts">
import { gsap } from "gsap";
import { onMounted, onUnmounted, ref } from "vue";
import heroImage from "../assets/hero.png";
import ctaImage from "../assets/hero.png";
import door from "../assets/door.jpeg";
import secret1 from "../assets/secret-1.jpeg";
import secret2 from "../assets/secret-2.jpeg";
import secret3 from "../assets/secret-3.jpeg";
import safariHeroVideo from "../assets/secreto_exit_safari_smooth.mov";
import webmHeroVideo from "../assets/secreto_exit.webm";
import nameBackground from "../assets/zo-onstond-de-naam.jpeg";

const background = ref<HTMLElement | null>(null);
const heroSection = ref<HTMLElement | null>(null);
const darkOverlay = ref<HTMLElement | null>(null);
const videoOverlay = ref<HTMLElement | null>(null);
const heroVideo = ref<HTMLVideoElement | null>(null);
const heading = ref<HTMLElement | null>(null);
const scrollHeadingSection = ref<HTMLElement | null>(null);
const scrollHeading = ref<HTMLElement | null>(null);
const videoSection = ref<HTMLElement | null>(null);
const scrollVideo = ref<HTMLVideoElement | null>(null);
const foodSection = ref<HTMLElement | null>(null);
const foodGalleryViewport = ref<HTMLElement | null>(null);
const foodGalleryTrack = ref<HTMLElement | null>(null);
const doorFigure = ref<HTMLElement | null>(null);
const doorImage = ref<HTMLImageElement | null>(null);
const secretSection = ref<HTMLElement | null>(null);
const secretCards = ref<HTMLElement[]>([]);
const secretText = ref<HTMLElement[]>([]);
const nameSection = ref<HTMLElement | null>(null);
const nameItems = ref<HTMLElement[]>([]);
const ctaSection = ref<HTMLElement | null>(null);
const secretImages = [secret1, secret2, secret3];
const isSafari =
  typeof navigator !== "undefined" &&
  /Safari/.test(navigator.userAgent) &&
  !/Chrome|Chromium|CriOS|FxiOS|EdgiOS|OPiOS|Android/.test(
    navigator.userAgent,
  );
const heroVideoSource = isSafari ? safariHeroVideo : webmHeroVideo;

let zoomTimer: ReturnType<typeof setTimeout> | undefined;
let zoomAnimation: ReturnType<typeof gsap.to> | undefined;
let textAnimation: gsap.core.Tween | undefined;
let videoFrame: number | undefined;
let foodFrame: number | undefined;
let heroFrame: number | undefined;
let doorFrame: number | undefined;
let scrollHeadingFrame: number | undefined;
let secretFrame: number | undefined;
let nameFrame: number | undefined;
let ctaFrame: number | undefined;
let secretDelayTimer: ReturnType<typeof setTimeout> | undefined;
let secretDelayUntil: number | undefined;
let videoCleanup: (() => void) | undefined;
let foodCleanup: (() => void) | undefined;
let heroCleanup: (() => void) | undefined;
let doorCleanup: (() => void) | undefined;
let scrollHeadingCleanup: (() => void) | undefined;
let secretCleanup: (() => void) | undefined;
let nameCleanup: (() => void) | undefined;
let ctaCleanup: (() => void) | undefined;
let heroVideoCleanup: (() => void) | undefined;
let videoReady = false;
let scrollLocked = false;
let previousBodyOverflow = "";
let previousDocumentOverflow = "";

const lockPageScroll = () => {
  if (scrollLocked) return;

  previousBodyOverflow = document.body.style.overflow;
  previousDocumentOverflow = document.documentElement.style.overflow;
  document.body.style.overflow = "hidden";
  document.documentElement.style.overflow = "hidden";
  scrollLocked = true;
};

const unlockPageScroll = () => {
  if (!scrollLocked) return;

  document.body.style.overflow = previousBodyOverflow;
  document.documentElement.style.overflow = previousDocumentOverflow;
  scrollLocked = false;
};

onMounted(() => {
  const shouldPlayHeroIntro = window.scrollY <= 1;
  let heroIntroDelayElapsed = false;
  let heroIntroStarted = false;

  if (heading.value) {
    gsap.set(heading.value, {
      autoAlpha: 0,
      y: 35,
      filter: "blur(12px)",
    });
  }

  const finishHeroIntro = () => {
    const clipTargets = [background.value, videoOverlay.value].filter(
      (element): element is HTMLElement => element !== null,
    );

    clipTargets.forEach((element) => {
      gsap.set(element, { clipPath: "inset(0% 0% round 0vw)" });
    });

    if (heading.value) {
      gsap.set(heading.value, {
        autoAlpha: 1,
        y: 0,
        filter: "blur(0px)",
      });
    }
  };

  if (shouldPlayHeroIntro) {
    lockPageScroll();
    const startHeroIntro = () => {
      if (heroIntroStarted || !heroIntroDelayElapsed) return;
      if (
        heroVideo.value &&
        heroVideo.value.readyState < HTMLMediaElement.HAVE_FUTURE_DATA
      ) {
        return;
      }

      heroIntroStarted = true;
      const clipTargets = [background.value, videoOverlay.value].filter(
        (element): element is HTMLElement => element !== null,
      );

      if (clipTargets.length > 0) {
        zoomAnimation = gsap.to(clipTargets, {
          clipPath: "inset(0% 0% round 0vw)",
          duration: 2.5,
          ease: "power4.inOut",
          onComplete: unlockPageScroll,
        });
      } else {
        unlockPageScroll();
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
    };

    zoomTimer = setTimeout(() => {
      heroIntroDelayElapsed = true;
      startHeroIntro();
    }, 3500);

    if (heroVideo.value) {
      const startWhenReady = () => startHeroIntro();
      heroVideo.value.addEventListener("canplay", startWhenReady);
      heroVideoCleanup = () => {
        heroVideo.value?.removeEventListener("canplay", startWhenReady);
      };
    }
  } else {
    finishHeroIntro();
  }

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
    const scrollRange = Math.max(1, sectionRect.height - videoRect.height);
    const progress = Math.min(1, Math.max(0, -sectionRect.top / scrollRange));
    video.currentTime = progress * video.duration;
  };

  const updateHeroBackground = () => {
    heroFrame = undefined;

    const section = heroSection.value;
    const overlay = darkOverlay.value;
    if (!section || !overlay) return;

    const sectionRect = section.getBoundingClientRect();
    const fadeStart = sectionRect.height / 2;
    const progress = Math.min(1, Math.max(0, -sectionRect.top / fadeStart - 1));
    overlay.style.opacity = String(progress);
  };

  const requestVideoUpdate = () => {
    if (videoFrame === undefined) {
      videoFrame = requestAnimationFrame(updateVideo);
    }
  };

  const updateFoodGallery = () => {
    foodFrame = undefined;

    const section = foodSection.value;
    const viewport = foodGalleryViewport.value;
    const track = foodGalleryTrack.value;
    if (!section || !viewport || !track) return;

    const sectionRect = section.getBoundingClientRect();
    const scrollRange = Math.max(1, sectionRect.height);
    const progress = Math.min(
      1,
      Math.max(0, (window.innerHeight - sectionRect.top) / scrollRange),
    );
    const maxTranslate = Math.max(0, track.scrollWidth - viewport.clientWidth);
    const lastImage = track.lastElementChild;
    const lastImageWidth =
      lastImage instanceof HTMLElement
        ? lastImage.getBoundingClientRect().width
        : 0;
    const targetTranslate = Math.max(0, maxTranslate - lastImageWidth / 2);

    track.style.transform = `translate3d(${-targetTranslate * progress}px, 0, 0)`;
  };

  const requestFoodUpdate = () => {
    if (foodFrame === undefined) {
      foodFrame = requestAnimationFrame(updateFoodGallery);
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

    const { top, height } = section.getBoundingClientRect();
    const scrollRange = Math.max(1, height - window.innerHeight);
    const progress = Math.min(1, Math.max(0, -top / scrollRange));

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
        (index === 0 ? 0 : Math.min(1, Math.max(0, animationProgress - index))),
      0,
    );

    secretCards.value.forEach((card, index) => {
      const distance = index - stackProgress;
      const behindProgress = Math.min(1, Math.max(0, -distance));
      const enteringProgress = Math.min(1, Math.max(0, 1 - distance));
      const translateY =
        distance > 0 ? (1 - enteringProgress) * 100 : -behindProgress * 4;
      const scale = distance <= 0 ? 1 - behindProgress * 0.16 : 1;

      card.style.transform = `translate3d(0, ${translateY}%, 0) scale(${scale})`;
      card.style.zIndex = String(index);
      card.style.opacity = distance > 1 ? "0" : "1";
    });

    secretText.value.forEach((text, index) => {
      const phaseProgress = Math.min(1, Math.max(0, animationProgress - index));
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

  const updateNameItems = () => {
    nameFrame = undefined;

    const section = nameSection.value;
    if (!section) return;

    const { top, height } = section.getBoundingClientRect();
    const scrollRange = Math.max(1, height - window.innerHeight);
    const progress = Math.min(1, Math.max(0, -top / scrollRange));
    const animationProgress = progress * nameItems.value.length;

    nameItems.value.forEach((item, index) => {
      const itemProgress = Math.min(1, Math.max(0, animationProgress - index));
      const easedProgress =
        itemProgress * itemProgress * (3 - 2 * itemProgress);
      const nextProgress = Math.min(
        1,
        Math.max(0, animationProgress - index - 1),
      );
      const easedNextProgress =
        nextProgress * nextProgress * (3 - 2 * nextProgress);
      const outgoingOpacity = 1 - Math.min(1, easedNextProgress * 1.6);
      item.style.setProperty("--fill-progress", `${easedProgress * 100}%`);
      const revealOpacity = index === 0 ? 1 : easedProgress;
      item.style.opacity = String(revealOpacity * outgoingOpacity);
    });
  };

  const requestNameUpdate = () => {
    if (nameFrame === undefined) {
      nameFrame = requestAnimationFrame(updateNameItems);
    }
  };

  const updateCtaPadding = () => {
    ctaFrame = undefined;

    const section = ctaSection.value;
    if (!section) return;

    const { top, height } = section.getBoundingClientRect();
    const scrollRange = Math.max(1, window.innerHeight);
    const progress = Math.min(
      1,
      Math.max(0, (window.innerHeight - top) / scrollRange),
    );
    const padding = (1 - progress) * 1.5;

    section.style.setProperty("--cta-padding", `${padding}rem`);
  };

  const requestCtaUpdate = () => {
    if (ctaFrame === undefined) {
      ctaFrame = requestAnimationFrame(updateCtaPadding);
    }
  };

  window.addEventListener("scroll", requestHeroUpdate, { passive: true });
  window.addEventListener("resize", requestHeroUpdate);
  requestHeroUpdate();

  window.addEventListener("scroll", requestFoodUpdate, { passive: true });
  window.addEventListener("resize", requestFoodUpdate);
  requestFoodUpdate();

  foodCleanup = () => {
    window.removeEventListener("scroll", requestFoodUpdate);
    window.removeEventListener("resize", requestFoodUpdate);
    if (foodFrame !== undefined) cancelAnimationFrame(foodFrame);
  };

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

  window.addEventListener("scroll", requestScrollHeadingUpdate, {
    passive: true,
  });
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

  window.addEventListener("scroll", requestNameUpdate, { passive: true });
  window.addEventListener("resize", requestNameUpdate);
  requestNameUpdate();

  nameCleanup = () => {
    window.removeEventListener("scroll", requestNameUpdate);
    window.removeEventListener("resize", requestNameUpdate);
    if (nameFrame !== undefined) cancelAnimationFrame(nameFrame);
  };

  window.addEventListener("scroll", requestCtaUpdate, { passive: true });
  window.addEventListener("resize", requestCtaUpdate);
  requestCtaUpdate();

  ctaCleanup = () => {
    window.removeEventListener("scroll", requestCtaUpdate);
    window.removeEventListener("resize", requestCtaUpdate);
    if (ctaFrame !== undefined) cancelAnimationFrame(ctaFrame);
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
  unlockPageScroll();
  heroCleanup?.();
  foodCleanup?.();
  doorCleanup?.();
  scrollHeadingCleanup?.();
  secretCleanup?.();
  nameCleanup?.();
  ctaCleanup?.();
  heroVideoCleanup?.();
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
    <div
      class="bg-background absolute inset-0 block h-full w-full object-cover"
    ></div>
    <div
      ref="background"
      class="absolute inset-0 bg-cover bg-center"
      :style="{
        backgroundImage: `url(${heroImage})`,
        clipPath: 'inset(22% 41% round 23vw)',
      }"
    ></div>
    <div
      ref="videoOverlay"
      class="absolute inset-0"
      :style="{ clipPath: 'inset(22% 41% round 23vw)' }"
    >
      <div
        class="absolute left-1/2 top-1/2 aspect-9/16 h-3/4 -translate-x-1/2 -translate-y-1/2"
      >
        <video
          ref="heroVideo"
          class="absolute inset-0 z-10 h-full w-full object-cover"
          playsinline
          autoplay
          muted
          preload="auto"
          :src="heroVideoSource"
        >
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
      class="absolute inset-x-0 top-1/2 z-30 w-full -translate-y-1/2 p-6 text-center text-white opacity-0 [filter:blur(12px)]"
    >
      Niet alles is wat het lijkt.<br />
      Sommige ervaringen moet je zelf ontdekken.
    </h1>
  </div>
  <div class="grid w-full grid-cols-2 items-center gap-16 px-40 py-30">
    <div class="flex w-full flex-col gap-6 text-left">
      <h2>Het begon met een deur<br />die niemand echt zag</h2>

      <p class="text-lg leading-[2]">
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
  <div ref="secretSection" class="relative h-[240vh] my-30">
    <div
      class="sticky top-0 grid h-screen grid-cols-1 items-center gap-10 px-6 py-12 lg:grid-cols-[minmax(0,1fr)_minmax(16rem,0.7fr)] lg:gap-16 lg:px-40"
    >
      <div
        class="relative mx-auto aspect-square w-full max-w-2xl overflow-hidden"
      >
        <img
          v-for="(image, index) in secretImages"
          :key="image"
          :ref="
            (element) => {
              if (element) secretCards[index] = element as HTMLElement;
            }
          "
          :src="image"
          :alt="`De verborgen ruimte van Secreto, beeld ${index + 1}`"
          class="absolute inset-0 h-full w-full origin-center object-cover will-change-transform"
        />
      </div>
      <div class="flex flex-col gap-2">
        <p
          v-for="(text, index) in [
            'Een andere kaart.',
            'Bijzondere wijnen.',
            'Een intiemere sfeer.',
          ]"
          :key="text"
          :ref="
            (element) => {
              if (element) secretText[index] = element as HTMLElement;
            }
          "
          class="scroll-fill-heading transition-opacity duration-300 text-2xl"
        >
          {{ text }}
        </p>
      </div>
    </div>
  </div>
  <div
    ref="nameSection"
    class="relative min-h-[300vh] w-full"
  >
    <div class="sticky top-0 h-screen w-full">
      <div
        class="absolute inset-0 bg-cover bg-center"
        :style="{
          backgroundImage: `linear-gradient(rgba(18, 9, 2, 0.52), rgba(18, 9, 2, 0.52)), url(${nameBackground})`,
        }"
      ></div>
      <div
        class="absolute inset-0 z-10 flex h-screen w-full items-center justify-center px-40 py-30"
      >
        <h2
          :ref="
            (element) => {
              if (element) nameItems[0] = element as HTMLElement;
            }
          "
          class="name-scroll-item scroll-fill-heading"
        >
          zo ontstond de naam
        </h2>
      </div>
      <div
        class="absolute inset-0 z-20 flex h-screen w-full items-center justify-center px-40 py-30"
      >
        <img
          :ref="
            (element) => {
              if (element) nameItems[1] = element as HTMLElement;
            }
          "
          src="../assets/secreto-logo-full.png"
          class="name-scroll-item h-auto w-80"
          alt="Logo"
        />
      </div>
      <div
        class="absolute inset-0 z-30 flex h-screen w-full items-center justify-center px-40 py-30"
      >
        <h2
          :ref="
            (element) => {
              if (element) nameItems[2] = element as HTMLElement;
            }
          "
          class="name-scroll-item scroll-fill-heading"
        >
          omdat het mooiste soms verborgen zit.
        </h2>
      </div>
    </div>
  </div>

  <!-- <div ref="videoSection" class="relative h-screen w-full bg-[var(--color-background-white)]">
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
    </div> -->
    <div class="my-20">
  <div
    ref="foodSection"
    class="relative h-screen "
  >
    <div class="flex h-full flex-col justify-center overflow-hidden py-6">
      <div ref="foodGalleryViewport" class="w-full overflow-hidden">
        <div
          ref="foodGalleryTrack"
          class="grid w-[calc(125%+0.25rem)] grid-cols-5 gap-4 will-change-transform"
        >
          <img
            src="../assets/food-1.jpeg"
            alt="Secreto gerecht 1"
            class="h-auto w-full"
          />
          <img
            src="../assets/food-2.jpeg"
            alt="Secreto gerecht 2"
            class="h-auto w-full"
          />
          <img
            src="../assets/food-3.jpeg"
            alt="Secreto gerecht 3"
            class="h-auto w-full"
          />
          <img
            src="../assets/food-4.jpeg"
            alt="Secreto gerecht 4"
            class="h-auto w-full"
          />
          <img
            src="../assets/food-5.jpeg"
            alt="Secreto gerecht 5"
            class="h-auto w-full"
          />
        </div>
      </div>
    </div>
    </div>
  </div>
  <div
    ref="ctaSection"
    class="cta-section h-screen )"
  >
    <div
      class="w-full h-full bg-cover bg-center flex flex-col items-center justify-center gap-6"
      :style="{
        backgroundImage: `url(${ctaImage})`,
      }"
    >
      <h2>Sommige geheimen moet je zelf ontdekken</h2>
      <button class="glass-button">Reserveer nu</button>
    </div>
  </div>
</template>

<style scoped>
.cta-section {
  --cta-padding: 1.5rem;
  padding: var(--cta-padding);
}

.scroll-fill-heading {
  --fill-progress: 0%;
  color: transparent;
  background: linear-gradient(
    to right,
    var(--color-heading) 0%,
    var(--color-heading) var(--fill-progress),
    color-mix(in srgb, var(--color-heading) 40%, transparent)
      var(--fill-progress),
    color-mix(in srgb, var(--color-heading) 40%, transparent) 100%
  );
  background-clip: text;
  -webkit-background-clip: text;
}

.name-scroll-item {
  --fill-progress: 0%;
  opacity: 0;
  will-change: opacity;
}
</style>
