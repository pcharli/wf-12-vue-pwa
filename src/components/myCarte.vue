<script setup>
//import de Leaflet
import L from 'leaflet'
import { onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'
// 👉 importer les assets d’icône pour que Vite les bundle
//router
const router = useRouter()

//import de la css
import 'leaflet/dist/leaflet.css'

//import des icônes de Leaflet
import iconUrl from 'leaflet/dist/images/marker-icon.png'
import iconRetinaUrl from 'leaflet/dist/images/marker-icon-2x.png'
import shadowUrl from 'leaflet/dist/images/marker-shadow.png'
//import du store users
import { useUserStore } from '@/stores/user'
const user = useUserStore()
console.log(user.user_id)

//lier icônes importées à Leaflet
// 👉 fixer les URLs par défaut
L.Icon.Default.mergeOptions({
  iconUrl,
  iconRetinaUrl,
  shadowUrl,
})

//la div caretDiv
const carteDiv = ref(null)
//va récupérer mon marker
let myMarker = null
//récupère la carte Leaflet
let map = null
//calque pour mettre mles markers des autres
let voisinsLayer = L.layerGroup()

//une fois le DOM réel chargé
onMounted(() => {
  //options pour géolocalisation
  const options = {
    enableHighAccuracy: true, // 🔹 active le GPS si dispo
    timeout: 5000, // 🔹 max 5 secondes
    maximumAge: 0, // 🔹 ne pas utiliser de cache
  }
  //fond de carte de base
  const osm = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: 'Pcharli',
  })
  // fond de carte statellite
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
      // injecton de la carte dans la div, centrée sur ma position au zoom 13
      map = L.map(carteDiv.value).setView([latitude, longitude], 13)

      // après avoir créé la map, ajout du calque créé
      voisinsLayer.addTo(map)
      //ajout du fond de carte
      osm.addTo(map) // couche par défaut
      //ajout du switch de fonds de carte
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

  // on écoute les changements de position de l'utilisateur
  navigator.geolocation.watchPosition(
    (position) => {
      //si on a pas encore de marker sur la carte, on return
      if (!myMarker) return // attendre la création dans getCurrentPosition

      const { latitude, longitude } = position.coords
      //changer la place du marker
      myMarker.setLatLng([latitude, longitude])
      // appeler une fonction pour éditer dans la database
      changePosition(position.coords)
    },
    (err) => {
      console.error('Erreur géolocalisation :', err)
    },
    options,
  ) //watch

  //appel de la fonction pour chercher les voisins
  lookVoisins()
})

const changePosition = (position) => {
  //ajax pour éditer ma position
  fetch('https://ingrwf12.cepegra-frontend.xyz/cockpit2/api/content/item/users', {
    method: 'post',
    headers: {
      'Content-type': 'application/json',
    },
    body: JSON.stringify({
      data: {
        lat: position.latitude,
        long: position.longitude,
        _id: user.user_id,
      },
    }),
  })
}
// rechercher les voisins
const lookVoisins = () => {
  //toutes les 5 sec.
  setInterval(() => {
    fetch('https://ingrwf12.cepegra-frontend.xyz/cockpit2/api/content/items/users')
      .then((resp) => resp.json())
      .then((resp) => {
        console.log(resp)
        //nettoyer le calque
        voisinsLayer.clearLayers()
        //pour chaque voisin
        resp.forEach((el) => {
          //si ce n'est pas moi (stocké dans le store)
          if (el._id != user.user_id) {
            //ajout du marker sur le calque
            L.marker([el.lat, el.long]).addTo(voisinsLayer).bindPopup(el.pseudo)
          }
        })
      })
  }, 5000)
}

const quit = () => {
  fetch('https://ingrwf12.cepegra-frontend.xyz/cockpit2/api/content/items/users' + user.user_id, {
    methods: 'delete',
  })
    .then((resp) => {
      resp.json()
    })
    .then((resp) => {
      console.log(resp)
      user.user_id = null
      router.put({ name: 'home' })
    })
}
</script>

<template>
  <!-- rel permet à VueJS de cibler le calque -->
  <div id="carte" ref="carteDiv"></div>
  <a href="#" @click="quit">Quitter</a>
</template>

<style scoped>
#carte {
  height: 400px;
  width: 100%;
}
</style>
