<template>
  <p class="loading" v-show="loading">Chargement...</p>
  <p class="error" v-if="error" role="alert">{{ error }}</p>

  <div v-if="!loading && country">
    <img v-if="drapeauUrl" :src="drapeauUrl" :alt="`Drapeau de ${country.name}`" style="width: 200px;" />
    <span v-else class="flag-emoji" role="img" :aria-label="`Drapeau de ${country.name}`">{{ country.flag }}</span>
    <h1>{{ country.name }}</h1>
    <p>Nom officiel : {{ country.name }}</p>
    <p>Capitale : {{ country.capital || '—' }}</p>
    <p>Région : {{ country.region || '—' }}</p>
    <p>Population : {{ Number(country.population || 0).toLocaleString('fr-FR') }} hab.</p>
    <p v-if="country.subregion">Sous-région : {{ country.subregion }}</p>
  </div>

  <button @click="$router.push('/')">Retour à la liste</button>
</template>

<script>
export default {
  name: 'CountryDetailView',
  data() {
    return {
      country: null,
      loading: true,
      error: null
    }
  },
  computed: {
    drapeauUrl() {
      if (!this.country) return null
      return this.country.flags?.svg || this.country.flags?.png ||
        (typeof this.country.flag === 'string' && this.country.flag.startsWith('http') ? this.country.flag : null)
    }
  },
  methods: {
    async chargerPays() {
      this.loading = true
      this.error = null
      this.country = null
      const code = encodeURIComponent(this.$route.params.code)
      const url = `https://countries.dev/alpha/${code}`

      try {
        const reponse = await fetch(url)
        if (!reponse.ok) throw new Error(`Erreur HTTP ${reponse.status}`)
        const donnees = await reponse.json()
        this.country = Array.isArray(donnees) ? donnees[0] : donnees
        if (!this.country || !this.country.name) throw new Error('Pays introuvable')
      } catch (e) {
        this.error = 'Impossible de charger les détails du pays.'
        console.error(e)
      } finally {
        this.loading = false
      }
    }
  },
  mounted() {
    this.chargerPays()
  },
  watch: {
    '$route.params.code'() {
      this.chargerPays()
    }
  }
}
</script>
