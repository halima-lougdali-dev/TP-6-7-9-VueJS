<template>
  <div v-if="student">
    <h1>{{ student.firstName }} {{ student.lastName }}</h1>

    <!-- TODO D complété : édition directe de l'objet du store -->
    <p>
      <label>Filière : <input v-model="student.filiere" type="text" /></label>
    </p>
    <p>
      <label>Année : <input v-model.number="student.year" type="number" min="1" /></label>
    </p>
    <p>
      <label>Email : <input v-model="student.email" type="email" /></label>
    </p>
  </div>
  <div v-else>
    <p>Étudiant introuvable.</p>
  </div>

  <button @click="$router.back()">Retour</button>
</template>

<script>
import { store } from '../store/students.js'

export default {
  name: 'StudentDetailView',
  computed: {
    // TODO C complété : paramètre de route (string) converti en nombre.
    student() {
      const id = Number(this.$route.params.id)
      return store.students.find(s => s.id === id)
    }
  }
}
</script>
