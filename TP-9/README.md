# TP 9 — Météo en direct (Vue.js)

Projet réalisé à partir du starter du professeur (les 6 TODO du script sont complétés, sans modifier le template).

## Installation

```bash
npm install
cp .env.example .env
```

Dans `.env`, remplacer `votre_cle_ici` par votre clé OpenWeatherMap obtenue depuis https://home.openweathermap.org/api_keys. Puis :

```bash
npm run dev
```

## Fonctionnalités

- Recherche différée (debounce 400 ms) et protection contre les réponses de recherche obsolètes
- Géocodage et suggestions (gestion 401, 429, liste vide et erreurs réseau)
- Sélection d'une ville et historique limité à 5 (conservé via `localStorage`)
- Météo actuelle et prévisions chargées en parallèle avec `Promise.all`
- Alternance Celsius/Fahrenheit en rechargeant les données
- Prévisions agrégées par date avec min/max et icône de la tranche la plus proche de midi

## Important

Ne jamais envoyer `.env` sur GitHub : le fichier est ignoré par Git.
L'usage de `VITE_OWM_API_KEY` rend néanmoins la clé visible dans les requêtes et le JavaScript du navigateur : ce n'est pas une protection pour un secret côté client. Pour une application de production, utiliser un backend/proxy et limiter la clé côté fournisseur.
