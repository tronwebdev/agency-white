<template> 
  <div> 
    <transition name="fade">
      <div id="loading" v-if="!data" class="flex w-full gap-[2rem] justify-center items-center bg-white h-full fixed z-[70000]">
        <div class="w-[5rem] border-[2rem] aspect-square border-red-500 border-solid"></div>
        <p class="font-bold ">Loading...</p>
      </div>

      <main v-else-if="previewMode || data" class="overflow-y-scroll"> 
        <NavBar :data="data" />
        <Hero :data="data" />
        <Consultation :data="data" />
        <Help :data="data" />
        <Services :data="data" /> 
        <Benefits :data="data" /> 
        <About :data="data" />
        <Testimonials :data="data" />
        <XFooter :data="data" />
      </main>
    </transition>
  </div>
</template>

<script setup lang="ts">
import { WizardResponse } from '~/type';

const previewMode = ref(false);
const data = ref<WizardResponse>();
const config = useRuntimeConfig();
const route = useRoute();

onBeforeMount(async () => {
  try {
    const [subdomain,_] = window.location.host.split(".");
    console.table({subdomain});
    
    const request = await fetch(`${config.public.baseURL}/d/wizards/${subdomain.split("-").at(-1)}`);
    data.value = await request.json();
    if (route.query.preview === "true") {
      previewMode.value = true;
    }
  } catch (err) {
    navigateTo('/404')
  }
})

useHead(() => ({
  title: data.value?.agency_wizard.seo.title || "Welcome to Agency Pages",
  link: [
    {
      href: data.value?.agency_wizard.website_details.favIcon,
      type: "image/png",
      rel: "icon",
    },
  ],
  ...( data.value?.agency_wizard.website_details.customJavascript 
  && !((data.value?.agency_wizard.website_details.customJavascript as string)?.includes('<script')) ? {
    script: [
    {
      innerHTML:
        data.value?.agency_wizard.website_details.customJavascript || "null",
      type: "text/javascript",
    },
  ],
  }: {})
}));

onMounted(() => {
  const div = document.createElement("div");
  const customJavascript = data.value?.agency_wizard.website_details.customJavascript as any;
  div.innerHTML = customJavascript;

  if (customJavascript && customJavascript.includes('<script')) {
    document.body.prepend(div);
  }
})
</script>
<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
