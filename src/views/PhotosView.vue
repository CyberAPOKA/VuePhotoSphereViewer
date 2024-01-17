<script setup>
import { onMounted, ref, reactive, watch } from 'vue';
import { RouterLink, RouterView } from 'vue-router'
import { Viewer } from '@photo-sphere-viewer/core';
import { MarkersPlugin } from '@photo-sphere-viewer/markers-plugin';
import axios from 'axios';
import PencilSquare from '@/assets/svgs/PencilSquare.vue'
import SvgPin1 from '@/assets/svgs/pin1.svg'
import router from '@/router/index.js'

const svgIcon = ref(null);

const photos = ref([]);

const getPhotos = async () => {
    try {
        const response = await axios.get('http://127.0.0.1:8000/api/photos');
        photos.value = response.data;
    } catch (error) {
        console.error('Erro ao obter fotos', error);
    }
};

const convertMarkers = (markers) => {
    return markers.map(marker => ({
        // id: marker.id.toString(),
        id: marker.code,
        image: marker.icon_path,
        size: { width: 32, height: 32 },
        anchor: 'bottom center',
        html: marker.html,
        tooltip: marker.tooltip,
        content: marker.content,
        position: { yaw: marker.yaw, pitch: marker.pitch },
    }));
};

const initViewer = (photo) => {
    console.log('X:', photo.id)
    new Viewer({
        container: document.getElementById(`viewer-${photo.id}`),
        panorama: `http://127.0.0.1:8000/api/storage/${photo.photo_path}`,
        caption: photo.caption,
        description: photo.description,
        navbar: [
            'zoom',
            'move',
            'caption',
            {
                id: 'edit',
                title: 'Edit photo',
                content: svgIcon.value.innerHTML,
                onClick() {
                    // alert(photo.id)
                    router.push(`/photo/${photo.id}`)
                    // levar para a rota http://localhost:5173/photo/{photo.id}
                },
            },

            'fullscreen',
        ],
        plugins: [
            [MarkersPlugin, {
                markers: convertMarkers(photo.markers),
            }],
        ],
    });
};

onMounted(async () => {
    await getPhotos();
    photos.value.forEach(photo => initViewer(photo));
    // console.log(photos.value)
});
</script>

<template>
    <div class="container mx-auto mt-4">
        <div class="grid grid-cols-2 gap-4 my-8">
            <div v-for="photo in photos" :key="photo.id" :id="`viewer-${photo.id}`" class="w-full h-[30rem] relative">
                <h1 class="absolute top-0 right-0 z-10 text-xs bg-white bg-opacity-30 p-[0.1rem] rounded-bl-md text-black">
                    By Christian André Steffens</h1>
            </div>
        </div>
    </div>

    <span ref="svgIcon" style="display: none;">

        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
            <path
                d="M21.731 2.269a2.625 2.625 0 0 0-3.712 0l-1.157 1.157 3.712 3.712 1.157-1.157a2.625 2.625 0 0 0 0-3.712ZM19.513 8.199l-3.712-3.712-12.15 12.15a5.25 5.25 0 0 0-1.32 2.214l-.8 2.685a.75.75 0 0 0 .933.933l2.685-.8a5.25 5.25 0 0 0 2.214-1.32L19.513 8.2Z" />
        </svg>

    </span>

    <RouterView />
</template>