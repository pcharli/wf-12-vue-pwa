<script setup>
import InstallBtn from '@/components/InstallBtn.vue'
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'

const user = useUserStore()

const pseudo = ref('')
const router = useRouter()

const options = {
  enableHighAccuracy: true, // 🔹 active le GPS si dispo
  timeout: 5000, // 🔹 max 5 secondes
  maximumAge: 0, // 🔹 ne pas utiliser de cache
}
const init = () => {
  navigator.geolocation.getCurrentPosition(
    (position) => {
      const { latitude, longitude } = position.coords
      const objet = {
        data: {
          pseudo: pseudo.value,
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
          user.user_id = resp._id

          //redirection
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
    <input type="text" v-model="pseudo" />
    <button>Envoyer</button>
  </form>
</template>
