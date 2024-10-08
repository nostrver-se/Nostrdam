<script setup>
import { ref } from "vue";
import {
  Dialog,
  DialogPanel,
  Disclosure,
  DisclosureButton,
  DisclosurePanel,
  Popover,
  PopoverButton,
  PopoverGroup,
  PopoverPanel,
  Menu,
  MenuButton,
  MenuItem,
  MenuItems,
} from "@headlessui/vue";
import { SafeIcon } from "@bitcoin-design/bitcoin-icons-vue/outline";
import {
  Bars3Icon,
  XMarkIcon,
  MapIcon,
  ShoppingCartIcon,
  ChevronDownIcon,
} from "@heroicons/vue/24/outline";
const mobileMenuOpen = ref(false);
import data from "~/config/setup";

import menu from "~/config/menu";

import { useProjectStore } from "~/store/shopcart";

const store = useProjectStore();
const totalItems = ref(store.getTotalItems());

import NDK from "@nostr-dev-kit/ndk";
import { bech32 } from "bech32";

const bytesToHex = (bytes) => {
  return Array.from(bytes)
    .map((byte) => byte.toString(16).padStart(2, "0"))
    .join("");
};

const npubToHex = (npub) => {
  const decoded = bech32.decode(npub);
  const pubkeyBytes = bech32.fromWords(decoded.words);
  return bytesToHex(Uint8Array.from(pubkeyBytes));
};

const skHex = npubToHex(data.nostradmin);

const fetchedEvent = ref('');


// Watch for changes in the cartItems array
watch(
  () => store.cartItems,
  () => {
    totalItems.value = store.getTotalItems();
  },
  { deep: true }
);




watch(fetchedEvent, (newVal) => {
  if (newVal) {
    try {
      eventData.value = JSON.parse(newVal);
    } catch (error) {
      console.error('Error parsing event data:', error);
      eventData.value = null;
    }
  }
});



onMounted(async () => {
  totalItems.value = store.getTotalItems();

  const ndk = new NDK({ explicitRelayUrls: data.relays });

  await ndk.connect();

  const filter = { kinds: [0], authors: [skHex] };

  const event = await ndk.fetchEvent(filter);

  if (event && event.content) {
    fetchedEvent.value = event.content;
  } else {
    fetchedEvent.value = 'No event content found';
  }

  //console.log(fetchedEvent.value)

});

const eventData = ref(null);


</script>

<template>
  <header
    class="bg-colorBgLight dark:bg-colorBgDark backdrop-blur-sm fixed w-full z-50 shadow-2xl dark:shadow-gray-700"
  >
    <nav
      class="mx-auto flex max-w-7xl items-center justify-between p-2 lg:px-8"
      aria-label="Global"
    >
      <div class="flex lg:flex-1">
        <NuxtLink :to="localePath('/')" class="flex">
          <span class="sr-only">{{ data.name }}</span>

          <span
            v-if="eventData"
            class="m-2 ml-4 text-2xl font-black dark:text-white uppercase"
            >{{ eventData.name }}</span
          >
        </NuxtLink>

      </div>
      <div class="flex lg:hidden">
        <NuxtLink class="flex m-2 text-black dark:text-white">
          RSVP
        </NuxtLink>
      </div>

      <PopoverGroup class="hidden lg:flex lg:gap-x-12">
        <NuxtLink
          :to="localePath(item.href)"
          v-for="item in menu.Headernavigation.basicmenu"
          :key="item.name"
          class="text-sm font-semibold leading-6 text-black dark:text-white"
        >
          {{ item.name }}</NuxtLink
        >

        <NuxtLink
          v-if="data.blog"
          :to="localePath('/notes')"
          class="text-sm font-semibold leading-6 text-black dark:text-white"
        >
          {{ menu.Headernavigation.bloglabel }}</NuxtLink
        >

        <NuxtLink
          v-if="data.contact"
          class="text-sm font-semibold leading-6 text-black dark:text-white"
          :to="localePath('/contact')"
        >
          Contact
        </NuxtLink>

        <Popover class="relative" v-if="menu.Headernavigation.MenuPopupName">
          <PopoverButton
            class="flex items-center gap-x-1 text-sm font-semibold leading-6 text-black dark:text-white"
          >
            {{ menu.Headernavigation.MenuPopupName }}
            <ChevronDownIcon
              class="h-5 w-5 flex-none text-black dark:text-white"
              aria-hidden="true"
            />
          </PopoverButton>

          <transition
            enter-active-class="transition ease-out duration-200"
            enter-from-class="opacity-0 translate-y-1"
            enter-to-class="opacity-100 translate-y-0"
            leave-active-class="transition ease-in duration-150"
            leave-from-class="opacity-100 translate-y-0"
            leave-to-class="opacity-0 translate-y-1"
          >
            <PopoverPanel
              class="absolute -left-8 top-full z-10 mt-3 w-screen max-w-md overflow-hidden rounded-3xl bg-gray-200 shadow-lg ring-1 ring-gray-900/5"
            >
              <div class="p-4">
                <div
                  v-for="item in menu.Headernavigation.MenuPopup"
                  :key="item.name"
                  class="group relative flex gap-x-6 rounded-lg p-4 text-sm leading-6 hover:bg-gray-50"
                >
                  <div class="flex-auto">
                    <NuxtLink
                      :to="localePath(item.href)"
                      class="block font-semibold text-gray-900"
                    >
                      {{ item.name }}
                      <span class="absolute inset-0" />
                    </NuxtLink>
                    <p class="mt-1 text-gray-600">{{ item.description }}</p>
                  </div>
                </div>
              </div>
              <div
                class="grid grid-cols-2 divide-x divide-gray-900/5 bg-gray-50"
              >
                <NuxtLink
                  v-for="item in callsToAction"
                  :key="item.name"
                  :to="localePath(item.href)"
                  class="flex items-center justify-center gap-x-2.5 p-3 text-sm font-semibold leading-6 text-gray-900 hover:bg-gray-100"
                >
                  <component
                    :is="item.icon"
                    class="h-5 w-5 flex-none text-gray-400"
                    aria-hidden="true"
                  />
                  {{ item.name }}
                </NuxtLink>
              </div>
            </PopoverPanel>
          </transition>
        </Popover>
      </PopoverGroup>
      <div class="hidden lg:flex lg:flex-1 lg:justify-end">
        <span class="text-colorBgLight text-xl">Follow us:</span>
        <a
          :href="'nostr:' +data.nostradmin"
          target="_blank"
          class="text-colorHighLight hover:text-colorHoverHighLight dark:text-colorHighDark dark:hover:text-colorHoverHighDark inline-block mx-3"
        >
          <svg
            version="1.1"
            id="Layer_1"
            xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink"
            x="0px"
            y="0px"
            class="h-7 w-7 fill-colorHighLight hover:fill-colorHoverHighLight dark:fill-colorHighDark hover:fill-colorHoverHighDark inline"
            viewBox="0 0 875 875"
            style="enable-background: new 0 0 875 875"
            xml:space="preserve"
          >
            <path
              class="st0"
              d="M684.7,485.6c0.2,12.6-11.9,51.5-38.7,81.3c-26.7,29.8-56,20.8-58.4,20.2s-3.1-4.5-7.9-3.8s-9.6,6.2-18.9,7.2
      s-17.5,1.7-26.1-1.4c-4.5,0.7-5.1,0.7-7.2,2.2s-17.8,10.8-21.6,11.5c0,7.2-1.4,44.6,0,55.9s3.8,25.7,7.5,36s2.7,10.6,7.5,9.9
      s13.4,0.3,15.8,4.1c2.4,3.8,1.4,6.5,5.5,8.2s60.7,17.1,99.4,19.2c26.7,0.7,42.9,2.7,52.1,19.5c1.4,7.9,7.5,13,11.3,14.1
      s8.2,2.1,12,5.8s1,8.2,5.5,11.7c4.5,3.4,14.7,8.6,25.4,13.7s15.1,13.4,15.8,16.1s1.7,11,1.7,11s-8.9,0-11-2.1s-2.7-5.8-2.7-5.8
      s-6.2,1-7.5,3.4s0.7,2.7-7.9,0.7s-11.7-3.8-18.2-8.6s-16.5-17.1-25-16.8c4.1,8.2,5.8,8.2,10.6,11s8.2,5.8,8.2,5.8l-7.2,4.5
      c0,0-4.5,2.1-14.7-0.7s-11.7-4.5-12.7-10.6s0-9.3-2.7-14.4s-4.1-15.8-22.3-21.3c-18.2-5.5-66.5-21.3-100.1-24.7s-22.6-2.7-28.1-1.4
      s-15.8,4.5-26.4-1.4S481,732,480.3,725.5s-1.7-11,0-19.2s3.4-19.9,1.7-26.7s-14.1-55.9-19.9-64.1c-13,1-50.7-0.7-50.7-0.7
      s-2.4-0.7-17.5,5.8s-36.5,13.8-46.8,19.9s-14.4,9.7-16.1,13.1c0.1,3-1.2,7.7-2.8,9.1s-12.5,2.4-12.5,2.4s-5.9,5.9-8.2,10
      c-6.9,9.6-55.2,125.1-66.5,149.8c-13.5,32.6-9.8,27.4-37.7,27.4s-8.1,0.3-8.1,0.3s-12.3,5.9-16.8,5.9s-18.9-2.4-26.4,0
      s-16.5,9.3-23.3,10.3s-4.9-1.3-8.4-3.7c-4-0.2-14.3-0.1-14.3-0.1s1.7-6.5,7.9-10.9c8.2-5.8,25.4-16.1,34.6-21.3s17.5-7.9,23.3-9.3
      s18.5-6.2,30.5-9.9s19.5-8.2,29.8-31.5s50.4-111.4,51.4-116.2c0.6-3,3.7-6.5,4.8-15.1c0.7-5.3-2.5-13,1.7-22.6
      c11-25,21.6-20.2,26.4-20.2s17.1,0.3,26.4-1.4s15.4-2.7,24.7-7.9s11.3-8.9,11.3-8.9l-19.9-3.4c0,0-18.5,0.7-25-4.5
      s-15.4-15.8-15.4-15.8l-7.5-7.2l1,8.6c0,0-5.1-8.9-6.5-10.3s-8.6-6.5-11.3-11.3s-7.5-25-7.5-25l-6.2,13l-1.7-18.9l-5.1,7.2
      l-2.7-16.1l-4.8,8.2l-3.4-14.4l-5.8,4.5l-2.4-10.3l-5.8-3.4c0,0-14.1-9.3-16.5-9.6s-4.5,3.4-4.5,3.4l1.4,12l-12.2-6.3l-7-11.9
      c0,0,2.4,4-9.6,7.5c-20.6,0-21.9-2.3-24.9-3.9c-1.3-6.6-5.6-10.1-5.6-10.1h-20.6l-0.3-6.9L141,468l0.7-10.3h-14.1l1-11.3h-8.9
      c0,0,3.1-9.3,25.7-23s25-16.5,46.3-17.1c21.3-0.7,32.9,2.7,46.3,8.2s38.7,13.7,43.9,17.5c11.3-9.9,28.5-19.9,34.3-19.9
      c1-2.4,6.2-12.3,18-17.6c35.3-15.8,108.1-34,131.5-35.5c31.2-2.1,7.9-1.4,39.1,2.1s54.2,7.5,69.6,12.7c12.6,4.2,25,9.6,34.3,2.1
      c4.3-1.8,11.8-1.3,17.8-5.1c30.7-25.1,34.7-32.4,43.6-42s20.1-24.9,22.5-45.1s4.5-58.3-10.6-88.1s-28.8-45.3-34.6-69.3
      s-8.2-61-6.2-73s5.1-22.3,6.9-30.5S658.1,7.9,668,6.2c9.9-1.7,17.8,1.4,22.3,4.8c4.5,3.4,11.7,6.3,13.4,10.3
      c0.3,1.7-1.4,6.5,8.2,8.2c9.6,1.7,16,4.2,16,4.2s15.6,4.3,3.1,7.7c-12.7,2.1-20.5-0.7-24.3,1.7s-7.2,10.1-9.6,11.1s-7.2,0.3-12,4.1
      s-9.6,6.9-12.7,14.4s-5.5,15.8-3.4,26.7s8.6,31.5,14.4,43.2s20.2,40.8,24.3,47.7s15.8,29.5,16.8,53.8s1,44.2,0,54.9
      s-10.8,51.6-35.5,85.9c-8.2,14.1-23.2,31.9-24.7,35c-1.5,3.1-3,4.9-1.6,7.6c4.6,9,12.9,22.1,14.7,29.2
      C679.8,463.5,684.5,473,684.7,485.6L684.7,485.6z"
            />
          </svg>
        </a>

      </div>
    </nav>
  </header>
</template>
