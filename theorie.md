# Cours : Introduction à ASP.NET Core MVC

## 1. Qu'est-ce qu'un Framework ?
Un **framework** est un ensemble d'outils, de bibliothèques et de conventions qui facilitent le développement d'applications en imposant une structure prédéfinie.

### **Caractéristiques d'un framework :**
- Offre une architecture préétablie.
- Fournit des bibliothèques et des outils intégrés.
- Facilite la gestion des dépendances.
- Encourage les bonnes pratiques de développement.

**Exemples de frameworks populaires :**
- **ASP.NET Core MVC** (pour les applications web en .NET)
- **Django** (Python)
- **Spring Boot** (Java)
- **Laravel** (PHP)

Un framework permet de **gagner du temps**, d'éviter de "réinventer la roue" et d'avoir une **architecture propre et modulaire**.

---

## 2. Qu'est-ce que ASP.NET Core MVC ?
ASP.NET Core MVC est un framework de développement web basé sur **l'architecture MVC (Modèle - Vue - Contrôleur)**. Il permet de développer des applications web robustes et maintenables.

### **L'architecture MVC**
1. **Modèle (Model)** : Gère la logique métier et les données (ex: base de données, validation des entrées).
2. **Vue (View)** : Gère l'affichage des données à l'utilisateur (HTML, CSS, Razor).
3. **Contrôleur (Controller)** : Gère la logique de l'application et interagit avec le modèle et la vue.

---

## 3. Les Contrôleurs en ASP.NET Core
Un **contrôleur** est une classe qui gère les requêtes HTTP entrantes et retourne une réponse.

### **Qu'est-ce que `IActionResult` ?**
- `IActionResult` est une interface qui définit le type de réponse qu'un contrôleur peut retourner.
- Exemples de **retours possibles** :
  - `return View();` (Retourne une vue HTML)
  - `return RedirectToAction("Index");` (Redirige vers une autre action)
  - `return Json(new { message = "Hello" });` (Retourne un JSON)
  - `return NotFound();` (Retourne une erreur 404)

---

## 4. Les Modèles et la Base de Données
Un **modèle** représente une entité dans la base de données.

### **Entity Framework Core : Gestion de la base de données**
ASP.NET Core utilise **Entity Framework Core** pour interagir avec la base de données.

**Génération de la base de données** :
```sh
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

##  5. Les Vues et Razor
### **Qu'est-ce qu'une Vue ?**
Une **vue** est un fichier HTML qui affiche les données envoyées par le contrôleur.

### **Le moteur de vue Razor**
Razor permet d'insérer du C# dans le HTML pour générer du contenu dynamique.

---


