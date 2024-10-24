<script setup>
import * as L from 'leaflet';
import { onMounted } from 'vue';
import { MdScienceRound } from "oh-vue-icons/icons";

function getCustomMarker(sectors) {
  // Obtener las letras de cada sector
  const letters = sectors.map(sector => {
    switch (sector) {
      case 'Science':
        return 'S'; // Ciencia
      case 'Technology':
        return 'T'; // Tecnología
      case 'Engineering':
        return 'E'; // Ingeniería
      case 'Arts':
        return 'A'; // Artes
      case 'Mathematics':
        return 'M'; // Matemáticas
      default:
        return '?'; // Default si no hay sector
    }
  }).join(' '); // Unir las letras con un espacio

  const markerDiv = L.divIcon({
    className: 'custom-marker', // Clase CSS para el estilo del marcador
    html: `<div>${letters}</div>`,
    iconSize: [32, 32],  // Tamaño del ícono
    iconAnchor: [16, 32], // Punto de anclaje del ícono
    popupAnchor: [0, -32], // Punto desde donde se abre el popup
  });

  return markerDiv;
}

const people = [
  { firstName: "Mia", lastName: "Doe", location: [51.505, -0.09], sectors: ["Science", "Technology"] },
  { firstName: "Jane", lastName: "Smith", location: [51.51, -0.1], sectors: ["Mathematics"] },
  { firstName: "Martha", lastName: "Beam", location: [51.52, -0.08], sectors: ["Engineering", "Arts", "Science"] },
  { firstName: "John", lastName: "Doe", location: [51.53, -0.07], sectors: ["Arts", "Mathematics"] },
  { firstName: "Tom", lastName: "Brown", location: [51.54, -0.06], sectors: ["Mathematics"] }
];

onMounted(() => {
  const map = L.map('map').setView([51.505, -0.09], 13);

  L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
  }).addTo(map);

  people.forEach(person => {
    const customMarker = getCustomMarker(person.sectors);

    const marker = L.marker(person.location, { icon: customMarker }).addTo(map);

    const popupContent = `
      <strong>${person.firstName} ${person.lastName}</strong><br>
      Sector: ${person.sectors.join(', ')}
    `;

    marker.bindPopup(popupContent);
  });
})

</script>

<template>
   <div id="map"></div>
</template>

<style >

#map { 
  height: 500px; /* Aumentar altura para una mejor visibilidad */
  width: 100%;   
  min-width: 600px;
  border: 2px solid black;
}

.custom-marker div {
  background-color: rgba(255, 255, 255, 0.8); /* Fondo blanco semitransparente */
  color: black; /* Color del texto */
  text-align: center; /* Centrar el texto */
  border-radius: 50%; /* Hacer el fondo circular */
  width: 32px; /* Ancho del marcador */
  height: 32px; /* Alto del marcador */
  display: flex; /* Usar flexbox para centrar */
  align-items: center; /* Centrar verticalmente */
  justify-content: center; /* Centrar horizontalmente */
  font-size: calc(16px - (3px * (var(--sectors-count) - 1))); /* Tamaño de la letra */
  font-weight: bold; /* Hacer la letra más gruesa */
  border: 2px solid #000; /* Bordes de los marcadores */
}
</style>
