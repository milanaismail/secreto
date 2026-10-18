<script setup lang="ts">
import { gsap } from "gsap";
import { onMounted, onUnmounted, ref } from "vue";
import HelloWorld from "./components/HelloWorld.vue";

const headerContent = ref<HTMLElement | null>(null);
let headerTimer: ReturnType<typeof setTimeout> | undefined;
let headerAnimation: ReturnType<typeof gsap.to> | undefined;

onMounted(() => {
  if (!headerContent.value) return;

  gsap.set(headerContent.value, { opacity: 0 });
  headerTimer = setTimeout(() => {
    if (headerContent.value) {
      headerAnimation = gsap.to(headerContent.value, {
        opacity: 1,
        duration: 1.8,
        ease: "sine.out",
      });
    }
  }, 4200);
});

onUnmounted(() => {
  if (headerTimer) clearTimeout(headerTimer);
  headerAnimation?.kill();
});
</script>

<template>
  <header>
 <div class="absolute top-0 z-100 w-full">
  <div ref="headerContent" class="flex items-center justify-between px-16 py-6">
  <img src="@/assets/logo.png" alt="Logo" class="logo" />
  <div class="grid grid-cols-4 gap-4 justify-center items-center">
    <a class="navigation" href="">Menu</a>
    <a class="navigation" href="">Contact</a>
    <a class="navigation" href="">NL</a>
    <button class="glass-button">Reserveer</button>
  </div>
  </div>
 </div>
  </header>
  <div class="wrapper">
    <HelloWorld />
  </div>

  <main>
    <TheWelcome />
  </main>
</template>

<style scoped>
.navigation{
  color: var(--vt-c-text-light-1);
  text-decoration: none;
  font-size: 1.2rem;
  text-align: center;
}
</style>
