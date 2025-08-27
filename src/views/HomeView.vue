<script setup>
import InstallBtn from '@/components/InstallBtn.vue'
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
//store users
const user = useUserStore()
//router
const router = useRouter()

//new peudo
const newPseudo = ref('')

// option de géolocalisation
const options = {
  enableHighAccuracy: true, // 🔹 active le GPS si dispo
  timeout: 5000, // 🔹 max 5 secondes
  maximumAge: 0, // 🔹 ne pas utiliser de cache
}

//si on soumet le formulaire
const init = () => {
  navigator.geolocation.getCurrentPosition(
    (position) => {
      const { latitude, longitude } = position.coords
      const objet = {
        data: {
          pseudo: newPseudo.value,
          lat: latitude,
          long: longitude,
        },
      }
      fetch('https://ingrwf12.cepegra-frontend.xyz/cockpit2/api/content/item/users', {
        method: 'post',
        body: JSON.stringify(objet),
        headers: {
          'Content-type': 'application/json',
        },
      })
        .then((resp) => resp.json())
        .then((resp) => {
          console.log(resp)
          //stocker l'_id dans le store
          user.user_id = resp._id

          //redirection vers la carte
          router.push({ name: 'carte' })
        })
    },
    errorPos,
    options,
  )
}

const errorPos = (err) => console.log(err)
</script>
<template>
  <h1>Home</h1>
  <InstallBtn></InstallBtn>

  <form action="" @submit.prevent="init">
    <label for="">Pseudo</label>
    <input type="text" v-model="newPseudo" />
    <button>Envoyer</button>
  </form>
</template>
