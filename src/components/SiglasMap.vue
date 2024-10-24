<script setup>
import * as L from 'leaflet';
import { onMounted } from 'vue';
import { MdScienceRound, BiCodeSlash} from "oh-vue-icons/icons";

function generateSvgIcon(component, color = 'black') {
  const svgString = `
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="${color}" width="32" height="32">
      ${component.raw}
    </svg>
  `;
  return `data:image/svg+xml;base64,${btoa(svgString)}`;
}

// Función para obtener un ícono personalizado por sector
function getCustomIcon(person) {
  let iconUrl = person.firstName.charAt(0).toUpperCase() + person.lastName.charAt(0).toUpperCase();
  console.log(iconUrl)

  return L.icon({
    iconUrl: iconUrl,
    iconSize: [32, 32],  // Tamaño del ícono
    iconAnchor: [16, 32], // Punto de anclaje del ícono
    popupAnchor: [0, -32], // Punto desde donde se abre el popup
  });
}

const people = [
  { firstName: "Mia", lastName: "Doe", location: [51.505, -0.09], sector: "Science" },
  { firstName: "Jane", lastName: "Smith", location: [51.51, -0.1], sector: "Mathematics" },
  { firstName: "Martha", lastName: "Beam", location: [51.52, -0.08], sector: "Engineering" },
  { firstName: "John", lastName: "Doe", location: [51.53, -0.07], sector: "Arts" },
  { firstName: "Tom", lastName: "Brown", location: [51.54, -0.06], sector: "Mathematics" }
];

onMounted(() => {
  const map = L.map('map').setView([51.505, -0.09], 13);

  L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
  }).addTo(map);

  people.forEach(person => {
    // Obtener el ícono personalizado basado en el sector
    const customIcon = getCustomIcon(person);

    // Crear un marcador para cada persona con el ícono personalizado
    const marker = L.marker(person.location, { icon: customIcon }).addTo(map);

    // Crear un popup con información sobre la persona
    const popupContent = `
      <strong>${person.firstName} ${person.lastName}</strong><br>
      Sector: ${person.sector}
    `;

    marker.bindPopup(popupContent);
  });
}); 

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
