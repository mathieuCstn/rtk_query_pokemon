# RTK Query vue d'ensemble

> [!NOTE]
> CE QUE VOUS APPRENDREZ
> 
> - Ce qu'est RTK Query et les problèmes qu'il résout
> - Quelles API sont incluses dans RTK Query
> - Utilisation de base de RTK Query

**RTK Query** est un puissant outil de récupération et de mise en cache de données. Il est conçu pour simplifier les cas courants de chargement de données dans une application web, **éliminant le besoin d'écrire manuellement la logique de récupération et de mise en cache des données**.

RTK Query est **une extension facultative incluse dans le package Redux Toolkit**, et sa fonctionnalité est construite sur les autres API de Redux Toolkit.

## Motivation

Les applications web doivent normalement récupérer des données auprès d'un serveur afin de les afficher. Elles doivent également mettre à jour ces données, envoyer ces mises à jour au serveur et maintenir la synchronisation entre les données mises en cache sur le client et les données sur le serveur. Cette situation est rendue plus complexe par la nécessité de mettre en œuvre d'autres comportements utilisés dans les applications d'aujourd'hui :

- Suivi de l'état de chargement afin d'afficher un ecrant de chargement (barre de progression, spinner) sur l'interface utilisateur
- Éviter la duplication des requètes pour la même donnée
- Des mise à jour optimiste de l'interface utilisateur pour donner l'impréssion d'un chargement rapide
- Gérer les durées de vie des caches lorsque l'utilisateur interagit avec l'interface utilisateur

Le noyeau de rédux à toujours été très minimaliste - il apartient au développeur de concevoire et d'écrire la logique adaptées. Ce qui veut dire que Redux n'a jamais rien intégré pour aider à résoudre ces cas d'utilisation. La documentation de Redux a enseigné quelques [modèles communs pour dispatcher les actions autour du cycle de vie de la requête afin de suivre l'état de chargement et les résultats de la requête](https://redux.js.org/tutorials/fundamentals/part-7-standard-patterns#async-request-status), et l'[API `createAsyncThunk` de Redux Toolkit](https://redux-toolkit.js.org/api/createAsyncThunk) a été conçue pour abstraire ce modèle typique. Cependant, les utilisateurs doivent toujours écrire des quantités significatives de logique de réducteur pour gérer l'état de chargement et les données mises en cache.

Au cours de ces quelques dernières années, la communauté React s'est rendu compte que la **"récupération et la mise en cache des données" est vraiment un ensemble de problématiques différentes de la "gestion de l'état"**. Bien que vous puissiez utiliser une bibliothèque de gestion d'état comme Redux pour mettre en cache des données, les cas d'utilisation sont suffisamment différents pour qu'il vaille la peine d'utiliser des outils spécialement conçus pour le cas d'utilisation de la collecte de données.

RTK Query s'inspire d'autres outils qui ont été les premiers à proposer des solutions pour la collecte de données, comme Apollo Client, React Query, Urql et SWR, mais ajoute une approche unique à la conception de son API :

- La logique de récupération et de mise en cache des données est construite sur les API `createSlice` et `createAsyncThunk` de Redux Toolkit.
- Parce que Redux Toolkit ne prévoit pas d'UI particulière, les fonctionnalités de RTK Query peuvent être utilisées avec n'importe quelle couche d'interface utilisateur.
- Les points de terminaison de l'API sont définis à l'avance, y compris la manière de générer des paramètres de requête à partir d'arguments et de transformer les réponses pour la mise en cache.
- RTK Query peut également générer des hooks React qui encapsulent l'ensemble du processus de récupération des données, fournissent les champs `data` et `isLoading` au composants, et gérer la durée de vie des données mises en cache au fur et à mesure que les composants sont montés et démontés.
- RTK Query propose des options de "cycle de vie de l'entrée du cache" qui permettent des cas d'utilisation tels que la diffusion des mises à jour du cache via des messages websocket après avoir récupéré les données initiales.
- Nous disposons d'exemples préliminaires de génération de code de tranches d'API à partir de schémas OpenAPI et GraphQL.
- Enfin, RTK Query est entièrement écrit en TypeScript, et est conçu pour fournir une excellente expérience d'utilisation de TS.

## Qu'est ce qui a été inclue ?

### Les APIs

RTK Query est inclus dans l'installation du paquet de base de Redux Toolkit. Il est disponible via l'un des deux points d'entrée ci-dessous :

```js
import { createApi } from '@reduxjs/toolkit/query'

/* React-specific entry point that automatically generates
   hooks corresponding to the defined endpoints */
import { createApi } from '@reduxjs/toolkit/query/react'
```

RTK Query comprend les API suivantes :

- [`createApi()`](https://redux-toolkit.js.org/rtk-query/api/createApi) : Le cœur de la fonctionnalité de RTK Query. Elle vous permet de définir un ensemble de "endpoints" qui décrivent comment récupérer des données à partir d'APIs backend et d'autres sources asynchrones, y compris la configuration de la manière de récupérer et de transformer ces données. Dans la plupart des cas, vous ne devriez l'utiliser qu'une seule fois par application, avec comme règle générale "une tranche d'API par URL de base".
- [`fetchBaseQuery()`](https://redux-toolkit.js.org/rtk-query/api/fetchBaseQuery) : Est un petit wrapper (enveloppe) au toure de [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) qui vise à simplifier les requêtes. Il s'agit de la `baseQuery` recommandée à utiliser dans `createApi` pour la majorité des utilisateurs.
- [`<ApiProvider />`](https://redux-toolkit.js.org/rtk-query/api/ApiProvider) : Peut-être utilisé comme `Provider` si vous **n'avez pas déjà un store Redux**.
- [`setupListeners()`](https://redux-toolkit.js.org/rtk-query/api/setupListeners) : Un utilitaire utilisé pour activer les comportements refetchOnMount et refetchOnReconnect.

### Taille du bundle

RTK Query ajoute une quantité fixe de taille de bundle à votre application, mais cela se produit uniquement une fois. Étant donné que RTK Query repose sur Redux Toolkit et React-Redux, la taille ajoutée varie en fonction de l'utilisation préalable de ces bibliothèques dans votre application. Les tailles estimées du bundle, minimales et compressées (min+gzip), sont les suivantes :

- Si vous utilisez déjà RTK : environ 9 Ko pour RTK Query et environ 2 Ko pour les hooks.
- Si vous n'utilisez pas encore RTK :
  - Sans React : 17 Ko pour RTK + dépendances + RTK Query.
  - Avec React : 19 Ko + React-Redux, qui est une dépendance externe.

L'ajout de définitions d'endpoints supplémentaires ne devrait augmenter la taille que selon le code réel à l'intérieur de ces définitions d'`endpoints`, ce qui sera généralement seulement de quelques octets.

La fonctionnalité incluse dans RTK Query compense rapidement la taille ajoutée au bundle, et l'élimination de la logique de récupération de données écrite à la main devrait représenter une amélioration nette en termes de taille pour la plupart des applications significatives.

## Usage basique

### Créer un API Slice

RTK Query est inclus dans l'installation du paquetage principal de Redux Toolkit. Il est disponible via l'un des deux points d'entrée ci-dessous :

```js
import { createApi } from '@reduxjs/toolkit/query'

/* React-specific entry point that automatically generates
   hooks corresponding to the defined endpoints */
import { createApi } from '@reduxjs/toolkit/query/react'
```

Pour une utilisation typique avec React, commencez par importer `createApi` et définir une "tranche d'API" qui répertorie l'URL de base du serveur et les points d'extrémité avec lesquels nous voulons interagir :

```js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import type { Pokemon } from './types'

// Define a service using a base URL and expected endpoints
export const pokemonApi = createApi({
  reducerPath: 'pokemonApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://pokeapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getPokemonByName: builder.query<Pokemon, string>({
      query: (name) => `pokemon/${name}`,
    }),
  }),
})

// Export hooks for usage in functional components, which are
// auto-generated based on the defined endpoints
export const { useGetPokemonByNameQuery } = pokemonApi
```

### Configurer le Store

La "tranche d'API" contient également un réducteur de tranche Redux généré automatiquement et un middleware personnalisé qui gère la durée de vie des abonnements. Les deux doivent être ajoutés au magasin Redux :

```js
import { configureStore } from '@reduxjs/toolkit'
// Or from '@reduxjs/toolkit/query/react'
import { setupListeners } from '@reduxjs/toolkit/query'
import { pokemonApi } from './services/pokemon'

export const store = configureStore({
  reducer: {
    // Add the generated reducer as a specific top-level slice
    [pokemonApi.reducerPath]: pokemonApi.reducer,
  },
  // Adding the api middleware enables caching, invalidation, polling,
  // and other useful features of `rtk-query`.
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(pokemonApi.middleware),
})

// optional, but required for refetchOnFocus/refetchOnReconnect behaviors
// see `setupListeners` docs - takes an optional callback as the 2nd arg for customization
setupListeners(store.dispatch)
```

### Utilisation des Hooks dans les composants


Enfin, importez les hooks React générés automatiquement à partir de la tranche d'API dans le fichier de votre composant, et appelez ces hooks dans votre composant avec les paramètres nécessaires. RTK Query récupérera automatiquement les données lors du montage, les récupérera lorsque les paramètres changent, fournira les valeurs `{data, isFetching}` dans le résultat, et réaffichera le composant à mesure que ces valeurs changent :

```js
import * as React from 'react'
import { useGetPokemonByNameQuery } from './services/pokemon'

export default function App() {
  // Using a query hook automatically fetches data and returns query values
  const { data, error, isLoading } = useGetPokemonByNameQuery('bulbasaur')
  // Individual hooks are also accessible under the generated endpoints:
  // const { data, error, isLoading } = pokemonApi.endpoints.getPokemonByName.useQuery('bulbasaur')

  // render UI based on data and loading state
}
```

## Informations complémentaires[](https://redux-toolkit.js.org/rtk-query/overview#further-information "Lien direct vers la section")

Consultez le [**tutoriel RTK Query Quick Start**](https://redux-toolkit.js.org/tutorials/rtk-query/) pour des exemples sur la façon d'ajouter RTK Query à un projet utilisant Redux Toolkit, de configurer une "tranche d'API" avec des définitions d'endpoints, et sur l'utilisation des hooks React générés automatiquement dans vos composants.

La section [**Guide d'utilisation de RTK Query**](https://redux-toolkit.js.org/rtk-query/usage/queries) contient des informations sur des sujets tels que [la récupération de données](https://redux-toolkit.js.org/rtk-query/usage/queries), [l'utilisation des mutations pour envoyer des mises à jour au serveur](https://redux-toolkit.js.org/rtk-query/usage/mutations), [la diffusion de mises à jour de la cache](https://redux-toolkit.js.org/rtk-query/usage/streaming-updates), et bien plus encore.

La [**page des exemples**](https://redux-toolkit.js.org/rtk-query/usage/examples) propose des CodeSandboxes exécutables qui démontrent des sujets tels que [la réalisation de requêtes avec GraphQL](https://redux-toolkit.js.org/rtk-query/usage/examples#react-with-graphql), [l'authentification](https://redux-toolkit.js.org/rtk-query/usage/examples#authentication), et même [l'utilisation de RTK Query avec d'autres bibliothèques d'interface utilisateur comme Svelte](https://redux-toolkit.js.org/rtk-query/usage/examples#svelte).