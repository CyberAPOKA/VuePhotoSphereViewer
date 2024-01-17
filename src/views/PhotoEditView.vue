<script setup>
import { onMounted, ref, reactive, watch } from 'vue';
import { RouterLink, RouterView } from 'vue-router'
import { Viewer } from '@photo-sphere-viewer/core';
import { MarkersPlugin } from '@photo-sphere-viewer/markers-plugin';
import axios from 'axios';
import router from '@/router/index.js'
import { useRoute } from 'vue-router';

const photo = ref();
const route = useRoute();
const photoId = route.params.id;

const fetchPhoto = async () => {
    try {
        const response = await axios.get(`http://127.0.0.1:8000/api/photo/${photoId}`);
        photo.value = response.data;
    } catch (error) {
        console.error('Erro ao obter foto', error);
    }
};

const initViewer = () => {
    if (photo.value) {
        new Viewer({
            container: document.getElementById(`viewer-${photo.value.id}`),
            panorama: `http://127.0.0.1:8000/api/storage/${photo.value.photo_path}`,
            caption: photo.value.caption,
            description: photo.value.description,
            // ... outros parâmetros e plugins
        });
    }
};

onMounted(async () => {
    await fetchPhoto();
    initViewer();
});
</script>
<template>
    <div class="container mx-auto mt-2" style="height: calc(100vh - 6rem);">
        <div v-if="photo" :id="`viewer-${photo.id}`" class="w-full h-full"></div>
    </div>
</template>