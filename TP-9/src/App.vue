<!--
  TP9 -- Meteo en direct.
  Le gabarit (template) est deja cable sur les donnees : il n'y a rien a
  modifier dans la partie template. Tout le travail est dans les methodes
  et la computed de la partie script, marquees TODO 1 a 6.
-->
<template>
  <div class="app">
    <header class="topbar">
      <h1>Météo en direct</h1>
      <button class="unit-toggle" @click="basculerUnite" v-if="ville">
        Afficher en {{ unite === 'metric' ? '°F' : '°C' }}
      </button>
    </header>

    <div class="search">
      <input
        v-model="query"
        @input="onInputRecherche"
        type="text"
        placeholder="Rechercher une ville... (ex: Safi, Paris, Tokyo)"
      />
      <p class="hint" v-if="chargementSuggestions">Recherche...</p>
      <p class="msg-error" v-if="erreurSuggestions">{{ erreurSuggestions }}</p>

      <ul class="suggestions" v-if="suggestions.length">
        <li v-for="s in suggestions" :key="s.lat + '-' + s.lon" @click="choisirVille(s)">
          {{ s.name }}<span v-if="s.state">, {{ s.state }}</span> ({{ s.country }})
        </li>
      </ul>
    </div>

    <div class="historique" v-if="historique.length">
      <span>Récent :</span>
      <button v-for="h in historique" :key="h.lat + '-' + h.lon" @click="choisirVille(h)">
        {{ h.name }}
      </button>
    </div>

    <div class="meteo" v-if="ville">
      <p class="loading" v-if="chargementMeteo">Chargement de la météo...</p>
      <p class="msg-error" v-else-if="erreurMeteo">{{ erreurMeteo }}</p>

      <div v-else-if="meteoActuelle" class="courante">
        <h2>{{ ville.name }}, {{ ville.country }}</h2>
        <div class="temp-actuelle">
          <img :src="iconeUrl(meteoActuelle.weather[0].icon)" :alt="meteoActuelle.weather[0].description" />
          <span class="valeur">{{ Math.round(meteoActuelle.main.temp) }}°</span>
          <span class="description">{{ meteoActuelle.weather[0].description }}</span>
        </div>
        <p class="details">
          Ressenti {{ Math.round(meteoActuelle.main.feels_like) }}° · Humidité {{ meteoActuelle.main.humidity }}%
        </p>
      </div>

      <div class="previsions" v-if="previsionsParJour.length">
        <div class="jour" v-for="j in previsionsParJour" :key="j.date">
          <span class="date">{{ j.date }}</span>
          <img :src="iconeUrl(j.icone)" :alt="j.description" />
          <span class="temps">{{ Math.round(j.min) }}° / {{ Math.round(j.max) }}°</span>
        </div>
      </div>
    </div>

    <p class="empty" v-if="!ville">Cherchez une ville pour voir sa météo.</p>
  </div>
</template>

<script>
// La cle est lue depuis .env (jamais committee -- voir .env.example et le sujet).
const API_KEY = import.meta.env.VITE_OWM_API_KEY
const GEO_URL = 'https://api.openweathermap.org/geo/1.0/direct'
const WEATHER_URL = 'https://api.openweathermap.org/data/2.5/weather'
const FORECAST_URL = 'https://api.openweathermap.org/data/2.5/forecast'

export default {
  name: 'App',
  data() {
    return {
      query: '',
      debounceTimer: null,
      suggestions: [],
      chargementSuggestions: false,
      erreurSuggestions: null,

      ville: null, // { name, country, state, lat, lon }
      historique: [],

      meteoActuelle: null,
      previsions: [], // tableau brut renvoye par /forecast (tranches de 3h)
      chargementMeteo: false,
      erreurMeteo: null,

      unite: 'metric' // 'metric' (°C) | 'imperial' (°F)
    }
  },
  computed: {
    // TODO 6 : une seule prevision par date, min/max des tranches de 3 h.
    previsionsParJour() {
      const jours = {}

      for (const tranche of this.previsions) {
        const date = tranche.dt_txt.split(' ')[0]
        const heure = Number(tranche.dt_txt.split(' ')[1].slice(0, 2))
        if (!jours[date]) {
          jours[date] = {
            date,
            min: tranche.main.temp_min,
            max: tranche.main.temp_max,
            icone: tranche.weather[0].icon,
            description: tranche.weather[0].description,
            distanceMidi: Math.abs(heure - 12)
          }
        } else {
          const jour = jours[date]
          jour.min = Math.min(jour.min, tranche.main.temp_min)
          jour.max = Math.max(jour.max, tranche.main.temp_max)
          if (Math.abs(heure - 12) < jour.distanceMidi) {
            jour.icone = tranche.weather[0].icon
            jour.description = tranche.weather[0].description
            jour.distanceMidi = Math.abs(heure - 12)
          }
        }
      }

      return Object.values(jours)
        .sort((a, b) => a.date.localeCompare(b.date))
        .map(({ distanceMidi, ...jour }) => jour)
    }
  },
  methods: {
    iconeUrl(code) {
      return `https://openweathermap.org/img/wn/${code}@2x.png`
    },

    // TODO 1 : annule le precedent minuteur avant de patienter 400 ms.
    onInputRecherche() {
      clearTimeout(this.debounceTimer)
      this.debounceTimer = null
      this.suggestions = []
      this.erreurSuggestions = null
      this.chargementSuggestions = false
      // Invalide aussi les eventuelles reponses d'une ancienne recherche.
      this.rechercheId = (this.rechercheId || 0) + 1

      if (!this.query.trim()) return

      this.debounceTimer = setTimeout(() => {
        this.rechercherVilles()
      }, 400)
    },

    // TODO 2 : recherche via l'API de geocodage et gestion des erreurs.
    async rechercherVilles() {
      const recherche = this.query.trim()
      if (!recherche) return
      const id = (this.rechercheId || 0) + 1
      this.rechercheId = id
      this.chargementSuggestions = true
      this.erreurSuggestions = null

      try {
        if (!API_KEY || API_KEY === 'votre_cle_ici') {
          throw new Error('Clé API absente : configurez VITE_OWM_API_KEY dans .env.')
        }
        const url = `${GEO_URL}?q=${encodeURIComponent(recherche)}&limit=5&appid=${API_KEY}`
        const reponse = await fetch(url)
        if (reponse.status === 401) throw new Error('Clé API invalide ou absente (401).')
        if (reponse.status === 429) throw new Error('Quota API dépassé (429). Réessayez plus tard.')
        if (!reponse.ok) throw new Error(`Erreur réseau/API (${reponse.status}).`)

        const villes = await reponse.json()
        if (id !== this.rechercheId) return
        this.suggestions = villes
        if (villes.length === 0) {
          this.erreurSuggestions = 'Aucune ville trouvée pour cette recherche.'
        }
      } catch (erreur) {
        if (id !== this.rechercheId) return
        this.suggestions = []
        this.erreurSuggestions = erreur.message || 'Impossible de rechercher les villes.'
      } finally {
        if (id === this.rechercheId) this.chargementSuggestions = false
      }
    },

    // TODO 3 : sélection d'une ville et historique des cinq dernières villes.
    choisirVille(ville) {
      clearTimeout(this.debounceTimer)
      this.debounceTimer = null
      this.rechercheId = (this.rechercheId || 0) + 1
      this.suggestions = []
      this.query = ''
      this.erreurSuggestions = null
      this.chargementSuggestions = false
      this.ville = ville

      this.historique = [
        ville,
        ...this.historique.filter(v => v.lat !== ville.lat || v.lon !== ville.lon)
      ].slice(0, 5)
      // Bonus : l'historique est conservé entre deux visites.
      try {
        localStorage.setItem('tp9-historique', JSON.stringify(this.historique))
      } catch (erreur) {
        console.warn('Historique local indisponible :', erreur)
      }
      this.chargerMeteo()
    },

    // TODO 4 : météo actuelle et prévisions lancées simultanément.
    async chargerMeteo() {
      if (!this.ville) return
      const { lat, lon } = this.ville
      const unite = this.unite
      const id = (this.meteoId || 0) + 1
      this.meteoId = id
      this.chargementMeteo = true
      this.erreurMeteo = null
      this.meteoActuelle = null
      this.previsions = []

      try {
        if (!API_KEY || API_KEY === 'votre_cle_ici') {
          throw new Error('Clé API absente : renseignez votre fichier .env.')
        }
        const params = `lat=${lat}&lon=${lon}&appid=${API_KEY}&units=${unite}&lang=fr`
        const [reponseMeteo, reponsePrevisions] = await Promise.all([
          fetch(`${WEATHER_URL}?${params}`),
          fetch(`${FORECAST_URL}?${params}`)
        ])

        for (const reponse of [reponseMeteo, reponsePrevisions]) {
          if (reponse.status === 401) throw new Error('Clé API invalide ou absente (401).')
          if (reponse.status === 429) throw new Error('Quota API dépassé (429). Réessayez plus tard.')
          if (!reponse.ok) throw new Error(`Impossible de charger la météo (${reponse.status}).`)
        }

        const [meteo, previsions] = await Promise.all([
          reponseMeteo.json(),
          reponsePrevisions.json()
        ])
        if (id !== this.meteoId) return
        if (!Array.isArray(previsions.list)) throw new Error('Format des prévisions inattendu.')
        this.meteoActuelle = meteo
        this.previsions = previsions.list
      } catch (erreur) {
        if (id !== this.meteoId) return
        this.erreurMeteo = erreur.message || 'Erreur de connexion. Vérifiez votre réseau.'
      } finally {
        if (id === this.meteoId) this.chargementMeteo = false
      }
    },

    // TODO 5 : changement d'unité avec nouvelle interrogation des API.
    basculerUnite() {
      this.unite = this.unite === 'metric' ? 'imperial' : 'metric'
      if (this.ville) this.chargerMeteo()
    }
  },
  mounted() {
    // Bonus : restauration de l'historique local.
    try {
      const valeurs = JSON.parse(localStorage.getItem('tp9-historique') || '[]')
      if (Array.isArray(valeurs)) {
        this.historique = valeurs.filter(v => v && typeof v.lat === 'number' && typeof v.lon === 'number').slice(0, 5)
      }
    } catch (erreur) {
      console.warn('Historique local illisible :', erreur)
    }
  },
  beforeUnmount() {
    clearTimeout(this.debounceTimer)
    this.rechercheId = (this.rechercheId || 0) + 1
    this.meteoId = (this.meteoId || 0) + 1
  }
}
</script>
