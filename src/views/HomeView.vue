<script setup>
import { onMounted, ref, reactive, watch } from 'vue';
import { RouterLink, RouterView } from 'vue-router'
import { Viewer } from '@photo-sphere-viewer/core';
import { MarkersPlugin } from '@photo-sphere-viewer/markers-plugin';
import Image from '@/assets/image.jpg';
import PaperClip from '@/assets/svgs/PaperClip.vue'
import axios from 'axios';
import InputText from 'primevue/inputtext';
import Textarea from 'primevue/textarea';
import Button from 'primevue/button';
import InputSwitch from 'primevue/inputswitch';
import Pin1 from '@/assets/svgs/Pin1.vue'
import Pin2 from '@/assets/svgs/Pin2.vue'
import Pin3 from '@/assets/svgs/Pin3.vue'
import SvgPin1 from '@/assets/svgs/pin1.svg'
import SvgPin2 from '@/assets/svgs/pin2.svg'
import SvgPin3 from '@/assets/svgs/pin3.svg'

const viewerRef = ref(null);
const fileName = ref(null);
const isImageSelected = ref(false);
const viewer = ref(null);
const photoId = ref(null);
const activeTab = ref('panorama');
const markers = ref([]);
const addingMarker = ref(false);
const selectedMarkerCode = ref(null);
const selectedMarkerId = ref(null);
const editingMarker = ref(false);
const markerType = ref(null)
const markerPin = ref(SvgPin1)

const uploadImageToServer = async (formData) => {
  try {
    const response = await axios.post('http://127.0.0.1:8000/api/upload-image', formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    });
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
  icon_path: '',
  fisheye: false,
  mousewheel: true,
  mousewheelCtrlKey: false,
  mousemove: true,
  touchmoveTwoFingers: false,
  moveInertia: true,
  navbarOptions: {
    zoom: true,
    move: true,
    caption: true,
    download: true,
    fullscreen: true,
  },
});

const initializeViewer = () => {
  viewer.value = new Viewer({
    // const viewer = new Viewer({
    container: viewerRef.value,
    panorama: `http://127.0.0.1:8000/api/storage/${fileName.value}`,
    caption: photoData.caption,
    description: photoData.description,
    fisheye: photoData.fisheye,
    mousewheel: photoData.mousewheel,
    mousewheelCtrlKey: photoData.mousewheelCtrlKey,
    mousemove: photoData.mousemove,
    touchmoveTwoFingers: photoData.touchmoveTwoFingers,
    moveInertia: photoData.moveInertia,
    navbar: [
      photoData.navbarOptions.zoom ? 'zoom' : null,
      photoData.navbarOptions.move ? 'move' : null,
      photoData.navbarOptions.caption ? 'caption' : null,
      photoData.navbarOptions.download ? 'download' : null,
      photoData.navbarOptions.fullscreen ? 'fullscreen' : null,
    ].filter(Boolean),

    plugins: [
      [MarkersPlugin],
    ],
  });

  const markersPlugin = viewer.value.getPlugin(MarkersPlugin);

  viewer.value.addEventListener('click', ({ data }) => {
    if (editingMarker && selectedMarkerCode.value) {
      console.log('my data', data)
      editableMarkerData.code = selectedMarkerCode.value;
      editableMarkerData.yaw = data.yaw;
      editableMarkerData.pitch = data.pitch;
      // editableMarkerData.icon_path = data.icon_path;

      const newMarkerCode = '#' + Math.random();

      console.log('newMarkerCode', newMarkerCode)
      if (selectedMarkerCode.value != null) {
        markersPlugin.removeMarker(selectedMarkerCode.value)
        selectedMarkerCode.value = null
      }

      selectedMarkerCode.value = newMarkerCode
      editableMarkerData.code = selectedMarkerCode.value;

      console.log(markerPin)
      console.log('icon_path', editableMarkerData.icon_path)
      markersPlugin.addMarker({
        id: newMarkerCode,
        position: { yaw: data.yaw, pitch: data.pitch },
        image: editableMarkerData.icon_path,
        size: { width: 32, height: 32 },
        anchor: 'bottom center',
        tooltip: editableMarkerData.tooltip,
        content: '<div>' + editableMarkerData.content + '</div>',
        data: {
          generated: true,
        },
      });

      console.log('call updateMarkerData', editableMarkerData)
      updateMarkerData(editableMarkerData);

    } else {
      if (!data.rightclick && markerData.tooltip != '' && markerData.content != '') {

        const markerCode = '#' + Math.random();

        markersPlugin.addMarker({
          id: markerCode,
          // id: '#' + Math.random(),
          position: { yaw: data.yaw, pitch: data.pitch },
          // position: { yaw: 0, pitch: 0 },
          image: markerPin.value,
          size: { width: 32, height: 32 },
          anchor: 'bottom center',
          tooltip: markerData.tooltip,
          content: markerData.content,
          data: {
            generated: true,
          },
        });

        sendMarkerData(data, markerCode);
        // markerData.id 
        markerData.tooltip = '';
        markerData.content = '';

      }
    }
  });

};

watch(() => [photoData.caption, photoData.description, photoData.fisheye, photoData.mousewheel, photoData.mousewheelCtrlKey, photoData.mousemove, photoData.touchmoveTwoFingers, photoData.moveInertia, photoData.navbarOptions.zoom, photoData.navbarOptions.move, photoData.navbarOptions.caption, photoData.navbarOptions.download, photoData.navbarOptions.fullscreen, photoData.yaw, photoData.pitch, photoData.code], (newValues) => {
  if (viewer.value) {
    viewer.value.setOptions({
      caption: newValues[0],
      description: newValues[1],
      fisheye: newValues[2],
      mousewheel: newValues[3],
      mousewheelCtrlKey: newValues[4],
      mousemove: newValues[5],
      touchmoveTwoFingers: newValues[6],
      moveInertia: newValues[7],
      navbar: [
        newValues[8] ? 'zoom' : null,
        newValues[9] ? 'move' : null,
        newValues[10] ? 'caption' : null,
        newValues[11] ? 'download' : null,
        newValues[12] ? 'fullscreen' : null,
      ].filter(Boolean),

    });
    console.log('new value yaw', newValues[13])
    console.log('new value pitch', newValues[14])
    console.log('new value code', newValues[15])
  }
});

const sendMarkerData = async (data, code) => {
  data.photo_id = markerData.photo_id
  data.code = code
  data.tooltip = markerData.tooltip
  data.content = markerData.content
  data.icon_path = markerPin.value
  selectedMarkerCode.value = code
  try {
    const response = await axios.post(`http://127.0.0.1:8000/api/add-marker/${data.photo_id}`, data);

    markers.value.push(response.data)
    addingMarker.value = false
    activeTab.value = 'markers'
    selectedMarkerCode.value = null
  } catch (error) {
    console.error('Erro ao enviar dados', error);
  }
};

const updatePhotoInfo = async (photoData) => {
  try {
    const response = await axios.post(`http://127.0.0.1:8000/api/update-photo-info/${photoId.value}`, photoData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    });
  } catch (error) {
    console.error('Erro ao enviar a imagem:', error);
  }
};

const editableMarkerData = reactive({
  code: '',
  tooltip: '',
  content: '',
  yaw: '',
  pitch: '',
});

const selectMarkerForEditing = (markerId, markerCode, markerIconPath) => {
  editingMarker.value = true;
  const marker = markers.value.find(m => m.code === markerCode);
  console.log('marker', marker)
  if (marker) {
    selectedMarkerId.value = markerId;
    selectedMarkerCode.value = markerCode;
    // editableMarkerData.code = marker.code;
    editableMarkerData.icon_path = marker.icon_path;
    editableMarkerData.tooltip = marker.tooltip;
    editableMarkerData.content = marker.content;
    editableMarkerData.yaw = marker.yaw;
    editableMarkerData.pitch = marker.pitch;
  }
};

const back = () => {
  editingMarker.value = false
  selectedMarkerCode.value = null
}

const updateMarkerData = async (editableMarkerData) => {
  try {
    console.log('selectedMarkerCode.value', selectedMarkerCode.value)
    const response = await axios.post(`http://127.0.0.1:8000/api/update-marker-data/${selectedMarkerId.value}`, editableMarkerData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    });
    photoData.icon_path = editableMarkerData.icon_path;
    photoData.code = editableMarkerData.code;
    photoData.yaw = editableMarkerData.yaw;
    photoData.pitch = editableMarkerData.pitch;
    console.log('updateMarkerData:', response)

    const updatedMarker = response.data;

    const markerIndex = markers.value.findIndex(m => m.id === selectedMarkerId.value);
    if (markerIndex !== -1) {
      markers.value[markerIndex].icon_path = updatedMarker.icon_path;
      markers.value[markerIndex].code = updatedMarker.code;
      markers.value[markerIndex].yaw = updatedMarker.yaw;
      markers.value[markerIndex].pitch = updatedMarker.pitch;
    }

    console.log('markers', markers.value)

  } catch (error) {
    console.error('Erro ao enviar a imagem:', error);
  }
};

</script>

<template>
  <div class="mx-2 md:mx-4 lg:mx-6 xl:mx-8 2xl:mx-10 xl:h-[80vh]">
    <!-- <div class="bg-black text-white flex flex-wrap p-2 gap-4">
      <div class="border p-4">
        addingMarker: {{ addingMarker }}
      </div>
      <div class="border p-4">
        selectedMarkerCode: {{ selectedMarkerCode }}
      </div>
      <div class="border p-4">
        editingMarker: {{ editingMarker }}
      </div>
    </div> -->
    <div class="flex flex-col gap-4 2xl:grid grid-cols-3 mt-12 xl:h-full">

      <div class="w-full border border-gray-50 shadow-xl h-full relative"
        v-if="addingMarker === false && editingMarker === false">
        <div class="grid grid-cols-4 bg-blue-500 shadow-lg hover:cursor-pointer">
          <div @click="activeTab = 'panorama'"
            class="p-4 w-full text-center text-gray-200 text-xs lg:text-sm font-semibold flex items-center justify-center"
            :class="{ 'bg-white bg-opacity-15 border-b-2 border-white text-white': activeTab === 'panorama' }">PANORAMA
          </div>
          <div @click="activeTab = 'standardOptions'"
            class="p-4 w-full text-center text-gray-200 text-xs lg:text-sm font-semibold flex items-center justify-center"
            :class="{ 'bg-white bg-opacity-15 border-b-2 border-white text-white': activeTab === 'standardOptions' }">
            STANDARD
            OPTIONS</div>
          <div @click="activeTab = 'navbarConfig'"
            class="p-4 w-full text-center text-gray-200 text-xs lg:text-smext-sm font-semibold flex items-center justify-center"
            :class="{ 'bg-white bg-opacity-15 border-b-2 border-white text-white': activeTab === 'navbarConfig' }">NAVBAR
            CONFIG
          </div>
          <div @click="activeTab = 'markers'"
            class="p-4 w-full text-center text-gray-200 text-xs lg:text-sm font-semibold flex items-center justify-center"
            :class="{ 'bg-white bg-opacity-15 border-b-2 border-white text-white': activeTab === 'markers' }">MARKERS
          </div>
        </div>
        <div v-if="activeTab === 'panorama'" class="flex flex-col gap-8 p-4 min-h-[40rem]">
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

        </div>
        <div v-if="activeTab === 'standardOptions'" class="min-h-[40rem]">
          <div class="grid grid-cols-2 gap-4 pt-8 px-4">
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Fisheye</label>
              <InputSwitch v-model="photoData.fisheye" />
            </div>
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Mousewheel</label>
              <InputSwitch v-model="photoData.mousewheel" />
            </div>
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Hold Ctrl to zoom</label>
              <InputSwitch v-model="photoData.mousewheelCtrlKey" />
            </div>
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Mouse move</label>
              <InputSwitch v-model="photoData.mousemove" />
            </div>
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Two fingers move</label>
              <InputSwitch v-model="photoData.touchmoveTwoFingers" />
            </div>
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Move inertia</label>
              <InputSwitch v-model="photoData.moveInertia" />
            </div>
          </div>
        </div>
        <div v-if="activeTab === 'navbarConfig'" class="min-h-[40rem]">
          <div class="grid grid-cols-2 gap-4 pt-8 px-4">
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Zoom</label>
              <InputSwitch v-model="photoData.navbarOptions.zoom" />
            </div>
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Move</label>
              <InputSwitch v-model="photoData.navbarOptions.move" />
            </div>
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Caption</label>
              <InputSwitch v-model="photoData.navbarOptions.caption" />
            </div>
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Download</label>
              <InputSwitch v-model="photoData.navbarOptions.download" />
            </div>
            <div class="flex justify-start items-center gap-2">
              <label class="font-semibold">Fullscreen</label>
              <InputSwitch v-model="photoData.navbarOptions.fullscreen" />
            </div>
          </div>
        </div>
        <div v-if="activeTab === 'markers'" class="flex flex-col gap-4 p-4 pt-8 min-h-[40rem] overflow-y-auto">

          <div>
            <h1>markers</h1>
            <div class="flex flex-col gap-2">
              <div v-for="marker in markers" :key="marker.id" class="bg-gray-100 p-2">
                <h1>{{ marker.id }}</h1>
                <!-- <h1>{{ marker.code }}</h1>
                <h1>{{ marker.yaw }}</h1>
                <h1>{{ marker.pitch }}</h1> -->
                <h1>{{ marker.icon_path }}</h1>
                <Button @click="selectMarkerForEditing(marker.id, marker.code, marker.icon_path)" label="Editar" />
              </div>
            </div>
          </div>

        </div>

        <div class="flex justify-between items-center w-full border-t absolute bottom-0">
          <Button label="RESETAR TUDO" class="bg-red-500 border-none w-fit m-4" />
          <Button label="ADICIONAR MARCADORES" class="w-fit m-4" @click="addingMarker = true" />
        </div>
      </div>

      <div class="w-full border border-gray-50 shadow-xl h-fit" v-if="addingMarker === true">
        <div class="flex flex-col gap-4 p-4 pt-8" v-if="markerType === null">

          <Button label="Imagem" class="w-full p-4" @click="markerType = 'image'" />
          <Button label="Texto" class="w-full p-4" @click="markerType = 'text'" />

        </div>
        <div v-if="markerType === 'image'">
          <div class="flex flex-col gap-4 p-4">

            <div class="flex flex-wrap gap-2">
              <div class="border p-2 hover:cursor-pointer" :class="{ 'border-2 border-blue-500': markerPin === SvgPin1 }"
                @click="markerPin = SvgPin1">
                <Pin1 :class="'w-12 h-12'" />
              </div>
              <div class="border p-2 hover:cursor-pointer" :class="{ 'border-2 border-blue-500': markerPin === SvgPin2 }"
                @click="markerPin = SvgPin2">
                <Pin2 :class="'w-12 h-12'" />
              </div>
              <div class="border p-2 hover:cursor-pointer" :class="{ 'border-2 border-blue-500': markerPin === SvgPin3 }"
                @click="markerPin = SvgPin3">
                <Pin3 :class="'w-12 h-12'" />
              </div>
            </div>
            <span class="p-float-label mt-2">
              <InputText id="marker_title" class="w-full" v-model="markerData.tooltip"
                @blur="updatePhotoInfo(photoData)" />
              <label for="marker_title">Título do Marcador</label>
            </span>

            <span class="p-float-label mt-2">
              <Textarea v-model="markerData.content" class="w-full" rows="8" @blur="updatePhotoInfo(photoData)" />
              <label>Descrição</label>
            </span>
          </div>
        </div>
        <div v-if="markerType === 'text'">
          <span class="p-float-label">
            <InputText id="marker_title" class="w-full" v-model="markerData.tooltip" @blur="updatePhotoInfo(photoData)" />
            <label for="marker_title">Título do Marcador</label>
          </span>

          <span class="p-float-label">
            <Textarea v-model="markerData.content" class="w-full" rows="8" @blur="updatePhotoInfo(photoData)" />
            <label>Descrição</label>
          </span>
        </div>
        <div>
          <Button label="Voltar" class="w-full p-4" @click="markerType = null" v-if="markerType != null" />
          <Button label="Voltar" class="w-full p-4" @click="addingMarker = false" v-else />
        </div>
      </div>

      <div class="w-full border border-gray-50 shadow-xl h-fit" v-if="editingMarker">
        <div class="flex flex-col gap-8 p-4 pt-8">
          <h3>Edit Marker</h3>
          <h2>Code: {{ editableMarkerData.code }}</h2>
          <span class="p-float-label">
            <InputText id="tooltip" class="w-full" v-model="editableMarkerData.tooltip"
              @blur="updatePhotoInfo(photoData)" />
            <label for="tooltip">Tooltip</label>
          </span>
          <span class="p-float-label">
            <Textarea v-model="editableMarkerData.content" class="w-full" rows="8" @blur="updatePhotoInfo(photoData)" />
            <label>Content</label>
          </span>
          <span class="p-float-label">
            <InputText id="yaw" class="w-full" v-model="editableMarkerData.yaw" />
            <label for="yaw">Yaw</label>
          </span>
          <span class="p-float-label">
            <InputText id="pitch" class="w-full" v-model="editableMarkerData.pitch" />
            <label for="pitch">Pitch</label>
          </span>
          <Button @click="back()" label="Voltar" />
        </div>
      </div>


      <div ref="viewerRef" class="w-full h-96 xl:h-full 2xl:col-span-2">
        <div class="w-full h-96 xl:h-full 2xl:col-span-2 flex items-center justify-center bg-gray-50 shadow-xl"
          v-if="!isImageSelected">
          <h1 class="text-3xl font-semibold text-gray-700 text-center">Please select a panorama file</h1>
        </div>
      </div>

    </div>

  </div>
</template>