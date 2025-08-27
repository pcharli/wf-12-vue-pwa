<script setup>
import L from 'leaflet'
import { onMounted, ref } from 'vue'

const carte = ref(null)

onMounted(() => {
  // Obtenir la position de l'utilisateur
  navigator.geolocation.watchPosition(
    (position) => {
      const { latitude, longitude } = position.coords

      const map = L.map(carte.value).setView([latitude, longitude], 13)

      // Ajouter le fond de carte OpenStreetMap
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: 'Dingo',
      }).addTo(map)

      // Ajouter un marqueur à la position actuelle
      L.marker([latitude, longitude]).addTo(map).bindPopup('Vous êtes ici.').openPopup()
    },
    (error) => {
      console.error('Erreur de géolocalisation :', error)
    },
  )
})
</script>

<template>
  <div id="carte" ref="carte"></div>
</template>

<style scoped>
#carte {
  height: 400px;
  width: 100%;
}
</style>
