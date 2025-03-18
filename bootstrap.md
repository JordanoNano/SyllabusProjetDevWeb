# Tutoriel : Intégrer et utiliser Bootstrap dans ASP.NET Core MVC

## Objectif
Ce tutoriel vous guidera pas à pas pour **intégrer Bootstrap** dans un projet **ASP.NET Core MVC** et l’utiliser efficacement pour styliser vos vues.

---

## Qu'est-ce que Bootstrap ?
**Bootstrap** est un **framework front-end** qui facilite la conception d'interfaces web modernes et responsives.

### **Caractéristiques principales :**
- Fournit des **composants réutilisables** (boutons, formulaires, cartes, alertes...)
- Utilise un **système de grille** pour structurer les mises en page.
- Intègre des **styles et animations CSS prédéfinis**.
- Compatible avec tous les navigateurs et écrans (responsive).

### **Pourquoi utiliser Bootstrap ?**
✔ Gain de temps grâce aux styles et composants prêts à l'emploi.  
✔ Compatible avec HTML, CSS, et JavaScript sans dépendances lourdes.  
✔ Permet de personnaliser facilement le design d'une application.  

---

## Ajouter Bootstrap à l’application

### **Méthode 1 : Via CDN **
Ajoutez ces lignes dans le fichier **`Views/Shared/_Layout.cshtml`**, dans la balise `<head>` :

```html
<!-- Bootstrap CSS -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">

<!-- Bootstrap JS (si nécessaire) -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
```

 **Avantages** :
- Facile et rapide.
- Pas besoin d’installation locale.
- Toujours à jour.

### **Méthode 2 : Installer Bootstrap via NuGet**
Si vous souhaitez utiliser Bootstrap **en local**, installez-le via **NuGet** :

```sh
dotnet add package bootstrap
```

Puis, dans **`wwwroot/css/site.css`**, ajoutez :

```css
@import url('/lib/bootstrap/dist/css/bootstrap.min.css');
```

Dans **`_Layout.cshtml`**, ajoutez ensuite :

```html
<link rel="stylesheet" href="~/css/bootstrap.min.css" asp-append-version="true">
```

 **Avantages** :
- Fonctionne sans connexion internet.
- Contrôle total sur la version utilisée.

---

Cette partie débouchera sur un laboratoire de plusieurs heures où vous aurez l'occasion de naviguer sur [Boostrap](https://getbootstrap.com/) et mettre en pratique ce Framework.
