# Guide Nuxt 4 - Démarrage rapide

## Architecture du projet

Nuxt 4 utilise une structure de dossiers spécifique :

```
mon-projet/
├── pages/                # Routes front-end (à la racine)
│   ├── index.vue        # Route: /
│   ├── documents.vue    # Route: /documents
│   └── documents/
│       └── [id].vue     # Route: /documents/:id
├── server/              # Code serveur
│   ├── api/            # Routes API
│   │   ├── documents.get.ts      # GET /api/documents
│   │   └── documents/
│   │       └── [id].get.ts       # GET /api/documents/:id
│   ├── routes/         # Routes serveur personnalisées
│   ├── middleware/     # Middleware serveur
│   └── utils/          # Utilitaires serveur
├── components/          # Composants Vue
├── composables/         # Composables réutilisables
├── public/             # Fichiers statiques
└── nuxt.config.ts      # Configuration Nuxt
```

**Points clés :**
- `pages/` : dossier à la racine pour le routing front-end
- `server/api/` : routes API accessibles via `/api/*`
- Les fichiers dans `pages/` créent automatiquement des routes
- Les fichiers dans `server/api/` créent automatiquement des endpoints API

---

## Routes API serveur

### Route simple sans paramètres : GET tous les documents

**Fichier :** `server/api/documents.get.ts`

```typescript
// GET /api/documents
export default defineEventHandler(async (event) => {
  // Simule une récupération de données
  // En vrai, tu ferais un appel DB ici
  const documents = [
    { id: 1, title: "Document 1", content: "Contenu du document 1" },
    { id: 2, title: "Document 2", content: "Contenu du document 2" },
    { id: 3, title: "Document 3", content: "Contenu du document 3" }
  ]
  
  return {
    success: true,
    data: documents
  }
})
```

**Points clés :**
- Le nom du fichier détermine la méthode HTTP : `.get.ts`, `.post.ts`, `.put.ts`, `.delete.ts`
- `defineEventHandler` est la fonction pour définir un handler d'événement
- Tu retournes directement l'objet JSON

---

### Route avec paramètre : GET un document par ID

**Fichier :** `server/api/documents/[id].get.ts`

```typescript
// GET /api/documents/:id
export default defineEventHandler(async (event) => {
  // Récupère le paramètre 'id' de l'URL
  const id = getRouterParam(event, 'id')
  
  // Simule une recherche en DB
  const documents = [
    { id: 1, title: "Document 1", content: "Contenu du document 1" },
    { id: 2, title: "Document 2", content: "Contenu du document 2" },
    { id: 3, title: "Document 3", content: "Contenu du document 3" }
  ]
  
  const document = documents.find(d => d.id === parseInt(id || '0'))
  
  if (!document) {
    // Erreur 404 si le document n'existe pas
    throw createError({
      statusCode: 404,
      statusMessage: 'Document non trouvé'
    })
  }
  
  return {
    success: true,
    data: document
  }
})
```

**Points clés :**
- Les crochets `[id]` dans le nom de fichier indiquent un paramètre dynamique
- Utilise `getRouterParam(event, 'nomParam')` pour récupérer le paramètre
- Tu peux utiliser `createError()` pour renvoyer des erreurs HTTP

---

## Appeler les routes API depuis les pages Vue

### Page pour lister tous les documents

**Fichier :** `pages/documents.vue`

```vue
<script setup lang="ts">
// useFetch : composable Nuxt pour fetcher des données
// Avantages : 
// - SSR-friendly (pas de double appel client/serveur)
// - Gestion automatique du loading et des erreurs
// - Données réactives
const { data, pending, error, refresh } = await useFetch('/api/documents')
</script>

<template>
  <div class="documents-page">
    <h1>Liste des documents</h1>
    
    <!-- Affiche un loader pendant le chargement -->
    <div v-if="pending" class="loading">
      Chargement...
    </div>
    
    <!-- Affiche une erreur si ça a planté -->
    <div v-else-if="error" class="error">
      Erreur : {{ error.message }}
    </div>
    
    <!-- Affiche les documents -->
    <div v-else-if="data?.success">
      <ul>
        <li v-for="doc in data.data" :key="doc.id">
          <NuxtLink :to="`/documents/${doc.id}`">
            {{ doc.title }}
          </NuxtLink>
        </li>
      </ul>
      
      <!-- Bouton pour rafraîchir les données -->
      <button @click="refresh">Rafraîchir</button>
    </div>
  </div>
</template>

<style scoped>
.documents-page {
  padding: 2rem;
}

.loading {
  color: #666;
  font-style: italic;
}

.error {
  color: red;
  padding: 1rem;
  background: #fee;
  border-radius: 4px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  margin: 1rem 0;
  padding: 1rem;
  background: #f5f5f5;
  border-radius: 4px;
}
</style>
```

---

### Page pour afficher un document spécifique

**Fichier :** `pages/documents/[id].vue`

```vue
<script setup lang="ts">
// Récupère les paramètres de la route
const route = useRoute()

// Fetch dynamique : l'URL se met à jour si l'id change
// La fonction fléchée () => permet de rendre l'URL réactive
const { data, pending, error } = await useFetch(() => `/api/documents/${route.params.id}`)
</script>

<template>
  <div class="document-page">
    <!-- Bouton retour -->
    <NuxtLink to="/documents" class="back-link">
      ← Retour à la liste
    </NuxtLink>
    
    <div v-if="pending" class="loading">
      Chargement...
    </div>
    
    <div v-else-if="error" class="error">
      <h2>Erreur</h2>
      <p>{{ error.message }}</p>
    </div>
    
    <div v-else-if="data?.success" class="document">
      <h1>{{ data.data.title }}</h1>
      <div class="content">
        {{ data.data.content }}
      </div>
      <div class="meta">
        <small>ID : {{ data.data.id }}</small>
      </div>
    </div>
  </div>
</template>

<style scoped>
.document-page {
  padding: 2rem;
  max-width: 800px;
  margin: 0 auto;
}

.back-link {
  display: inline-block;
  margin-bottom: 2rem;
  color: #0066cc;
  text-decoration: none;
}

.back-link:hover {
  text-decoration: underline;
}

.loading {
  color: #666;
  font-style: italic;
}

.error {
  color: red;
  padding: 1rem;
  background: #fee;
  border-radius: 4px;
}

.document {
  background: #fff;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.content {
  margin: 2rem 0;
  line-height: 1.6;
}

.meta {
  margin-top: 2rem;
  padding-top: 1rem;
  border-top: 1px solid #eee;
  color: #666;
}
</style>
```

---

## useFetch vs $fetch - Quand utiliser quoi ?

### `useFetch` - Pour les composants Vue (recommandé)

```vue
<script setup>
// ✅ BON : useFetch dans un composant
// - Pas de double fetch (serveur + client)
// - Données transférées via le payload Nuxt
// - Gestion automatique du state (pending, error, refresh)
const { data, pending, error } = await useFetch('/api/documents')
</script>
```

**Utilise `useFetch` quand :**
- Tu es dans un composant Vue
- Tu veux afficher des données au chargement de la page
- Tu veux le SSR (Server-Side Rendering)

---

### `$fetch` - Pour les actions utilisateur

```vue
<script setup>
// ✅ BON : $fetch pour une action (clic, submit, etc.)
async function deleteDocument(id: number) {
  try {
    await $fetch(`/api/documents/${id}`, {
      method: 'DELETE'
    })
    // Rafraîchir la liste après suppression
    await refresh()
  } catch (error) {
    console.error('Erreur suppression:', error)
  }
}
</script>

<template>
  <button @click="deleteDocument(doc.id)">Supprimer</button>
</template>
```

**Utilise `$fetch` quand :**
- C'est déclenché par une action utilisateur (clic, submit)
- Tu es dans un event handler
- Tu fais un POST, PUT, DELETE

---

## Résumé rapide

| Besoin | Solution |
|--------|----------|
| Créer une page | Fichier dans `pages/` |
| Créer une API | Fichier dans `server/api/` |
| GET tous | `server/api/ressource.get.ts` |
| GET un | `server/api/ressource/[id].get.ts` |
| Appeler API au chargement | `useFetch('/api/...')` |
| Appeler API sur action | `$fetch('/api/...')` |

---

## Commandes utiles

```bash
# Lancer le serveur de dev
npm run dev

# Builder pour la production
npm run build

# Prévisualiser le build de prod
npm run preview

# Générer un site statique
npm run generate
```

---

## Pour aller plus loin

- **Documentation Nuxt 4** : https://nuxt.com/docs/4.x
- **Routing** : https://nuxt.com/docs/4.x/directory-structure/app/pages
- **Server API** : https://nuxt.com/docs/4.x/directory-structure/server
- **Data Fetching** : https://nuxt.com/docs/4.x/getting-started/data-fetching
