<template>
  <h1>Annuaire des étudiants</h1>

  <!-- TODO A complété : formulaire d'ajout -->
  <form @submit.prevent="ajouterEtudiant">
    <label>
      Prénom :
      <input v-model.trim="newStudent.firstName" type="text" required />
    </label>
    <label>
      Nom :
      <input v-model.trim="newStudent.lastName" type="text" required />
    </label>
    <label>
      Filière :
      <input v-model.trim="newStudent.filiere" type="text" />
    </label>
    <label>
      Année :
      <input v-model.number="newStudent.year" type="number" min="1" />
    </label>
    <label>
      Email :
      <input v-model.trim="newStudent.email" type="email" />
    </label>
    <button type="submit">Ajouter</button>
  </form>

  <!-- TODO B complété : recherche et tri -->
  <div class="toolbar">
    <input
      v-model="recherche"
      type="search"
      placeholder="Rechercher par nom ou prénom"
    />
    <select v-model="tri">
      <option value="lastName">Trier par nom</option>
      <option value="filiere">Trier par filière</option>
    </select>
  </div>

  <!-- TODO B complété : affichage de la liste filtrée et triée -->
  <StudentCard
    v-for="student in etudiantsAffiches"
    :key="student.id"
    :student="student"
  />
  <p v-if="etudiantsAffiches.length === 0">Aucun étudiant trouvé.</p>
</template>

<script>
import StudentCard from '../components/StudentCard.vue'
import { store } from '../store/students.js'

export default {
  name: 'StudentListView',
  components: { StudentCard },
  data() {
    return {
      store,
      recherche: '',
      tri: 'lastName',
      newStudent: { firstName: '', lastName: '', filiere: '', year: 1, email: '' }
    }
  },
  computed: {
    // Filtrer par nom/prénom (sans tenir compte de la casse), puis trier.
    etudiantsAffiches() {
      const texte = this.recherche.trim().toLocaleLowerCase('fr')

      return this.store.students
        .filter(student =>
          student.firstName.toLocaleLowerCase('fr').includes(texte) ||
          student.lastName.toLocaleLowerCase('fr').includes(texte)
        )
        .slice() // On ne trie pas directement le tableau partagé du store.
        .sort((a, b) =>
          a[this.tri].localeCompare(b[this.tri], 'fr', { sensitivity: 'base' })
        )
    }
  },
  methods: {
    // Ajouter avec un nouvel ID ; prénom et nom obligatoires.
    ajouterEtudiant() {
      const firstName = this.newStudent.firstName.trim()
      const lastName = this.newStudent.lastName.trim()

      if (!firstName || !lastName) {
        alert('Veuillez renseigner le prénom et le nom.')
        return
      }

      const nouvelId = Math.max(0, ...this.store.students.map(s => s.id)) + 1
      this.store.students.push({
        id: nouvelId,
        firstName,
        lastName,
        filiere: this.newStudent.filiere,
        year: this.newStudent.year,
        email: this.newStudent.email
      })

      this.newStudent = {
        firstName: '', lastName: '', filiere: '', year: 1, email: ''
      }
    }
  }
}
</script>
