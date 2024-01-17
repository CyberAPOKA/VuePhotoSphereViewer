<script setup>
import { RouterLink, RouterView } from 'vue-router'
import favIcon from '@/assets/favicon.png'

</script>

<template>
  <nav>
    <div class="flex flex-wrap items-center justify-between mx-auto px-4 py-3 border-b border-gray-300 md:h-20">

      <div class="flex justify-center items-center gap-4">
        <img :src="favIcon" alt="fav icon" class="w-10">
        <h1 class="text-xl font-semibold text-gray-700">Photo Sphere Viewer API</h1>
      </div>
      <ul class="flex gap-4">
        <li>
          <RouterLink to="/" class="text-xl font-bold text-black">Home</RouterLink>
        </li>
        <li>
          <RouterLink to="/photos" class="text-xl font-bold text-black">Fotos</RouterLink>
        </li>
        <li>
          <RouterLink to="/about" class="text-xl font-bold text-black">About</RouterLink>
        </li>
      </ul>
    </div>
  </nav>
  <RouterView />
</template>