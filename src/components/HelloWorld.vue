<template>
  <div class="container">
    <h1>Álbum de Videos de YouTube</h1>
  
    <div v-if="videoList.length === 0" class="video-list">
      <p>No hay videos guardados.</p>
    </div>
  
    <div v-else class="video-list">
      <div v-for="video in videoList" :key="video.id" class="card card-small">
        <img :src="video.thumbnail" :alt="video.title" class="card-img-top" @click="abrirModal(video)" />
        <div class="card-body">
          <!-- <h3 class="card-title">{{ video.title }}</h3> -->
          <!-- <p class="card-text">{{ video.description }}</p> -->
          <button @click="eliminarVideo(video.id)" class="btn btn-danger">Eliminar</button>
        </div>
      </div>
    </div>
    
    <div class="modal" v-if="modalVideo">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">{{ modalVideo.title }}</h5>
            <button type="button" class="btn-close" @click="cerrarModal"></button>
          </div>
          <div class="modal-body">
            <div class="video-container">
              <iframe :src="getVideoUrl(modalVideo.id)" frameborder="0" allowfullscreen></iframe>
            </div>
            <div class="video-description">
              <p v-html="modalVideo.description"></p>
            </div>
          </div>
        </div>
      </div>
    </div>
  
    <h2>Agregar nuevo video</h2>
  
    <input type="text" v-model="videoUrl" placeholder="Pega el enlace del video de YouTube" class="form-control" />
    <div class="mb-3">
      <button @click="agregarVideo" class="btn btn-primary">Agregar</button>
    </div>
    
  </div>
</template>

<script setup>
import Swal from 'sweetalert2';
import { ref, onMounted } from 'vue';
import axios from 'axios';

const videoList = ref([]);
const videoUrl = ref('');
const modalVideo = ref(null);




async function cargarVideos() {
  const response = await axios.get('http://localhost:3000/videos');
  videoList.value = response.data;
}
function validarCampos() {
  if (!videoUrl.value ) {
    Swal.fire({
      title: 'Campos vacíos',
      text: 'Por favor, completa todos los campos.',
      icon: 'error'
    });
    return false;
  }
  return true;
}




async function agregarVideo() {
  if (validarCampos()) {
    const videoId = extraerIdVideo(videoUrl.value);
    const videoInfo = await obtenerInfoVideo(videoId);
    console.log(videoInfo);
  
    const response = await axios.post('http://localhost:3000/videos', videoInfo);
    /* const response = await axios.post('http://localhost:3000/videos',  { 
      id: "Si3gjswSPmQ", 
      title: "Inevitable", 
      thumbnail: "https://i.ytimg.com/vi/Si3gjswSPmQ/default.jpg", 
      description: "x" }, {
      headers: {'Content-Type': 'application/json'}
    }); */
    videoList.value.push(response.data);

    

    Swal.fire({
      title: 'Video agregado',
      text: 'El video se ha añadido correctamente.',
      icon: 'success',
      timer: 2000, // Duración de la alerta en milisegundos (opcional)
      showConfirmButton: false // Oculta el botón de confirmación (opcional)
    });

        // Restablecer los valores de los campos del formulario
     
  }
}

/* async function eliminarVideo(videoId) {
  const confirmacion = window.confirm('¿Estás seguro de que deseas eliminar este video?');
  if (confirmacion) {
  
  }
 
} */

async function eliminarVideo(videoId) {
  const confirmacion = await Swal.fire({
    title: '¿Estás seguro?',
    text: 'Esta acción no se puede deshacer',
    icon: 'warning',
    showCancelButton: true,
    confirmButtonText: 'Eliminar',
    cancelButtonText: 'Cancelar',
    reverseButtons: true
  });

  if (confirmacion.isConfirmed) {
    await axios.delete(`http://localhost:3000/videos/${videoId}`);
    videoList.value = videoList.value.filter(video => video.id !== videoId);
  }
}

function extraerIdVideo(url) {
  const match = url.match(/^[^v]+v=(.{11}).*/);
  return (match && match[1]) || '';
}

async function obtenerInfoVideo(videoId) {
  const response = await axios.get(`https://www.googleapis.com/youtube/v3/videos?id=${videoId}&key=AIzaSyChvIDZ5TwfcKSnpfSiWS1y1ec7b36yigc&part=snippet`);
  const video = response.data.items[0];
 
  return {
    id: video.id,
    title: video.snippet.title,
    thumbnail: video.snippet.thumbnails.default.url,
    description: video.snippet.description
  };
}

function abrirModal(video) {
  modalVideo.value = video;
}

function cerrarModal() {
  modalVideo.value = null;
}
function getVideoUrl(videoId) {
  return `https://www.youtube.com/embed/${videoId}`;
}
onMounted(cargarVideos);
</script>

<style scoped>

.video-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;
}

/* Estilos para el modal */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 999;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-dialog {
  max-width: 800px;
  width: 90%;
}

.modal-content {
  position: relative;
  display: flex;
  flex-direction: column;
  padding: 1rem;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.modal-title {
  margin: 0;
}

.btn-close {
  padding: 0.5rem;
}

.modal-body {
  flex-grow: 1;
  overflow-y: auto;
}

.video-container {
  position: relative;
  padding-bottom: 56.25%;
  height: 0;
  overflow: hidden;
}

.video-container iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: 0;
}

.description {
  margin-top: 1rem;
  overflow-y: auto;
  max-height: 200px;
}

.card-small {
  width: 150px;
  height: auto;
  margin-right: 10px;
  margin-bottom: 10px;
}

.card-img-top {
  width: 100%;
  height: auto;
}
.card-text {
  margin-top: 1rem;
  max-height: 100px;
  overflow-y: auto;
}


.video-description {
  max-height: 150px; /* Ajusta la altura máxima según tus necesidades */
  overflow-y: auto; /* Habilita el desplazamiento vertical */
}


.mb-3 {
  margin-top: 22px;
}

</style>