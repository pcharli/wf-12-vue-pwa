<script setup>
import L from 'leaflet'
import { onMounted, ref } from 'vue'
// 👉 importer les assets d’icône pour que Vite les bundle

import 'leaflet/dist/leaflet.css'

import iconUrl from 'leaflet/dist/images/marker-icon.png'
import iconRetinaUrl from 'leaflet/dist/images/marker-icon-2x.png'
import shadowUrl from 'leaflet/dist/images/marker-shadow.png'

import { useUserStore } from '@/stores/user'

const user = useUserStore()
console.log(user.user_id)

// 👉 fixer les URLs par défaut
L.Icon.Default.mergeOptions({
  iconUrl,
  iconRetinaUrl,
  shadowUrl,
})

const carteDiv = ref(null)

let myMarker = null

let markers = []

let map = null

let voisinsLayer = L.layerGroup()

onMounted(() => {
  const options = {
    enableHighAccuracy: true, // 🔹 active le GPS si dispo
    timeout: 5000, // 🔹 max 5 secondes
    maximumAge: 0, // 🔹 ne pas utiliser de cache
  }
  const osm = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: 'Pcharli',
  })

  const esri = L.tileLayer(
    'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
    {
      attribution: 'Pitou',
    },
  )
  // Obtenir la position de l'utilisateur
  navigator.geolocation.getCurrentPosition(
    (position) => {
      const { latitude, longitude } = position.coords

      map = L.map(carteDiv.value).setView([latitude, longitude], 13)

      // après avoir créé la map :
      voisinsLayer.addTo(map)

      osm.addTo(map) // couche par défaut

      L.control
        .layers({
          Plan: osm,
          Satellite: esri,
        })
        .addTo(map)

      // Ajouter un marqueur à la position actuelle
      myMarker = L.marker([latitude, longitude]).addTo(map).bindPopup('Vous êtes ici.').openPopup()
    },
    (error) => {
      console.error('Erreur de géolocalisation :', error)
    },
    options,
  ) //getCurrent

  navigator.geolocation.watchPosition(
    (position) => {
      if (!myMarker) return // attendre la création dans getCurrentPosition
      const { latitude, longitude } = position.coords
      myMarker.setLatLng([latitude, longitude])
      changePosition(position.coords)
    },
    (err) => {
      console.error('Erreur géolocalisation :', err)
    },
    options,
  ) //watch

  lookVoisins()
})

const changePosition = (position) => {
  fetch('https://ingrwf12.cepegra-frontend.xyz/cockpit2/api/content/item/users', {
    method: 'post',
    headers: {
      'Content-type': 'application/json',
    },
    body: JSON.stringify({
      data: {
        lat: position.latitude,
        long: position.longitude,
        _id: '68aee6d8fa399814cb8d04c7',
      },
    }),
  })
}

const lookVoisins = () => {
  setInterval(() => {
    fetch('https://ingrwf12.cepegra-frontend.xyz/cockpit2/api/content/items/users')
      .then((resp) => resp.json())
      .then((resp) => {
        console.log(resp)
        voisinsLayer.clearLayers()
        resp.forEach((el) => {
          if (el._id != user.user_id) {
            L.marker([el.lat, el.long]).addTo(voisinsLayer).bindPopup(el.pseudo)
          }
        })
      })
  }, 5000)
}
</script>

<template>
  <div id="carte" ref="carteDiv"></div>
</template>

<style scoped>
#carte {
  height: 400px;
  width: 100%;
}
</style>
