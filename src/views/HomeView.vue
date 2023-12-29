<script setup>
import { onMounted, ref, reactive, watch } from 'vue';
import { RouterLink, RouterView } from 'vue-router'
import { Viewer } from '@photo-sphere-viewer/core';
import { MarkersPlugin } from '@photo-sphere-viewer/markers-plugin';
import Image from '@/assets/image.jpg';
import SvgMarker from '@/assets/svgs/marker.svg'
import PaperClip from '@/assets/svgs/PaperClip.vue'
import axios from 'axios';
import InputText from 'primevue/inputtext';
import Textarea from 'primevue/textarea';
import Button from 'primevue/button';

const viewerRef = ref(null);
const fileName = ref(null);
const isImageSelected = ref(false);
const viewer = ref(null);
const photoId = ref(null);
const activeTab = ref('panorama');

const uploadImageToServer = async (formData) => {
  try {
    const response = await axios.post('http://127.0.0.1:8000/api/upload-image', formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    });
    // console.log(response.data.url)
    // console.log(response.data.id)
    if (response.data && response.data.url && response.data.id) {
      fileName.value = response.data.url;
      markerData.photo_id = response.data.id;
      photoId.value = response.data.id
      isImageSelected.value = true;
      initializeViewer();
    }

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
  photo_id: null,
  tooltip: '',
  content: '',
  yaw: '',
  pitch: '',
});

const photoData = reactive({
  caption: '',
  description: '',
});

const initializeViewer = () => {
  viewer.value = new Viewer({
    // const viewer = new Viewer({
    container: viewerRef.value,
    panorama: `http://127.0.0.1:8000/api/storage/${fileName.value}`,
    caption: photoData.caption,
    description: photoData.description,

    plugins: [
      [MarkersPlugin],
    ],
  });

  console.log('viewer', viewer)

  const markersPlugin = viewer.value.getPlugin(MarkersPlugin);

  viewer.value.addEventListener('click', ({ data }) => {
    console.log('data:', data)
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

watch(() => [photoData.caption, photoData.description], (newValues) => {
  if (viewer.value) {
    viewer.value.setOptions({
      caption: newValues[0],
      description: newValues[1],
    });
  }
});

const sendMarkerData = async (data) => {
  console.log('dataStore', data)
  data.photo_id = markerData.photo_id
  data.tooltip = markerData.tooltip
  data.content = markerData.content
  try {
    await axios.post(`http://127.0.0.1:8000/api/add-marker/${data.photo_id}`, data);
    console.log('Dados enviados com sucesso');
    console.log('data:', data)
  } catch (error) {
    console.error('Erro ao enviar dados', error);
    console.log('data:', data)
  }
};

const updatePhotoInfo = async (photoData) => {
  try {
    const response = await axios.post(`http://127.0.0.1:8000/api/update-photo-info/${photoId.value}`, photoData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    });
    console.log('teste:', response)
  } catch (error) {
    console.error('Erro ao enviar a imagem:', error);
  }
};
</script>

<template>
  <div class="mx-2 md:mx-4 lg:mx-6 xl:mx-8 2xl:mx-10">
    <div class="flex flex-col gap-4 2xl:grid grid-cols-3 mt-12">

      <div class="w-full border border-gray-50 shadow-xl h-fit">
        <div class="flex justify-between bg-blue-500 shadow-lg hover:cursor-pointer">
          <div @click="activeTab = 'panorama'" class="p-4 w-full text-center text-gray-200 text-sm font-semibold"
            :class="{ 'bg-white bg-opacity-15 border-b-2 border-white text-white': activeTab === 'panorama' }">PANORAMA
          </div>
          <div @click="activeTab = 'standardOptions'" class="p-4 w-full text-center text-gray-200 text-sm font-semibold"
            :class="{ 'bg-white bg-opacity-15 border-b-2 border-white text-white': activeTab === 'standardOptions' }">
            STANDARD
            OPTIONS</div>
          <div @click="activeTab = 'navbarConfig'" class="p-4 w-full text-center text-gray-200 text-sm font-semibold"
            :class="{ 'bg-white bg-opacity-15 border-b-2 border-white text-white': activeTab === 'navbarConfig' }">NAVBAR
            CONFIG
          </div>
        </div>
        <div v-if="activeTab === 'panorama'" class="flex flex-col gap-8 p-4">
          <label for="dropzone-photo" class="hover:cursor-pointer">
            <div class="border shadow-md p-4 flex gap-4">
              <PaperClip /> <span>Panorama image</span>
            </div>
            <input id="dropzone-photo" type="file" @change="addImage" accept=".png, .jpeg, .jpg" class="hidden" />
          </label>
          <span class="p-float-label">
            <InputText id="caption" class="w-full" v-model="photoData.caption" @blur="updatePhotoInfo(photoData)" />
            <label for="caption">Caption</label>
          </span>
          <span class="p-float-label">
            <Textarea v-model="photoData.description" class="w-full" rows="8" @blur="updatePhotoInfo(photoData)" />
            <label>Description</label>
          </span>

          <div class="py-8">
            <input type="text" v-model="markerData.tooltip" placeholder="Título do Marcador" />
            <textarea v-model="markerData.content" placeholder="Descrição"></textarea>
          </div>
        </div>
        <div v-if="activeTab === 'standardOptions'">
          
        </div>
        <div v-if="activeTab === 'navbarConfig'">
         
        </div>

      </div>

      <div ref="viewerRef" class="w-full h-[40rem] 2xl:col-span-2">
        <div class="w-full h-[40rem] 2xl:col-span-2 flex items-center justify-center bg-gray-50 shadow-xl"
          v-if="!isImageSelected">
          <h1 class="text-3xl font-semibold text-gray-700">Please select a panorama file</h1>
        </div>
      </div>

    </div>

</div></template>