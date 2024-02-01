# Démarrage rapide avec RTK Query

> [!TIP]
> Ce que vous apprendrez
> 
> - Comment configurer et utiliser la fonctionnalité de récupération de données "RTK Query" de Redux Toolkit

> [!WARNING]
> Prérequis
>
> - Compréhension des [termes et concepts de Redux](https://redux.js.org/tutorials/fundamentals/part-2-concepts-data-flow)

## Introduction[](https://redux-toolkit.js.org/tutorials/rtk-query#introduction "Lien direct vers la section")

Bienvenue dans le tutoriel Redux Toolkit Query ! **Ce tutoriel vous introduira brièvement à la capacité de récupération de données "RTK Query" de Redux Toolkit et vous apprendra comment commencer à l'utiliser correctement**.

> [!NOTE]
> Pour un tutoriel plus approfondi sur RTK Query, consultez le tutoriel complet ["Essentiels de Redux"](https://redux.js.org/tutorials/essentials/part-7-rtk-query-basics) sur le site de documentation principal de Redux.

RTK Query est un outil avancé de récupération et de mise en cache de données, conçu pour simplifier les cas courants de chargement de données dans une application web. RTK Query lui-même est construit sur le noyau de Redux Toolkit et exploite les API de RTK telles que [`createSlice`](https://redux-toolkit.js.org/api/createSlice) et [`createAsyncThunk`](https://redux-toolkit.js.org/api/createAsyncThunk) pour implémenter ses fonctionnalités.

RTK Query est inclus dans le package `@reduxjs/toolkit` en tant qu'extension supplémentaire. Vous n'êtes pas obligé d'utiliser les API RTK Query lorsque vous utilisez Redux Toolkit, mais nous pensons que de nombreux utilisateurs bénéficieront de la récupération de données et de la mise en cache de RTK Query dans leurs applications.

### Comment lire ce tutoriel[](https://redux-toolkit.js.org/tutorials/rtk-query#how-to-read-this-tutorial "Lien direct vers la section")

Pour ce tutoriel, nous supposons que vous utilisez Redux Toolkit avec React, mais vous pouvez également l'utiliser avec d'autres couches d'interface utilisateur. Les exemples sont basés sur [une structure de dossier typique de Create-React-App](https://create-react-app.dev/docs/folder-structure) où tout le code de l'application est dans un dossier `src`, mais les modèles peuvent être adaptés à n'importe quel projet ou configuration de dossier que vous utilisez.

## Configuration de votre magasin et du service API[](https://redux-toolkit.js.org/tutorials/rtk-query#setting-up-your-store-and-api-service "Lien direct vers la section")

Pour voir comment fonctionne RTK Query, parcourons un exemple d'utilisation de base. Pour cet exemple, nous supposerons que vous utilisez React et souhaitez utiliser les hooks React générés automatiquement par RTK Query.

### Créer un service API[](https://redux-toolkit.js.org/tutorials/rtk-query#create-an-api-service "Lien direct vers la section")

Tout d'abord, créons une définition de service qui interroge la [PokeAPI](https://pokeapi.co/) disponible publiquement.

<details>
<summary>TypeScript</summary>
src/services/pokemon.ts

```tsx
// Besoin d'utiliser le point d'entrée spécifique à React pour importer createApi
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import type { Pokemon } from './types'

// Définir un service en utilisant une URL de base et des points d'extrémité attendus
export const pokemonApi = createApi({
  reducerPath: 'pokemonApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://pokeapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getPokemonByName: builder.query<Pokemon, string>({
      query: (name) => `pokemon/${name}`,
    }),
  }),
})

// Exporter des hooks pour une utilisation dans les composants fonctionnels, qui sont
// générés automatiquement en fonction des points d'extrémité définis
export const { useGetPokemonByNameQuery } = pokemonApi
```
</details>

<details>
<summary>JavaScript</summary>

src/services/pokemon.js

```js
// Besoin d'utiliser le point d'entrée spécifique à React pour importer createApi
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

// Définir un service en utilisant une URL de base et des points d'extrémité attendus
export const pokemonApi = createApi({
  reducerPath: 'pokemonApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://pokeapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getPokemonByName: builder.query({
      query: (name) => `pokemon/${name}`,
    }),
  }),
})

// Exporter des hooks pour une utilisation dans les composants fonctionnels, qui sont
// générés automatiquement en fonction des points d'extrémité définis
export const { useGetPokemonByNameQuery } = pokemonApi
```
</details>

Avec RTK Query, vous définissez généralement l'ensemble de votre API à un seul endroit. Cela est très probablement différent de ce que vous voyez avec d'autres bibliothèques telles que `swr` ou `react-query`, et il existe plusieurs raisons à cela. Notre point de vue est qu'il est _beaucoup_ plus facile de suivre le comportement des requêtes, de l'invalidation du cache et de la configuration générale de l'application lorsqu'ils sont tous au même endroit central, par rapport au fait d'avoir un certain nombre de hooks personnalisés dans différents fichiers de votre application.

> [!TIP]
> En général, vous ne devriez avoir qu'une seule tranche API par URL de base avec laquelle votre application doit communiquer. Par exemple, si votre site récupère des données à la fois depuis `/api/posts` et `/api/users`, vous auriez une seule tranche API avec `/api/` comme URL de base, et des définitions d'extrémité séparées pour `posts` et `users`. Cela vous permet de tirer pleinement parti du [rechargement automatique](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching) en définissant des [relations](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching#tags) [d'étiquettes](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching#tags) entre les extrémités.
> 
> Pour des raisons de maintenabilité, vous pouvez souhaiter répartir les définitions d'extrémité sur plusieurs fichiers, tout en maintenant une seule tranche API qui inclut toutes ces extrémit

és. Consultez la [division de code](https://redux-toolkit.js.org/rtk-query/usage/code-splitting) pour savoir comment utiliser la propriété `injectEndpoints` pour injecter les extrémités API depuis d'autres fichiers dans une seule définition de tranche API.

### Ajouter le service à votre magasin[](https://redux-toolkit.js.org/tutorials/rtk-query#add-the-service-to-your-store "Lien direct vers la section")

Un service RTKQ génère un "slice reducer" qui doit être inclus dans le réducteur racine de Redux, et un middleware personnalisé qui gère la récupération des données. Les deux doivent être ajoutés au magasin Redux.

<details>

<summary>TypeScript</summary>

src/store.ts

```ts
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

</details>

<details>

<summary>JavaScript</summary>

src/store.js

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

</details>

### Enveloppez votre application avec le `Provider`[](https://redux-toolkit.js.org/tutorials/rtk-query#wrap-your-application-with-the-provider "Lien direct vers la section")

Si ce n'est pas déjà fait, suivez le modèle standard pour fournir le magasin Redux au reste de l'arborescence des composants de votre application React :

<details>

<summary>TypeScript</summary>

src/index.tsx

```ts
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

</details>

<details>

<summary>JavaScript</summary>

src/index.jsx

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

</details>

## Utiliser la requête dans un composant[](https://redux-toolkit.js.org/tutorials/rtk-query#use-the-query-in-a-component "Lien direct vers la section")

Une fois qu'un service a été défini, vous pouvez importer les hooks pour effectuer une requête.

<details>

<summary>TypeScript</summary>

src/App.tsx

```ts
import * as React from 'react'
import { useGetPokemonByNameQuery } from './services/pokemon'

export default function App() {
  // L'utilisation d'un hook de requête récupère automatiquement les données et renvoie les valeurs de la requête
  const { data, error, isLoading } = useGetPokemonByNameQuery('bulbasaur')
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
          <img src={data.sprites.front_shiny} alt={data.species.name} />
        </>
      ) : null}
    </div>
  )
}
```

</details>

<details>

<summary>JavaScript</summary>

src/App.jsx

```js
import * as React from 'react'
import { useGetPokemonByNameQuery } from './services/pokemon'

export default function App() {
  // L'utilisation d'un hook de requête récupère automatiquement les données et renvoie les valeurs de la requête
  const { data, error, isLoading } = useGetPokemonByNameQuery('bulbasaur')
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
          <img src={data.sprites.front_shiny} alt={data.species.name} />
        </>
      ) : null}
    </div>
  )
}
```

</details>

Lors de la réalisation d'une requête, vous pouvez suivre l'état de plusieurs manières. Vous pouvez toujours vérifier `data`, `status` et `error` pour déterminer la bonne interface utilisateur à rendre. De plus, `useQuery` fournit également des booléens utilitaires tels que `isLoading`, `isFetching`, `isSuccess`, et `isError` pour la dernière requête.

#### Exemple de base

App.tsx

```tsx
import './styles.css'
import { useGetPokemon

ByNameQuery } from './services/pokemon'

export default function App() {
  const { data, error, isLoading } = useGetPokemonByNameQuery('bulbasaur')

  return (
    <div className="App">
      {error ? (
        <>Oh non, une erreur s'est produite</>
      ) : isLoading ? (
        <>Chargement...</>
      ) : data ? (
        <>
          <h3>{data.species.name}</h3>
          <img src={data.sprites.front_shiny} alt={data.species.name} />
        </>
      ) : null}
    </div>
  )
}
```

D'accord, c'est intéressant... mais que se passe-t-il si vous voulez afficher plusieurs Pokémon en même temps ? Que se passe-t-il si plusieurs composants chargent le même Pokémon ?

#### Exemple avancé[]

RTK Query veille à ce que tout composant qui s'abonne à la même requête utilise toujours les mêmes données. RTK Query déduplique automatiquement les requêtes, vous n'avez donc pas à vous soucier de vérifier les requêtes en cours et des optimisations de performances de votre côté. Évaluons le bac à sable ci-dessous - assurez-vous de consulter le panneau Réseau dans les outils de développement de votre navigateur. Vous verrez 3 requêtes, même s'il y a 4 composants abonnés - `bulbasaur` ne fait qu'une seule requête, et l'état de chargement est synchronisé entre les deux composants. Amusez-vous, essayez de changer la valeur du menu déroulant de `Off` à `1s` pour voir ce comportement se poursuivre lorsque la requête est relancée.

`App.tsx`

```tsx
import './styles.css'
import { Pokemon } from './Pokemon'
import { useState } from 'react'

const pokemon = ['bulbasaur', 'pikachu', 'ditto', 'bulbasaur']

export default function App() {
  const [pollingInterval, setPollingInterval] = useState(0)

  return (
    <div className="App">
      <select
        onChange={(change) => setPollingInterval(Number(change.target.value))}
      >
        <option value={0}>Off</option>
        <option value={1000}>1s</option>
        <option value={5000}>5s</option>
      </select>
      <div>
        {pokemon.map((poke, index) => (
          <Pokemon key={index} name={poke} pollingInterval={pollingInterval} />
        ))}
      </div>
    </div>
  )
}
```