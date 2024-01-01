<script setup>
import { onMounted, ref, reactive, watch } from 'vue';
import { RouterLink, RouterView } from 'vue-router'
import { Viewer } from '@photo-sphere-viewer/core';
import { MarkersPlugin } from '@photo-sphere-viewer/markers-plugin';
import axios from 'axios';

const viewerRef = ref(null);
const imageUrl = ref(null);
const fileName = ref(null);
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
        tooltip: marker.tooltip,
        content: marker.content,
        position: { yaw: marker.yaw, pitch: marker.pitch },
    }));
};

const initViewer = (photo) => {
    new Viewer({
        container: document.getElementById(`viewer-${photo.id}`),
        panorama: `http://127.0.0.1:8000/api/storage/${photo.photo_path}`,
        caption: photo.caption,
        description: photo.description,
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
    console.log(photos.value)
});

const uploadImageToServer = async (formData) => {
    try {
        const response = await axios.post('http://127.0.0.1:8000/api/upload-image', formData, {
            headers: {
                'Content-Type': 'multipart/form-data',
            },
        });
        console.log(response.data.url)
        if (response.data && response.data.url) {
            fileName.value = response.data.url;
            console.log('file name = ', fileName.value)
        }

        initializeViewer(imageUrl.value);
    } catch (error) {
        console.error('Erro ao enviar a imagem:', error);
    }
};

const addImage = (event) => {
    const file = event.target.files[0];
    if (file) {
        const formData = new FormData();
        formData.append('photo', file);
        uploadImageToServer(formData);
    }
};

const markerData = reactive({
    tooltip: '',
    content: '',
    yaw: '',
    pitch: '',
});

const initializeViewer = (url) => {
    const viewer = new Viewer({
        container: viewerRef.value,
        panorama: `http://127.0.0.1:8000/api/storage/${fileName.value}`,
        plugins: [
            [MarkersPlugin, {
                markers: [
                    {
                        id: 1,
                        position: { yaw: '0deg', pitch: '0deg' },
                        circle: 10,
                        anchor: 'bottom center',
                        tooltip: 'Circle',
                    },
                ],
            }],
        ],
    });
    const markersPlugin = viewer.getPlugin(MarkersPlugin);

    viewer.addEventListener('click', ({ data }) => {
        // console.log(data)
        if (!data.rightclick && markerData.tooltip != '' && markerData.content != '') {
            markersPlugin.addMarker({
                id: '#' + Math.random(),
                position: { yaw: data.yaw, pitch: data.pitch },
                image: SvgMarker,
                size: { width: 32, height: 32 },
                anchor: 'bottom center',
                tooltip: markerData.tooltip,
                content: '<div>' + markerData.content + '</div>',
                data: {
                    generated: true,
                },
            });

            sendMarkerData(data);

            markerData.tooltip = '';
            markerData.content = '';

        }
    });
};

const sendMarkerData = async (data) => {
    data.id = markerData.id
    data.tooltip = markerData.tooltip
    data.content = markerData.content
    try {
        await axios.post('http://127.0.0.1:8000/api/add-marker', data);
        console.log('Dados enviados com sucesso');
        console.log('data:', data)
    } catch (error) {
        console.error('Erro ao enviar dados', error);
        console.log('data:', data)
    }
};
</script>

<template>
    <div class="container mx-auto mt-4">
        <div class="grid grid-cols-2 gap-4 my-8">
            <div v-for="photo in photos" :key="photo.id" :id="`viewer-${photo.id}`" class="w-full h-[30rem]"></div>
        </div>
    </div>

    <RouterView />
</template>