# Rapport de Débogage — `index.html`

**Date :** 2026-04-22  
**Fichier analysé :** `index.html`  
**Statut :** Aucune correction appliquée

## Résumé

7 problèmes détectés au total : 1 critique, 3 majeurs, 3 mineurs. Les problèmes touchent la structure sémantique HTML, des conflits de styles entre le `<style>` inline et `style.css`, des liens vers des fichiers inexistants, l'accessibilité, et des doublons de règles CSS.

---

## Problèmes Détectés

### 🔴 Problème 1 — `<main>` commenté : structure sémantique brisée

**Fichier :** `index.html` (lignes 286, 322)  
**Sévérité :** Critique  
**Type :** Sémantique HTML5 / Accessibilité

**Code problématique :**
```html
<!--<main class="entry-main">-->
<div class="ent-hero reveal">
  ...
</div>
<div class="entry-choices">
  ...
</div>
<!--</main>-->
```

**Explication :**
La balise `<main>` est commentée, remplacée par des `<div>` nus. Cela prive la page de son point de repère sémantique principal, ce qui nuit à l'accessibilité (les lecteurs d'écran ne peuvent pas identifier le contenu principal) et au SEO. De plus, la classe `.entry-main` définie dans le `<style>` (flex, padding) n'est appliquée nulle part puisque le `<div>` qui la contiendrait n'existe plus. La mise en page souhaitée (flex column, padding 4rem) est donc silencieusement perdue.

**Correction proposée :**
```html
<main class="entry-main">
  <div class="ent-hero reveal">
    ...
  </div>
  <div class="entry-choices">
    ...
  </div>
</main>
```

**Changement :** Décommenter `<main class="entry-main">` et `</main>` pour restaurer la sémantique et appliquer les styles définis.

---

### 🟠 Problème 2 — Liens vers des fichiers inexistants

**Fichier :** `index.html` (lignes 255, 256, 276, 277, 279)  
**Sévérité :** Majeure  
**Type :** Chemin de fichier cassé

**Code problématique :**
```html
<li><a href="blog/index.html">Blog</a></li>
<li><a href="contact.html" class="nav-cta">Contact</a></li>
...
<a href="blog/index.html">Blog</a>
<a href="contact.html">Contact</a>
```

**Explication :**
- `/blog/index.html` n'existe pas dans le projet (répertoire absent).
- `/contact.html` n'existe pas non plus.
- `/particuliers/index.html` existe mais est un fichier vide (1 ligne), ce qui produira une page blanche.

Ces liens mèneront à des erreurs 404 (en production) ou à des pages vides (en local).

**Correction proposée :**
```html
<li><a href="#" aria-disabled="true">Blog</a></li>
<li><a href="#" aria-disabled="true" class="nav-cta">Contact</a></li>
```

**Changement :** Neutraliser les liens cassés en attendant que les fichiers soient créés.

---

### 🟠 Problème 3 — Conflit et duplication de règles CSS : `.nav-mobile` et `.nav-burger`

**Fichiers :** `index.html` (lignes 192–235) et `assets/css/style.css` (lignes 415–463)  
**Sévérité :** Majeure  
**Type :** Conflit CSS / Override non intentionnel

**Explication :**
Le bloc `<style>` dans `index.html` redéfinit `.nav-mobile`, `.nav-burger`, `.nav-burger span` et `.nav-mobile a` avec des valeurs légèrement différentes de `style.css`.

| Propriété | `style.css` | `<style>` inline |
|---|---|---|
| `.nav-mobile` `background` | `var(--sable)` | `var(--blanc)` |
| `.nav-mobile` `gap` | `0` | `1.2rem` |
| `.nav-mobile a` `font-size` | `0.8rem` | `0.85rem` |
| `.nav-mobile a` `letter-spacing` | `0.1em` | `0.12em` |
| `.nav-mobile a` `padding` | `0.75rem 0` | `0.5rem 0` |
| Media query burger | `max-width: 900px` | `max-width: 600px` |

Le style inline écrase `style.css`. Le menu mobile aura un fond blanc au lieu de sable, des espacements différents. Le media query conflicte aussi : `style.css` active le burger à 900px, le `<style>` inline le ferait à 600px seulement.

**Correction proposée :** Supprimer du `<style>` inline toutes les règles qui dupliquent `style.css`.

**Changement :** Supprimer les règles dupliquées du `<style>` inline pour éviter les conflits et unifier le comportement mobile.

---

### 🟡 Problème 4 — `.btn-choice:hover .choice-arrow` défini deux fois

**Fichier :** `index.html` (lignes 122–125 et 159–162)  
**Sévérité :** Mineure  
**Type :** Duplication de règle CSS

**Code problématique :**
```css
/* Première occurrence — ligne 122 */
.btn-choice:hover .choice-arrow {
  opacity: 1;
  transform: translateX(4px);
}

/* Deuxième occurrence — ligne 159 */
.btn-choice:hover .choice-arrow {
  opacity: 1;
  transform: translateX(4px);
}
```

**Correction proposée :** Supprimer la deuxième occurrence (lignes 159–162).

**Changement :** Suppression du doublon.

---

### 🟠 Problème 5 — Classes définies mais jamais appliquées dans `.ent-hero`

**Fichier :** `index.html` (lignes 288, 290, 297)  
**Sévérité :** Majeure  
**Type :** Interaction CSS non intentionnelle

**Code problématique :**
```html
<div class="ent-hero reveal">
  <p>Laboratoire de clarté digitale</p>
  <h1>Technologie<br>et <em>conscience.</em></h1>
  <p>Dans un monde qui nous incite...</p>
  <p>Vous êtes…</p>
</div>
```

**Explication :** Les classes `.entry-label`, `.entry-sub`, `.entry-question` sont définies dans le `<style>` mais ne sont appliquées à aucune balise. Les trois `<p>` reçoivent donc tous le même style générique de `style.css`.

**Correction proposée :**
```html
<p class="entry-label">Laboratoire de clarté digitale</p>
<p class="entry-sub">Dans un monde qui nous incite...</p>
<p class="entry-question">Vous êtes…</p>
```

**Changement :** Appliquer les classes prévues aux bons paragraphes.

---

### 🟡 Problème 6 — `<nav>` sans attribut `aria-label`

**Fichier :** `index.html` (ligne 246)  
**Sévérité :** Mineure  
**Type :** Accessibilité

**Correction proposée :**
```html
<nav aria-label="Navigation principale">
```

**Changement :** Ajout de `aria-label` sur la balise `<nav>`.

---

### 🟡 Problème 7 — Copyright figé à 2025

**Fichier :** `index.html` (ligne 332)  
**Sévérité :** Mineure  
**Type :** Bonne pratique / Maintenance

**Code problématique :**
```html
<p>© 2025 Faï</p>
```

**Correction proposée :**
```html
<p id="footer-year">© 2025 Faï</p>
```
```js
document.getElementById('footer-year').textContent =
  '© ' + new Date().getFullYear() + ' Faï';
```

**Changement :** Année du copyright mise à jour dynamiquement via JS.

---

## Récapitulatif des Modifications

| # | Fichier | Élément | Sévérité | Statut |
|---|---------|---------|----------|--------|
| 1 | `index.html` | Lignes 286, 322 — Décommenter `<main>` | Critique | ⏳ En attente |
| 2 | `index.html` | Lignes 255, 256, 276–279 — Liens cassés | Majeure | ⏳ En attente |
| 3 | `index.html` | Lignes 192–235 — Conflits CSS nav mobile | Majeure | ⏳ En attente |
| 4 | `index.html` | Lignes 159–162 — Doublon `.btn-choice:hover` | Mineure | ⏳ En attente |
| 5 | `index.html` | Lignes 288, 290, 297 — Classes non appliquées | Majeure | ⏳ En attente |
| 6 | `index.html` | Ligne 246 — `aria-label` manquant sur `<nav>` | Mineure | ⏳ En attente |
| 7 | `index.html` | Ligne 332 — Copyright dynamique | Mineure | ⏳ En attente |
