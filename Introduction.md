## 1. Présentation de RTK Query

Bienvenue dans la première section de notre vidéo d'introduction à Redux Toolkit Query (RTK Query). Dans cette section, nous allons explorer ce qu'est RTK Query et comprendre pourquoi il peut être un outil puissant dans le développement d'applications React.

### 1.1 Qu'est-ce que RTK Query ?

RTK Query est une extension de Redux Toolkit qui simplifie la gestion des requêtes côté client dans les applications React. Il fournit une approche rationalisée pour effectuer des requêtes réseau, gérer les données dans le store Redux, et synchroniser automatiquement l'état du cache avec l'état de l'application.

Voici quelques caractéristiques clés de RTK Query :

- **Simplicité d'utilisation :** RTK Query utilise des conventions par défaut pour réduire la configuration boilerplate. Cela permet aux développeurs de se concentrer davantage sur la logique de l'application plutôt que sur la gestion de l'état.

- **Normalisation des données :** Les données provenant des requêtes sont automatiquement normalisées dans le store Redux. Cela simplifie la gestion des relations entre les entités et améliore la cohérence des données.

- **Gestion des états :** RTK Query simplifie la gestion des états liés aux requêtes tels que le chargement, le succès et les erreurs. Les hooks générés automatiquement fournissent un moyen intuitif de gérer ces états dans les composants React.

- **Mise en cache automatique :** Les résultats des requêtes sont mis en cache automatiquement, réduisant ainsi le besoin de refaire des appels réseau inutiles.

- **Intégration transparente avec Redux Toolkit :** RTK Query s'intègre harmonieusement avec Redux Toolkit, profitant ainsi des fonctionnalités telles que le découpage (slicing) du store Redux.

### 1.2 Pourquoi utiliser RTK Query dans vos projets ?

- **Productivité accrue :** En suivant des conventions, RTK Query réduit la quantité de code que vous devez écrire pour gérer les requêtes et les mutations, ce qui accélère le processus de développement.

- **Meilleure organisation du code :** Les points d'extrémité d'API dans RTK Query vous permettent de regrouper vos requêtes et mutations de manière logique, ce qui rend le code plus organisé et maintenable.

- **Réduction des erreurs :** En normalisant automatiquement les données et en gérant les états de requête, RTK Query contribue à réduire les erreurs liées à la gestion de l'état dans votre application.

- **Réactivité :** Les résultats des requêtes sont réactifs, ce qui signifie que les composants sont automatiquement mis à jour lorsqu'une requête retourne de nouveaux résultats ou que des mutations sont effectuées.

# 2. Installation

## 2.1 Initialisation d'un projet React

```bash
npm create vite@latest nom_du_projet
```

## 2.2 Installation de Redux Toolkit et RTK Query

```bash
npm install react-redux @reduxjs/toolkit
```

## 3. Création d'API Endpoints

### 3.1 Définition des points d'extrémité pour les requêtes et mutations

Création de la définition du service qui interrogera l'API

Dans `src/` créer un dossier `services/` dans lequel se trouvera le fichier suivant intitulé `pokemon.js`

```js
// Import des fonctions createApi() et fetchBaseQuery()
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

// Définition du service avec l'URL de l'API
export const pokemonApi = createApi({
  reducerPath: 'pokemonApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://pokeapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getPokemonByName: builder.query({
      query: (name) => `pokemon/${name}`,
    }),
  }),
})

// Export des hooks qui nous permettent d'appliquer des actions spécifique au service.
// généré automatiquement sur la base des endpoints définis dans createApi()
export const { useGetPokemonByNameQuery } = pokemonApi
```

## 4. Configuration de votre application

### 4.1 Configuration du magasin (store)

Dans `src/` créer un fichier `store.js`

```js
import { configureStore } from '@reduxjs/toolkit'
// Ou de '@reduxjs/toolkit/query/react'
import { setupListeners } from '@reduxjs/toolkit/query'
import { pokemonApi } from './services/pokemon'

export const store = configureStore({
  reducer: {
    // Ajouter le réducteur généré en tant que tranche spécifique de niveau supérieur
    [pokemonApi.reducerPath]: pokemonApi.reducer,
  },
  // Ajouter le middleware API permet d'activer la mise en cache, l'invalidation, le sondage,
  // et d'autres fonctionnalités utiles de `rtk-query`.
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(pokemonApi.middleware),
})

// facultatif, mais nécessaire pour les comportements de refetchOnFocus/refetchOnReconnect
// voir la documentation de `setupListeners` - prend éventuellement un rappel en option en tant que 2e argument pour la personnalisation
setupListeners(store.dispatch)
```

### 4.2 Envelopper votre application dans le provider

Rendez-vous dans `index.js` pour appliquer le state global de votre application (le store)

```js
import * as React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'

import App from './App'
import { store } from './store'

const rootElement = document.getElementById('root')
render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
)
```

## 5. Utilisation de notre service mise en place avec RTK Query

Il est temps de profiter de toute l'installation qu'on a effectué jusqu'à présent en affichant nos données récupérées via l'API

```js
import * as React from 'react'
import { useGetPokemonByNameQuery } from './services/pokemon'

export default function App() {
  // L'utilisation d'un hook de requête récupère automatiquement les données et renvoie les valeurs de la requête
  const { data, error, isLoading } = useGetPokemonByNameQuery('pikachu')
  // Les hooks individuels sont également accessibles sous les points d'extrémité générés :
  // const { data, error, isLoading } = pokemonApi.endpoints.getPokemonByName.useQuery('bulbasaur')

  return (
    <div className="App">
      {error ? (
        <>Oh non, une erreur s'est produite</>
      ) : isLoading ? (
        <>Chargement...</>
      ) : data ? (
        <>
          <h3>{data.species.name}</h3>
          <img src={data.sprites.front_default} alt={data.species.name} />
        </>
      ) : null}
    </div>
  )
}
```