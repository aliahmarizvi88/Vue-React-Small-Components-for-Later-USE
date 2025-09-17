<script setup>
import { ref } from 'vue';

const active = ref(null);

// const sections = [
//   { id: 'red', name: 'red', hex: '#dc2626', tw: 'red-600', bg: 'bg-red-600' },
//   {
//     id: 'blue',
//     name: 'blue',
//     hex: '#2563eb',
//     tw: 'blue-600',
//     bg: 'bg-blue-600',
//   },
//   {
//     id: 'green',
//     name: 'green',
//     hex: '#16a34a',
//     tw: 'green-600',
//     bg: 'bg-green-600',
//   },
// ];

const props = defineProps({
  sections: { type: Array, required: true },
});

const toggle = (section) => {
  active.value = active.value === section ? null : section;
};
</script>

<template>
  <div class="flex m-10 w-full gap-2">
    <div
      v-for="section in sections"
      :key="section.id"
      class="relative h-[500px] cursor-pointer transition-all duration-500"
      :class="
        active === section.id
          ? `w-[400px] ${section.bg}`
          : `w-[100px] ${section.bg}`
      "
      @click="toggle(section.id)"
    >
      <div class="absolute bottom-2 right-2 text-right">
        <p class="text-white text-2xl font-extrabold">{{ section.name }}</p>

        <transition name="fade-slide" appear>
          <div v-if="active === section.id" class="mt-1">
            <p class="text-sm text-gray-200">Hex: {{ section.hex }}</p>
            <p class="text-sm text-gray-200">tailwind: {{ section.tw }}</p>
          </div>
        </transition>
      </div>
    </div>
  </div>
</template>

<style scoped>
.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.4s ease;
}
.fade-slide-enter-from {
  opacity: 0;
  transform: translateY(10px);
}
.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(10px);
}
</style>
