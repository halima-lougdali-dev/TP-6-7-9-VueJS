<template>
  <router-link :to="`/pays/${country.alpha3Code}`" class="country-card">
    <img v-if="drapeauUrl" :src="drapeauUrl" :alt="`Drapeau de ${country.name}`" />
    <span v-else class="flag-emoji" role="img" :aria-label="`Drapeau de ${country.name}`">{{ country.flag || '🌍' }}</span>
    <h3>{{ country.name }}</h3>
    <p>{{ country.capital || '—' }}</p>
    <p>{{ country.region || '—' }} · {{ Number(country.population || 0).toLocaleString('fr-FR') }} hab.</p>
  </router-link>
</template>

<script>
export default {
  name: 'CountryCard',
  props: {
    country: { type: Object, required: true }
  },
  computed: {
    drapeauUrl() {
      return this.country.flags?.svg || this.country.flags?.png ||
        (typeof this.country.flag === 'string' && this.country.flag.startsWith('http') ? this.country.flag : null)
    }
  }
}
</script>
