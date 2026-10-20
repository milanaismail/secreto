<script setup lang="ts">
import { gsap } from "gsap";
import { onMounted, onUnmounted, ref } from "vue";
import HelloWorld from "./components/HelloWorld.vue";

const headerContent = ref<HTMLElement | null>(null);
let headerTimer: ReturnType<typeof setTimeout> | undefined;
let headerAnimation: ReturnType<typeof gsap.to> | undefined;
const name = ref("");
const email = ref("");
const message = ref("");
const status = ref("");
const isSubmitting = ref(false);

const submitForm = async () => {
  isSubmitting.value = true;
  status.value = "Sending...";

  try {
    const response = await fetch("https://api.web3forms.com/submit", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        access_key: "YOUR_ACCESS_KEY_HERE", // Replace with your free Web3Forms access key
        email: email.value,
      }),
    });

    const result = await response.json();
    if (result.success) {
      status.value = "Email sent successfully!";
      email.value = "";
    } else {
      status.value = result.message || "Something went wrong.";
    }
  } catch (error) {
    status.value = "Error sending email. Please try again.";
  } finally {
    isSubmitting.value = false;
  }
};
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
      <div
        ref="headerContent"
        class="flex items-center justify-between px-16 py-6"
      >
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
  <footer class=" pt-40 ">
   
    <div
      class="bg-(--color-background-burgundy) py-20 px-16 border-t-l-rounded rounded-t-4xl flex gap-20 flex-col"
    >
      <div class="grid grid-cols-2">
        <img src="./assets/secreto-logo-full.png" alt="Logo" class="w-60" />
        <div class="flex flex-col gap-6">
         <div class="flex flex-col gap-2">
          <h2>Ontdek het geheim</h2>
          <p>
            Ontdek als eerste onze exclusieve evenementen, bijzondere avonden en
            alles wat zich achter de schermen van Secreto afspeelt.
          </p>
          </div>
          <form @submit.prevent="submitForm">
            <div class="flex gap-4">
              <input
                class="bg-(--color-background-white) w-full h-auto"
                id="email"
                type="email"
                v-model="email"
                required
              />

              <button
                class="white-button"
                type="submit"
                :disabled="isSubmitting"
              >
                {{ isSubmitting ? "Sending..." : "Inschrijven" }}
              </button>
            </div>

            <p v-if="status">{{ status }}</p>
          </form>
        </div>
      </div>
      <hr class="border bg-(--color-background-white) opacity-40"/>
      <div class="grid grid-cols-4 gap-6">
        <div>
          <p class="font-bold ">Contact</p>
          <p>+32 33 44 79 18</p>
          <p>info@secretoantwerp.be</p>
        </div>
        <div>
          <p class="font-bold ">Locatie</p>
          <p>Secreto</p>
          <p>
            Vlaamsekaai 14<br />
            2000 Antwerpen
          </p>
        </div>
        <div>
          <p class="font-bold ">Openingsuren</p>
          <p>
            maandag tot dinsdag:<br />
            gesloten
          </p>
          <p>
            woensdag tot zondag:<br />
            17:30 - 01:00
          </p>
        </div>
        <div>
          <p class="font-bold ">Volg ons</p>
          <div class="flex gap-4 mt-2">
            <a href="https://www.instagram.com/secretoantwerp/" target="_blank" class="rounded-full  w-10 h-10 flex items-center justify-center  border-2 border-(-color--background-white">
              <svg class="h-5 w-auto" xmlns="http://www.w3.org/2000/svg" width="10" height="20" viewBox="0 0 10 20" fill="none">
<path fill-rule="evenodd" clip-rule="evenodd" d="M2.08077 9.68357V18.967C2.08077 19.1271 2.1608 19.2071 2.32086 19.2071H6.00221C6.16227 19.2071 6.2423 19.1271 6.2423 18.967V9.52351H8.96331C9.12337 9.52351 9.2034 9.44348 9.2034 9.28343L9.44348 6.40236C9.44348 6.2423 9.36345 6.16227 9.2034 6.16227H6.2423V4.16154C6.2423 3.68136 6.64245 3.28121 7.20266 3.28121H9.28343C9.52351 3.28121 9.60354 3.20118 9.60354 3.04112V0.240089C9.60354 0.0800297 9.52351 0 9.36346 0H5.84216C3.76139 0 2.08077 1.52056 2.08077 3.44127V6.16227H0.240089C0.0800297 6.16227 0 6.2423 0 6.40236V9.28343C0 9.44348 0.0800297 9.52351 0.240089 9.52351H2.08077V9.68357Z" fill="#F4E7D8"/>
</svg>
            </a>
            <a href="https://www.instagram.com/secretoantwerp/" target="_blank" class="rounded-full  w-10 h-10 flex items-center justify-center  border-2 border-(-color--background-white">
               <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 20 20" fill="none">
<path d="M14.9918 3.53662C14.3364 3.53662 13.8448 4.01888 13.8448 4.66189C13.8448 5.30491 14.3364 5.78717 14.9918 5.78717C15.6471 5.78717 16.1387 5.30491 16.1387 4.66189C16.1387 4.01888 15.6471 3.53662 14.9918 3.53662Z" fill="#F4E7D8"/>
<path d="M9.83073 4.9834C7.12729 4.9834 4.99731 7.15357 4.99731 9.72562C4.99731 12.2977 7.20922 14.4678 9.83073 14.4678C12.4522 14.4678 14.6641 12.2977 14.6641 9.72562C14.6641 7.15357 12.5342 4.9834 9.83073 4.9834ZM9.83073 12.7799C8.11036 12.7799 6.71768 11.4135 6.71768 9.72562C6.71768 8.03771 8.11036 6.67131 9.83073 6.67131C11.5511 6.67131 12.9438 8.03771 12.9438 9.72562C12.9438 11.4135 11.5511 12.7799 9.83073 12.7799Z" fill="#F4E7D8"/>
<path d="M13.7629 0H5.98033C2.62151 0 0 2.57205 0 5.78711V13.4229C0 16.6379 2.62151 19.21 5.8984 19.21H13.681C16.9579 19.21 19.5794 16.6379 19.5794 13.4229V5.78711C19.6613 2.57205 17.0398 0 13.7629 0ZM17.7771 13.5033C17.7771 15.6734 15.9748 17.5221 13.681 17.5221H5.8984C3.6865 17.5221 1.80229 15.7538 1.80229 13.5033V5.86749C1.80229 3.69732 3.60458 1.84866 5.8984 1.84866H13.681C15.8929 1.84866 17.7771 3.61695 17.7771 5.86749V13.5033Z" fill="#F4E7D8"/>
</svg>
            </a>
          </div>
         
        
        </div>
      </div>
    </div>
  </footer>
</template>

<style scoped>
.navigation {
  color: var(--vt-c-text-light-1);
  text-decoration: none;
  font-size: 1.2rem;
  text-align: center;
}
</style>
