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

    // Affiche le formulaire de modification avec les informations existantes
    [HttpGet("edit/{id}")]
    public IActionResult Edit(int id)
    {
        var utilisateur = _context.Utilisateurs.Find(id);
        if (utilisateur == null)
        {
            return NotFound();
        }
        return View(utilisateur);
    }

    // Met à jour un utilisateur en base de données
    [HttpPost("edit/{id}")]
    public IActionResult Edit(int id, Utilisateur utilisateur)
    {
        if (!ModelState.IsValid)
        {
            return View(utilisateur);
        }

        var existingUser = _context.Utilisateurs.Find(id);
        if (existingUser == null)
        {
            return NotFound();
        }

        existingUser.Nom = utilisateur.Nom;
        existingUser.Prenom = utilisateur.Prenom;
        existingUser.Email = utilisateur.Email;
        existingUser.Adresse = utilisateur.Adresse;
        existingUser.Role = utilisateur.Role;

        if (!string.IsNullOrEmpty(utilisateur.MotDePasse))
        {
            existingUser.MotDePasse = BCrypt.Net.BCrypt.HashPassword(utilisateur.MotDePasse);
        }

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
