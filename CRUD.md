# Documentation : CRUD sur `AdminUtilisateurController.cs` et ses Vues

## Introduction
Cette page explique en détail le fonctionnement du **contrôleur `AdminUtilisateurController.cs`** ainsi que des **vues associées**.  
Ce contrôleur permet :
- **D'afficher la liste des utilisateurs enregistrés**.
- **De créer un nouvel utilisateur en base de données avec un mot de passe haché**.
- **De modifier un utilisateur existant**.
- **De supprimer un utilisateur existant**.

---

## `AdminUtilisateurController.cs` : Gestion des utilisateurs

### Définition du contrôleur
Le contrôleur `AdminUtilisateurController` est responsable de la gestion des utilisateurs via l'interface d’administration.  
Il est accessible via l’URL `/admin/utilisateurs` et contient les méthodes suivantes :
- **`List`** : Affiche tous les utilisateurs enregistrés.
- **`Create (GET)`** : Affiche un formulaire de création d’utilisateur.
- **`Create (POST)`** : Traite les données soumises via le formulaire et enregistre l’utilisateur.
- **`Edit (GET)`** : Affiche un formulaire pré-rempli pour modifier un utilisateur existant.
- **`Edit (POST)`** : Met à jour les informations de l’utilisateur en base de données.
- **`Delete`** : Supprime un utilisateur.

### Code du contrôleur

```csharp
using Microsoft.AspNetCore.Mvc;
using MonApplication.Data;
using MonApplication.Models;
using BCrypt.Net;

[Route("admin/utilisateurs")]
public class AdminUtilisateurController : Controller
{
    private readonly ECommerceDbContext _context;

    // Constructeur qui injecte le contexte de base de données
    public AdminUtilisateurController(ECommerceDbContext context)
    {
        _context = context;
    }

    // Affiche la liste des utilisateurs
    [HttpGet("list")]
    public IActionResult List()
    {
        var users = _context.Utilisateurs.ToList();
        return View("AdminUtilisateur", users);
    }

    // Affiche un formulaire vide avec des valeurs par défaut pour créer un utilisateur
    [HttpGet("create")]
    public IActionResult Create()
    {
        return View(new Utilisateur
        {
            Nom = string.Empty,
            Prenom = string.Empty,
            Email = string.Empty,
            MotDePasse = string.Empty,
            Adresse = string.Empty,
            Role = "Client"
        });
    }

    // Traite l'ajout d'un utilisateur
    [HttpPost("create")]
    public IActionResult Create(Utilisateur utilisateur)
    {
        if (!ModelState.IsValid)
        {
            return View(utilisateur);
        }

        // Hachage du mot de passe
        utilisateur.MotDePasse = BCrypt.Net.BCrypt.HashPassword(utilisateur.MotDePasse);

        // Ajout en base de données
        _context.Utilisateurs.Add(utilisateur);
        _context.SaveChanges();

        return RedirectToAction("List");
    }

    // Supprime un utilisateur
    [HttpPost("delete/{id}")]
    public IActionResult Delete(int id)
    {
        var utilisateur = _context.Utilisateurs.Find(id);
        if (utilisateur == null)
        {
            return NotFound();
        }

        _context.Utilisateurs.Remove(utilisateur);
        _context.SaveChanges();

        return RedirectToAction("List");
    }
}
```
`AdminUtilisateur.cshtml`

```csharp
@using MonApplication.Models
@model IEnumerable<MonApplication.Models.Utilisateur>

<h2>Liste des utilisateurs</h2>

<table border="1">
    <tr>
        <th>ID</th>
        <th>Nom</th>
        <th>Prénom</th>
        <th>Email</th>
        <th>Rôle</th>
        <th>Actions</th>
    </tr>
    @foreach (var user in Model)
    {
        <tr>
            <td>@user.Id</td>
            <td>@user.Nom</td>
            <td>@user.Prenom</td>
            <td>@user.Email</td>
            <td>@user.Role</td>
            <td>
                <a href="/admin/utilisateurs/edit/@user.Id">Modifier</a> |
                <form method="post" action="/admin/utilisateurs/delete/@user.Id" style="display:inline;">
                    <button type="submit" onclick="return confirm('Voulez-vous vraiment supprimer cet utilisateur ?');">Supprimer</button>
                </form>
            </td>
        </tr>
    }
</table>

<a href="/admin/utilisateurs/create">Ajouter un nouvel utilisateur</a>
```
`Create.cshtml`
```csharp
@model MonApplication.Models.Utilisateur

<h2>Créer un utilisateur</h2>

<form method="post" asp-action="create" asp-controller="AdminUtilisateur">
    <div>
        <label>Nom</label>
        <input type="text" asp-for="Nom" name="Nom" required>
        <span asp-validation-for="Nom" class="text-danger"></span>
    </div>

    <div>
        <label>Prénom</label>
        <input type="text" asp-for="Prenom" name="Prenom" required>
        <span asp-validation-for="Prenom" class="text-danger"></span>
    </div>

    <div>
        <label>Email</label>
        <input type="email" asp-for="Email" name="Email" required>
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <div>
        <label>Mot de passe</label>
        <input type="password" asp-for="MotDePasse" name="MotDePasse" required>
        <span asp-validation-for="MotDePasse" class="text-danger"></span>
    </div>

    <div>
        <label>Adresse</label>
        <input type="text" asp-for="Adresse" name="Adresse">
        <span asp-validation-for="Adresse" class="text-danger"></span>
    </div>

    <div>
        <label>Rôle</label>
        <select asp-for="Role" name="Role">
            <option value="Client">Client</option>
            <option value="Admin">Admin</option>
        </select>
        <span asp-validation-for="Role" class="text-danger"></span>
    </div>

    <button type="submit">Créer l'utilisateur</button>
</form>

<a href="/admin/utilisateurs/list">Retour à la liste</a>
```

# Mise à jour d'un utilisateur (UPDATE) - ASP.NET Core MVC  

## Introduction  

Cette section explique le **fonctionnement de la mise à jour d'un utilisateur** en utilisant :  
- Un **ViewModel (`UtilisateurViewModel`)** pour séparer les données de la vue et du modèle.  
- Une **gestion du mot de passe sécurisée** (si laissé vide, il n'est pas modifié).  

---

## **Création du ViewModel `UtilisateurViewModel`**  

Le ViewModel permet de **séparer les données de la vue et du modèle de base de données**.  
Cela évite de manipuler directement l'ID et le mot de passe dans la vue.

**Fichier : `Models/ViewModels/UtilisateurViewModel.cs`**  

```csharp
using System.ComponentModel.DataAnnotations;

namespace MonApplication.Models.ViewModels
{
    public class UtilisateurViewModel
    {
        [Required]
        public string Nom { get; set; }

        [Required]
        public string Prenom { get; set; }

        [Required, EmailAddress]
        public string Email { get; set; }

        public string? MotDePasse { get; set; } // Pas obligatoire

        public string Adresse { get; set; }

        [Required]
        public string Role { get; set; }
    }
}
```
```csharp
[HttpGet("edit/{id}")]
public IActionResult Edit(int id)
{
    var utilisateur = _context.Utilisateurs.Find(id);
    if (utilisateur == null)
    {
        return NotFound();
    }

    // Convertir en ViewModel
    var viewModel = new UtilisateurViewModel
    {
        Nom = utilisateur.Nom,
        Prenom = utilisateur.Prenom,
        Email = utilisateur.Email,
        Adresse = utilisateur.Adresse,
        Role = utilisateur.Role
    };

    return View(viewModel);
}

[HttpPost("edit/{id}")]
public IActionResult Edit(int id, UtilisateurViewModel utilisateurViewModel)
{
    Console.WriteLine($"🔹 Modification utilisateur ID={id}, Nouveau Nom={utilisateurViewModel.Nom}, Email={utilisateurViewModel.Email}");

    if (!ModelState.IsValid)
    {
        Console.WriteLine("⚠ ModelState invalide !");
        foreach (var error in ModelState)
        {
            Console.WriteLine($"🔸 {error.Key}: {string.Join(", ", error.Value.Errors.Select(e => e.ErrorMessage))}");
        }
        return View(utilisateurViewModel);
    }

    var existingUser = _context.Utilisateurs.Find(id);
    if (existingUser == null)
    {
        return NotFound();
    }

    Console.WriteLine("✅ Utilisateur trouvé, mise à jour en cours...");

    existingUser.Nom = utilisateurViewModel.Nom;
    existingUser.Prenom = utilisateurViewModel.Prenom;
    existingUser.Email = utilisateurViewModel.Email;
    existingUser.Adresse = utilisateurViewModel.Adresse;
    existingUser.Role = utilisateurViewModel.Role;

    // Vérifier si un mot de passe a été fourni
    if (!string.IsNullOrEmpty(utilisateurViewModel.MotDePasse))
    {
        existingUser.MotDePasse = BCrypt.Net.BCrypt.HashPassword(utilisateurViewModel.MotDePasse);
        Console.WriteLine("🔒 Mot de passe mis à jour !");
    }

    try
    {
        _context.SaveChanges();
        Console.WriteLine("✅ Utilisateur mis à jour avec succès !");
        return RedirectToAction("List");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"❌ Erreur lors de la mise à jour : {ex.Message}");
        return View(utilisateurViewModel);
    }
}

```
`Edit.cshtml`
```csharp
@model MonApplication.Models.ViewModels.UtilisateurViewModel

<h2>Modifier un utilisateur</h2>

<form method="post" asp-action="Edit">
    <div>
        <label>Nom</label>
        <input type="text" asp-for="Nom" name="Nom" value="@Model.Nom" required>
        <span asp-validation-for="Nom" class="text-danger"></span>
    </div>

    <div>
        <label>Prénom</label>
        <input type="text" asp-for="Prenom" name="Prenom" value="@Model.Prenom" required>
        <span asp-validation-for="Prenom" class="text-danger"></span>
    </div>

    <div>
        <label>Email</label>
        <input type="email" asp-for="Email" name="Email" value="@Model.Email" required>
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <div>
        <label>Mot de passe (laisser vide pour ne pas changer)</label>
        <input type="password" asp-for="MotDePasse" name="MotDePasse">
        <span asp-validation-for="MotDePasse" class="text-danger"></span>
    </div>

    <div>
        <label>Adresse</label>
        <input type="text" asp-for="Adresse" name="Adresse" value="@Model.Adresse">
        <span asp-validation-for="Adresse" class="text-danger"></span>
    </div>

    <div>
        <label>Rôle</label>
        <select asp-for="Role" name="Role">
            <option value="Client" selected="@(Model.Role == "Client")">Client</option>
            <option value="Admin" selected="@(Model.Role == "Admin")">Admin</option>
        </select>
        <span asp-validation-for="Role" class="text-danger"></span>
    </div>

    <button type="submit">Mettre à jour</button>
</form>
```
