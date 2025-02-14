<template>
  <div style="background-color: white; position: relative">
    <div id="map"></div>
    <MiniMap
      :center="[28.50291, -15.88168]"
      :zoom="0"
      style="
        height: 250px;
        width: 250px;
        position: absolute;
        bottom: 0px;
        right: 0px;
      "
    />
    <ListComponent
      v-if="showList"
      :markers="listData"
      :visible="showList"
      @close="showList = false"
      style="position: absolute; top: 10px; right: 10px; z-index: 1000"
    />

    <Card 
      v-if="showCard"
      :person=target

      @close="showCard = false"
      style="position: absolute; top: 50px; right: 50px; z-index: 1000"
    />
  </div>
</template>

<script setup>
import MiniMap from "./MiniMap.vue";
import ListComponent from "./ListComponent.vue";
import Card from './Card.vue'
import * as L from "leaflet";
import "@maptiler/leaflet-maptilersdk";
import "leaflet.markercluster/dist/MarkerCluster.Default.css";
import "leaflet.markercluster";
import { onMounted, ref } from "vue";
import { CoLgtm, MdScienceRound, CoLocationPin } from "oh-vue-icons/icons";
const showList = ref(false);
const showCard = ref(false);
const clusterMarkers = ref([]);
const listData = ref([]);
const target = ref({})

const renderIcon = () => {
  // Generar el SVG del icono
  const iconSvg = generateSvgIcon( "purple"); // Cambia el color a violeta
  console.log(iconSvg);
  return L.divIcon({
    html: iconSvg, // Usar el SVG como HTML
    className: "custom-icon",
    iconSize: [32, 32], // Tamaño del ícono
    iconAnchor: [16, 32], // Punto de anclaje del ícono
    popupAnchor: [0, -32], // Punto desde donde se abre el popup
  });
};

function generateSvgIcon(color = "black") {
  return `
    <svg
      xmlns="http://www.w3.org/2000/svg"
      xmlns:xlink="http://www.w3.org/1999/xlink"
      version="1.1"
      width="32" height="32" viewBox="0 0 256 256"
      xml:space="preserve"
    >
      <defs></defs>
      <g
        style="fill: none; fill-rule: nonzero; opacity: 1;"
        transform="translate(1.4065934065934016 1.4065934065934016) scale(2.81 2.81)"
      >
        <path
          d="M 45 0 C 25.463 0 9.625 15.838 9.625 35.375 c 0 8.722 3.171 16.693 8.404 22.861 L 45 90 l 26.97 -31.765 c 5.233 -6.167 8.404 -14.139 8.404 -22.861 C 80.375 15.838 64.537 0 45 0 z M 45 48.705 c -8.035 0 -14.548 -6.513 -14.548 -14.548 c 0 -8.035 6.513 -14.548 14.548 -14.548 s 14.548 6.513 14.548 14.548 C 59.548 42.192 53.035 48.705 45 48.705 z"
          style="fill: ${color}; stroke: none;"
          transform="matrix(1 0 0 1 0 0)"
          stroke-linecap="round"
        />
      </g>
    </svg>
  `;
}


const people = [
  {
    firstName: "Mia",
    lastName: "Doe",
    location: [28.120058, -15.43],
    sectors: ["Science", "Technology"],
  },
  {
    firstName: "Jane",
    lastName: "Smith",
    location: [28.1232, -15.45],
    sectors: ["Mathematics"],
  },
  {
    firstName: "Martha",
    lastName: "Beam",
    location: [28.1249, -15.5],
    sectors: ["Engineering", "Arts", "Science"],
  },
  {
    firstName: "John",
    lastName: "Doe",
    location: [28.463, -16.27],
    sectors: ["Arts", "Mathematics"],
  },
  {
    firstName: "Tom",
    lastName: "Brown",
    location: [28.996, -13.592],
    sectors: ["Mathematics"],
  },
  { firstName: "Martha", lastName: "Brown", location: [40.66, 128.65], sectors: ["Mathematics"] }

];

onMounted(() => {
  const map = L.map("map", {
    center: L.latLng(28.50291, -15.88168),
    zoom: 8.4,
    maxZoom: 13,
    minZoom: 2,
    scrollWheelZoom: false,
    maxBounds: [
      [-85, -180], // Coordenadas suroeste (límite inferior izquierdo)
      [85, 180]    // Coordenadas noreste (límite superior derecho)
    ],
    maxBoundsViscosity: 1.0,
    
  });

  const mtLayer = new L.MaptilerLayer({
    apiKey: "CNX23CDiEaOjDZ7zvUIS",
    style:
      "https://api.maptiler.com/maps/9acac1d3-e6e0-404c-8213-7da3ad245870/style.json",
    noWrap: true,
  }).addTo(map);

  let mapTile = document.querySelector(".leaflet-control-container");
  let mapTilerLink = document.querySelector(
    'a[href="https://www.maptiler.com"]'
  );

  if (mapTile && mapTile.lastElementChild) {
    mapTile.removeChild(mapTile.lastElementChild);
  }
  if (mapTilerLink) {
    mapTilerLink.remove();
  }

  // Crear un grupo de marcadores
  /* const markers = L.markerClusterGroup();  */

  /* const  markers = L.markerClusterGroup({
	spiderfyOnMaxZoom: false,
	showCoverageOnHover: false,
	zoomToBoundsOnClick: false
}); 
 */

  const markers = L.markerClusterGroup({
    iconCreateFunction: function (cluster) {
      const count = cluster.getChildCount();
      const size = count < 10 ? "30px" : count < 20 ? "35px" : "40px";
      const color = count < 10 ? "#d8c1e6" : count < 20 ? "#a87db0" : "#ffe4ff";
      const borderColor =
        count < 10 ? "#a87db0" : count < 20 ? "#d8c1e6" : "#d8c1e6";

      return L.divIcon({
        html: `<div style="background-color: ${color}; width: ${size}; height: ${size}; border: 2px solid ${borderColor}; border-radius: 20px; display: flex; align-items: center; justify-content: center; color: black; font-weight: bold;">${count}</div>`,
        className: "marker-cluster-custom",
        iconSize: L.point(size.replace("px", ""), size.replace("px", "")),
      });
    },
    zoomToBoundsOnClick: false
  });
  markers.on("clusterclick", function (event) {
    event.originalEvent.stopPropagation()
    showCard.value = false
    showList.value = true
    const childMarkers = event.layer.getAllChildMarkers();
    listData.value = childMarkers.map((marker) => {
      const person = people.find(
        (p) =>
          p.location[0] === marker.getLatLng().lat &&
          p.location[1] === marker.getLatLng().lng
      );
     
      
      return {
        id: marker._leaflet_id, // Asignar un ID único para cada marcador
        ...person
      };
    });
  });

  people.forEach((person) => {
    const customIcon = renderIcon(); // Usar la función para generar el ícono
    const marker = L.marker(person.location, { icon: customIcon });
    /* const popupContent = `
      <strong>${person.firstName} ${person.lastName}</strong><br>
      Sector: ${person.sectors.join(", ")}
    `; */

    //marker.bindPopup(popupContent);
    // Agregar un evento de clic al marcador para mostrar la lista
    marker.on("click", () => {
      target.value = 
        {
          id: marker._leaflet_id,
          ...person
        }
        
        showList.value = false

      showCard.value = true; // Mostrar la lista
    });

    markers.addLayer(marker);
  });
  map.addLayer(markers);
});
</script>

<style scoped>
#map {
  height: 800px; /* Asegúrate de tener una altura definida */
  width: 50vw; /* O define un ancho específico */
  border: 2px solid black;
}
/* Cambiar el color de fondo de los clusters */
.marker-cluster {
  background-color: rgba(255, 0, 0, 0.6); /* Cambia a rojo, por ejemplo */
  border-radius: 20px; /* Mantiene el borde redondeado */
  border: 2px solid rgba(255, 255, 255, 0.8); /* Agrega un borde blanco */
}

/* Cambiar el estilo de los clusters pequeños */
.marker-cluster-small {
  background-color: rgba(0, 255, 0, 0.6); /* Cambia a verde, por ejemplo */
}

/* Personalizar el texto dentro de los clusters */
.marker-cluster div {
  width: 30px; /* Puedes ajustar el tamaño */
  height: 30px; /* Puedes ajustar el tamaño */
  margin-left: 0; /* Ajusta el margen según sea necesario */
  margin-top: 0; /* Ajusta el margen según sea necesario */
  text-align: center; /* Centra el texto */
  border-radius: 15px; /* Mantiene el borde redondeado */
  font: 14px "Helvetica Neue", Arial, Helvetica, sans-serif; /* Ajusta la fuente y tamaño */
  color: white; /* Cambia el color del texto */
}

/* Cambiar el color de fondo de los clusters pequeños */
.marker-cluster-small div {
  background-color: rgba(0, 0, 255, 0.6); /* Cambia a azul, por ejemplo */
}

.leaflet-marker-icon {
  color: red;
}

.custom-icon {
  width: 32px; /* Asegúrate de que el tamaño sea adecuado */
  height: 32px; /* Asegúrate de que el tamaño sea adecuado */
}
</style>
