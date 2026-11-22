<template>
  <h1>Explorateur de pays</h1>

  <p class="loading" v-show="loading">Chargement des pays...</p>
  <p class="error" v-if="error" role="alert">{{ error }}</p>

  <div class="toolbar">
    <input v-model="recherche" placeholder="Rechercher un pays..." aria-label="Rechercher un pays" />
  </div>

  <p v-if="!loading && !error && paysAffiches.length === 0">Aucun pays trouvé.</p>
  <div class="grid" v-if="!loading && !error">
    <CountryCard v-for="country in paysAffiches" :key="country.alpha3Code" :country="country" />
  </div>
</template>

<script>
import CountryCard from '../components/CountryCard.vue'

const API_URL = 'https://countries.dev/countries'

export default {
  name: 'CountryListView',
  components: { CountryCard },
  data() {
    return {
      countries: [],
      loading: true,
      error: null,
      recherche: ''
    }
  },
  computed: {
    paysAffiches() {
      const recherche = this.recherche.trim().toLowerCase()
      return this.countries.filter(country =>
        (country.name || '').toLowerCase().includes(recherche)
      )
    }
  },
  methods: {
    async chargerPays() {
      this.loading = true
      this.error = null
      try {
        const reponse = await fetch(API_URL)
        if (!reponse.ok) throw new Error(`Erreur HTTP ${reponse.status}`)
        const donnees = await reponse.json()
        if (!Array.isArray(donnees)) throw new Error('Format de réponse inattendu')
        this.countries = donnees
      } catch (e) {
        this.countries = []
        this.error = 'Impossible de charger les pays.'
        console.error(e)
      } finally {
        this.loading = false
      }
    }
  },
  mounted() {
    this.chargerPays()
  }
}
</script>
