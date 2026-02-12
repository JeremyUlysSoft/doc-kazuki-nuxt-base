# Guide Nuxt 4 - Démarrage rapide

## Architecture du projet

Nuxt 4 introduit une nouvelle structure avec le dossier **`app/`** qui centralise tout le code de l'application Vue :

```
mon-projet/
├── app/                 # 🆕 Dossier principal de l'application
│   ├── app.vue         # Point d'entrée principal (optionnel si tu as pages/)
│   ├── pages/          # Routes front-end
│   │   ├── index.vue   # Route: /
│   │   ├── documents.vue    # Route: /documents
│   │   └── documents/
│   │       └── [id].vue     # Route: /documents/:id
│   ├── layouts/        # Layouts pour structurer les pages
│   │   ├── default.vue # Layout par défaut
│   │   └── custom.vue  # Layout personnalisé
│   ├── components/     # Composants Vue réutilisables
│   ├── composables/    # Composables (logique réutilisable)
│   ├── middleware/     # Middleware de navigation
│   ├── plugins/        # Plugins Nuxt
│   └── utils/          # Fonctions utilitaires
├── server/             # 🔧 Code serveur (séparé de l'app)
│   ├── api/           # Routes API
│   │   ├── documents.get.ts      # GET /api/documents
│   │   └── documents/
│   │       └── [id].get.ts       # GET /api/documents/:id
│   ├── routes/        # Routes serveur personnalisées
│   ├── middleware/    # Middleware serveur
│   └── utils/         # Utilitaires serveur
├── public/            # Fichiers statiques (images, fonts, etc.)
├── .nuxt/             # ⚠️ Fichiers générés (ne pas toucher)
├── nuxt.config.ts     # Configuration Nuxt
└── package.json
```

**Points clés de Nuxt 4 :**
- **`app/`** : Tout le code front-end (Vue) est dans ce dossier
- **`server/`** : Tout le code back-end (API) est séparé
- **Meilleure séparation** : code client vs code serveur
- **Type-safety améliorée** : TypeScript fonctionne mieux avec cette structure
- Les fichiers dans `app/pages/` créent automatiquement des routes
- Les fichiers dans `server/api/` créent automatiquement des endpoints API

---

## Le fichier app.vue - Point d'entrée principal

Le fichier `app/app.vue` est le composant racine de ton application. C'est par là que tout passe !

### Cas 1 : Projet sans pages (landing page simple)

Si tu ne crées **pas** de dossier `app/pages/`, Nuxt affiche par défaut `<NuxtWelcome />` :

**Fichier :** `app/app.vue` (celui créé par défaut)

```vue
<template>
  <div>
    <NuxtWelcome />
  </div>
</template>
```

👉 `<NuxtWelcome />` est juste un écran de bienvenue. **Tu dois le supprimer ou remplacer** par ton contenu !

---

### Cas 2 : Projet avec routing (le plus courant)

Dès que tu crées le dossier `app/pages/`, tu **dois** utiliser `<NuxtPage />` dans `app.vue` :

**Fichier :** `app/app.vue`

```vue
<template>
  <div>
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </div>
</template>
```

**Explications :**
- `<NuxtPage />` : Affiche la page correspondant à la route actuelle (contenu de `app/pages/`)
- `<NuxtLayout>` : Entoure la page avec un layout (header, footer, menu, etc.)

---

### Variante : Sans layouts

Si tu n'as **pas besoin** de layouts (pas de structure commune entre tes pages), tu peux simplifier :

```vue
<template>
  <div>
    <NuxtPage />
  </div>
</template>
```

Mais c'est rare ! En général, on veut un header/footer commun, donc on garde `<NuxtLayout>`.

---

### Variante : Structure globale dans app.vue

Tu peux aussi mettre des éléments globaux directement dans `app.vue` :

```vue
<template>
  <div>
    <header>
      <h1>Mon site</h1>
      <nav>
        <NuxtLink to="/">Accueil</NuxtLink>
        <NuxtLink to="/documents">Documents</NuxtLink>
      </nav>
    </header>

    <main>
      <NuxtLayout>
        <NuxtPage />
      </NuxtLayout>
    </main>

    <footer>
      <p>© 2025 Mon site</p>
    </footer>
  </div>
</template>
```

⚠️ **Attention** : Tout ce que tu mets dans `app.vue` est **global** (CSS, JS) et inclus dans toutes les pages.

---

## Les Layouts - Structure commune des pages

Les layouts permettent d'avoir une structure commune (header, footer, sidebar) pour plusieurs pages.

### Layout par défaut

**Fichier :** `app/layouts/default.vue`

```vue
<template>
  <div class="layout">
    <header class="header">
      <nav>
        <NuxtLink to="/">Accueil</NuxtLink>
        <NuxtLink to="/documents">Documents</NuxtLink>
      </nav>
    </header>

    <main class="content">
      <!-- C'est ici que le contenu de la page s'affiche -->
      <slot />
    </main>

    <footer class="footer">
      <p>© 2025 Mon application</p>
    </footer>
  </div>
</template>

<style scoped>
.layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  background: #333;
  color: white;
  padding: 1rem;
}

.header nav {
  display: flex;
  gap: 1rem;
}

.header a {
  color: white;
  text-decoration: none;
}

.content {
  flex: 1;
  padding: 2rem;
}

.footer {
  background: #f5f5f5;
  padding: 1rem;
  text-align: center;
}
</style>
```

**Important :**
- Le `<slot />` est **obligatoire** : c'est là que le contenu de la page s'insère
- Le layout doit avoir **un seul élément racine** (ici `<div class="layout">`)

---

### Layout personnalisé (exemple : admin)

**Fichier :** `app/layouts/admin.vue`

```vue
<template>
  <div class="admin-layout">
    <aside class="sidebar">
      <h2>Admin</h2>
      <nav>
        <NuxtLink to="/admin">Dashboard</NuxtLink>
        <NuxtLink to="/admin/users">Users</NuxtLink>
      </nav>
    </aside>

    <div class="main-content">
      <slot />
    </div>
  </div>
</template>

<style scoped>
.admin-layout {
  display: flex;
}

.sidebar {
  width: 250px;
  background: #2c3e50;
  color: white;
  padding: 1rem;
}

.main-content {
  flex: 1;
  padding: 2rem;
}
</style>
```

### Utiliser un layout spécifique dans une page

**Fichier :** `app/pages/admin/index.vue`

```vue
<script setup lang="ts">
// Définir le layout pour cette page
definePageMeta({
  layout: 'admin'
})
</script>

<template>
  <div>
    <h1>Admin Dashboard</h1>
    <p>Bienvenue dans l'espace admin</p>
  </div>
</template>
```

---

### Pas de layout pour une page

Si tu veux désactiver le layout pour une page spécifique :

```vue
<script setup lang="ts">
definePageMeta({
  layout: false
})
</script>

<template>
  <div>
    <h1>Page sans layout</h1>
  </div>
</template>
```

---

## ✅ Checklist pour démarrer

1. **Supprimer ou remplacer** `<NuxtWelcome />` dans `app.vue`
2. Créer le dossier `app/pages/` pour activer le routing
3. Mettre `<NuxtLayout><NuxtPage /></NuxtLayout>` dans `app.vue`
4. Créer `app/layouts/default.vue` avec ton header/footer
5. Créer tes pages dans `app/pages/`

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

**Fichier :** `app/pages/documents.vue`

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

**Fichier :** `app/pages/documents/[id].vue`

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
| Créer une page | Fichier dans `app/pages/` |
| Créer un layout | Fichier dans `app/layouts/` |
| Créer une API | Fichier dans `server/api/` |
| GET tous | `server/api/ressource.get.ts` |
| GET un | `server/api/ressource/[id].get.ts` |
| Appeler API au chargement | `useFetch('/api/...')` |
| Appeler API sur action | `$fetch('/api/...')` |
| Point d'entrée de l'app | `app/app.vue` |
| Composant réutilisable | `app/components/` |

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
