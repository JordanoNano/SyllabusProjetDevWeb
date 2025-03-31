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

### Code du Models
```csharp
// Importation de l'espace de noms pour les attributs de validation comme [Required], [StringLength]...
using System.ComponentModel.DataAnnotations;

// Importation de l'espace de noms pour les annotations de db comme [Key], [DatabaseGenerated]...
using System.ComponentModel.DataAnnotations.Schema;

// Déclaration du namespace qui regroupe les classes de type Models
namespace MonApplication.Models 
{
    // Définition de la classe Utilisateur — représente un utilisateur dans la DB
    public class Utilisateur
    {   
        // Indique à Entity Framework que c'est la clé primaire de la table
        [Key] 
        // Spécifie que la valeur est auto-incrémentée 
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)] 
        // Déclaration de la propriété id de l'utilisateur
        public int IdUtilisateur { get; set; }
        // Liaison avec la table Role
        // Validation : ce champ doit être renseigné (sinon erreur affichée)
        [Required(ErrorMessage = "Le Role est obligatoire.")] 
        // Clé étrangère qui peut être null vers la table Role (Apparement c'est qqchose propre au binding ASP NET CORE.... à creuser)
        // Binding : ASP.NET Core fait correspondre les champs du formulaire HTML (grâce au name="...") 
        // avec les propriétés du modèle C# (ex: name="Nom" =  Utilisateur.Nom)


        // ----- RELATION AVEC ROLE -----
        public int? IdRole { get; set; } 
        // Propriété de navigation vers l'objet Role (Accèder directement à l'objet rôle pour par exemple, récupérer l'intitulé du rôle)
        public Role? Role { get; set; } 
        // Liaison avec la table Localite
        // Validation : champ obligatoire
        [Required(ErrorMessage = "Le Localite est obligatoire.")] 
        // Clé étrangère qui peut-être null vers la table Localite (Apparement c'est qqchose propre au binding ASP NET CORE.... à creuser)


        // ----- RELATION AVEC LOCALITE -----
        public int? IdLocalite { get; set; } 
        // Propriété de navigation vers l'objet Localité (Accèder directement à l'objet localité pour par exemple, récupérer le nom de la localité)
        public Localite? Localite { get; set; } 
        // Informations personnelles de l'utilisateur
        // Champ requis pour le nom
        [Required(ErrorMessage = "Le nom est obligatoire.")] 
        // Limite de longueur
        [StringLength(50, ErrorMessage = "Le nom ne peut pas dépasser 50 caractères.")] 
        // Le mot-clé required impose l'initialisation à l’instanciation


        // ----- INFOS PERSOS SUR UN UTILISATEUR -----
        public required string Nom { get; set; } 
        // Champ requis pour le prénom
        [Required(ErrorMessage = "Le prénom est obligatoire.")] 
        // Limite de longueur
        [StringLength(50, ErrorMessage = "Le prénom ne peut pas dépasser 50 caractères.")] 
        public required string Prenom { get; set; }
        // Champ requis
        [Required(ErrorMessage = "L'adresse est obligatoire.")] 
        // Limite de longueur
        [StringLength(255, ErrorMessage = "L'adresse ne peut pas dépasser 255 caractères.")] 
        public required string Adresse { get; set; }
        // Champ requis
        [Required(ErrorMessage = "L'email est obligatoire.")] 
        // Vérifie que le champ est bien une adresse mail valide (jordano@gmail.com)
        [EmailAddress(ErrorMessage = "L'email n'est pas valide.")] 
        public required string Email { get; set; }
        // Champ requis pour le mot de passe
        [Required(ErrorMessage = "Le mot de passe est obligatoire.")] 
        public required string MotDePasse { get; set; } 
    }
}

```

### Code du contrôleur

```csharp
// Importation des outils nécessaires au MVC (Models, Controllers, Views)
using Microsoft.AspNetCore.Mvc;

// Importation pour utiliser SelectList et SelectListItem (dropdownList pour les vues)
using Microsoft.AspNetCore.Mvc.Rendering;

// Importation Entity Framework Core (requêtes, Include, SaveChanges,...)
using Microsoft.EntityFrameworkCore;

// Importation du DbContext de l'application
using MonApplication.Data;

// Importation des modèles
using MonApplication.Models;
using MonApplication.ViewModels;

namespace MonApplication.Controllers
{
    // Route personnalisée du contrôleur : toutes les URLs commenceront par /admin/utilisateurs !
    [Route("admin/utilisateurs")]
    public class AdminUtilisateurController : Controller
    {
        // Dépendance vers la base de données (via DbContext)
        private readonly ECommerceDbContext _context;

        // Logger pour afficher des infos ou erreurs dans la console
        private readonly ILogger<AdminUtilisateurController> _logger;

        // Constructeur avec injection de dépendances
        // Injection de dépendances : Recevoir un objet prêt à l'emploi qui vient "d'ailleurs",
        // un service externe, des outils fournit par le framwork..
        public AdminUtilisateurController(ECommerceDbContext context, ILogger<AdminUtilisateurController> logger)
        {
            _context = context;
            _logger = logger;
        }

        // ---------------------------
        // GET: Affiche le formulaire de création
        // ---------------------------
        [HttpGet("create")]
        public IActionResult Create()
        {
            // ATTENTION : Le ViewBag c'est un peu la méthode wish pour les ViewModels !
            // Vous savez que ça existe et il y aura peut-être des cas où ce sera adapté mais sinon on passe au ViewModels pour la suite
            // Remplit la liste des rôles pour le select dropdown
            ViewBag.Roles = _context.Role
                .Select(r => new SelectListItem
                {
                    Value = r.IdRole.ToString(),
                    Text = r.IntituleRole
                }).ToList();

            // Remplit la liste des localités pour le select dropdown
            ViewBag.Localites = _context.Localite
                .Select(l => new SelectListItem
                {
                    Value = l.IdLocalite.ToString(),
                    Text = l.NomLocalite
                }).ToList();

            // On renvoie la vue avec un objet Utilisateur vide
            return View(new Utilisateur
            {
                IdRole = null,
                IdLocalite = null,
                Nom = string.Empty,
                Prenom = string.Empty,
                Adresse = string.Empty,
                Email = string.Empty,
                MotDePasse = string.Empty
            });
        }

        // ---------------------------
        // POST: Traite le formulaire de création
        // ---------------------------
        [HttpPost("create")]
        [ValidateAntiForgeryToken]
        public IActionResult Create([Bind("Nom,Prenom,Adresse,Email,MotDePasse,IdRole,IdLocalite")] Utilisateur utilisateur)
        {
            // --- LOGGING : Début de l'action POST ---
            _logger.LogInformation(">>> Début POST Create");
            _logger.LogInformation("Modèle reçu : Nom={Nom}, Prénom={Prenom}, Email={Email}, IdRole={IdRole}, IdLocalite={IdLocalite}",
                utilisateur.Nom, utilisateur.Prenom, utilisateur.Email, utilisateur.IdRole, utilisateur.IdLocalite);

            // Vérifie si le rôle fourni existe en db
            if (!_context.Role.Any(r => r.IdRole == utilisateur.IdRole))
            {
                _logger.LogWarning("Le rôle ID {IdRole} est invalide.", utilisateur.IdRole);
                ModelState.AddModelError("IdRole", "Le rôle sélectionné est invalide.");
            }
            else
            {
                _logger.LogInformation("Le rôle ID {IdRole} est valide.", utilisateur.IdRole);
            }

            // Vérifie si la localité fournie existe
            if (!_context.Localite.Any(l => l.IdLocalite == utilisateur.IdLocalite))
            {
                _logger.LogWarning("La localité ID {IdLocalite} est invalide.", utilisateur.IdLocalite);
                ModelState.AddModelError("IdLocalite", "La localité sélectionnée est invalide.");
            }
            else
            {
                _logger.LogInformation("La localité ID {IdLocalite} est valide.", utilisateur.IdLocalite);
            }

            // Si le modèle est invalide (erreurs de validation), on log tout et on retourne à la vue
            // Ici faire attention avec les validations ASP NET, ça ne semble pas fonctionner à 100%
            if (!ModelState.IsValid)
            {
                _logger.LogWarning("Modèle invalide, on retourne à la vue.");
                foreach (var key in ModelState.Keys)
                {
                    var state = ModelState[key];
                    foreach (var error in state.Errors)
                    {
                        _logger.LogWarning("Erreur champ '{Key}' : {Message}", key, error.ErrorMessage);
                    }
                }

                // On recharge les listes des dropdowns pour la vue
                ViewBag.Roles = _context.Role
                    .Select(r => new SelectListItem
                    {
                        Value = r.IdRole.ToString(),
                        Text = r.IntituleRole
                    }).ToList();

                ViewBag.Localites = _context.Localite
                    .Select(l => new SelectListItem
                    {
                        Value = l.IdLocalite.ToString(),
                        Text = l.NomLocalite
                    }).ToList();

                // On retourne le formulaire avec les erreurs affichées
                return View(utilisateur);
            }

            // Si tout est valide :
            _logger.LogInformation("Modèle valide, on enregistre l'utilisateur.");

            // Hash du mot de passe avant sauvegarde (très important pour la sécurité, sinon viré)
            utilisateur.MotDePasse = BCrypt.Net.BCrypt.HashPassword(utilisateur.MotDePasse);

            // Ajout de l'utilisateur en db
            _context.Utilisateur.Add(utilisateur);
            _context.SaveChanges();

            _logger.LogInformation("Utilisateur créé avec succès : {Email}", utilisateur.Email);

            // Redirection vers la liste
            return RedirectToAction("List");
        }

        // ---------------------------
        // GET: Liste des utilisateurs
        // ---------------------------
        [HttpGet("list")]
        public IActionResult List()
        {
            // Récupère tous les utilisateurs avec leurs rôles et localités 
            // Include = chargement des relations
            var users = _context.Utilisateur
                .Include(u => u.Role)
                .Include(u => u.Localite)
                .ToList();

            // Envoie la liste à la vue
            // users qu'on appelle dans la vue
            return View("AdminUtilisateur", users);
        }

        // ---------------------------
        // POST: Suppression d’un utilisateur
        // ---------------------------
        [HttpPost("delete/{id}")]
        [ValidateAntiForgeryToken]
        public IActionResult Delete(int id)
        {
            _logger.LogInformation(">>> Tentative de suppression utilisateur ID : {Id}", id);

            // On tente de retrouver l'utilisateur à supprimer
            var utilisateur = _context.Utilisateur.Find(id);
            if (utilisateur == null)
            {
                _logger.LogWarning("Utilisateur introuvable : ID {Id}", id);
                 // Renvoie une 404 si l'utilisateur n'existe pas
                return NotFound();
            }

            // Suppression en db
            _context.Utilisateur.Remove(utilisateur);
            _context.SaveChanges();

            _logger.LogInformation("Utilisateur supprimé : {Email}", utilisateur.Email);
            // On revient à la liste après suppression
            return RedirectToAction("List"); 
        }

        // ---------------------------
        // GET: Modification d’un utilisateur
        // ---------------------------
       [HttpGet("edit/{id}")]
public IActionResult Edit(int id)
{
    _logger.LogInformation(">>> [GET] Edit appelé pour l'utilisateur ID : {Id}", id);

    // Étape 1 : Récupération de l'utilisateur dans la db
    var utilisateur = _context.Utilisateur.FirstOrDefault(u => u.IdUtilisateur == id);

    if (utilisateur == null)
    {
        _logger.LogWarning("Aucun utilisateur trouvé avec l'ID : {Id}", id);
        return NotFound(); // Affiche une 404 si l'utilisateur n'existe pas
    }

    // Étape 2 : Log des données récupérées
    _logger.LogInformation("Utilisateur récupéré : {Nom} {Prenom}, Email : {Email}, IdRole : {IdRole}, IdLocalite : {IdLocalite}",
        utilisateur.Nom, utilisateur.Prenom, utilisateur.Email, utilisateur.IdRole, utilisateur.IdLocalite);

    // Étape 3 : Création du ViewModel avec les données de l'utilisateur
    var vm = new UtilisateurEditViewModel
    {
        IdUtilisateur = utilisateur.IdUtilisateur,
        Nom = utilisateur.Nom,
        Prenom = utilisateur.Prenom,
        Adresse = utilisateur.Adresse,
        Email = utilisateur.Email,
        IdRole = utilisateur.IdRole,
        IdLocalite = utilisateur.IdLocalite,

        // Remplissage des listes déroulantes
        Roles = _context.Role.Select(r => new SelectListItem
        {
            Value = r.IdRole.ToString(),
            Text = r.IntituleRole
        }).ToList(),

        Localites = _context.Localite.Select(l => new SelectListItem
        {
            Value = l.IdLocalite.ToString(),
            Text = l.NomLocalite
        }).ToList()
    };

    // Étape 4 : Log pour valider le contenu du ViewModel
    _logger.LogInformation("ViewModel prêt à être envoyé à la vue. Nom : {Nom}, Email : {Email}", vm.Nom, vm.Email);

    // Étape 5 : Affichage de la vue
    return View(vm);
}


[HttpPost("edit/{id}")]
[ValidateAntiForgeryToken]
public IActionResult Edit(int id, UtilisateurEditViewModel vm)
{
    _logger.LogInformation(">>> POST Edit pour ID {id}", id);

    if (!ModelState.IsValid)
    {
        _logger.LogWarning("Modèle invalide.");
        vm.Roles = _context.Role.Select(r => new SelectListItem
        {
            Value = r.IdRole.ToString(),
            Text = r.IntituleRole
        }).ToList();

        vm.Localites = _context.Localite.Select(l => new SelectListItem
        {
            Value = l.IdLocalite.ToString(),
            Text = l.NomLocalite
        }).ToList();

        return View(vm);
    }

    var utilisateur = _context.Utilisateur.Find(id);
    if (utilisateur == null) return NotFound();

    utilisateur.Nom = vm.Nom;
    utilisateur.Prenom = vm.Prenom;
    utilisateur.Adresse = vm.Adresse;
    utilisateur.Email = vm.Email;
    utilisateur.IdRole = vm.IdRole;
    utilisateur.IdLocalite = vm.IdLocalite;

    if (!string.IsNullOrWhiteSpace(vm.MotDePasse))
    {
        utilisateur.MotDePasse = BCrypt.Net.BCrypt.HashPassword(vm.MotDePasse);
    }

    _context.SaveChanges();
    _logger.LogInformation("Utilisateur mis à jour avec succès.");

    return RedirectToAction("List");
        }

    }
}

```
### Code de la vue qui affiche

```csharp
<!----------- Importation du namespace contenant le modèle Utilisateur ----------->
@using MonApplication.Models 

<!----------- Déclaration du modèle de la vue : ici, on affiche une liste d'objets Utilisateur ----------->
@model IEnumerable<MonApplication.Models.Utilisateur> 

@{
    // Titre de la page, utilisé dans le layout (_Layout.cshtml)
    ViewData["Title"] = "Liste des utilisateurs"; 
}

<!----- Titre affiché sur la page ----->
<h2>Liste des utilisateurs</h2> 

<!----- Tableau HTML dégueu :) ----->
<table border="1" cellpadding="5" cellspacing="0">
    <thead>
        <tr>
            <!----- En-têtes du tableau ----->
            <th>ID</th>
            <th>Rôle</th>
            <th>Localité</th>
            <th>Nom</th>
            <th>Prénom</th>
            <th>Adresse</th>
            <th>Email</th>
            <th>Actions</th>
        </tr>
    </thead>

    <tbody>
        <!----- Boucle sur chaque utilisateur de la liste, code dans le controller  ----->
        @foreach (var user in Model)
        {
            <tr>
                <!----- Affichage des données utilisateur ----->
                <td>@user.IdUtilisateur</td>
                <td>@user.Role?.IntituleRole</td> <!----- Se fait grâce à l'accès données par la la propriété de navigation vers Role ----->
                <td>@user.Localite?.NomLocalite</td> <!----- Se fait grâce à l'accès données par la propriété de navigation vers Localite ----->
                <td>@user.Nom</td>
                <td>@user.Prenom</td>
                <td>@user.Adresse</td>
                <td>@user.Email</td>
                <td>
                    <!----- ----------- Lien vers l'édition de l'utilisateur ----------- ----->
                    <a href="/admin/utilisateurs/edit/@user.IdUtilisateur">Modifier</a>

                    <!----- ----------- Formulaire pour supprimer l'utilisateur ----------- ----->
                    <form method="post" action="/admin/utilisateurs/delete/@user.IdUtilisateur" style="display:inline;">
                    <!----- Token anti-CSRF pour sécuriser le POST : c'est un peu avancé mais sympa à savoir,
                     ASP NET va ajouter un Token caché,dans le controller on add [ValidateAntiForgeryToken] qui attend le Token provenant du form
                      ----->
                        @Html.AntiForgeryToken() 

                        <button type="submit" onclick="return confirm('Voulez-vous vraiment supprimer cet utilisateur ?');">
                            Supprimer
                        </button>
                    </form>
                </td>
            </tr>
        }
    </tbody>
</table>

<!----- Lien pour accéder à la création d’un nouvel utilisateur ----->
<p>
    <a href="/admin/utilisateurs/create">Ajouter un nouvel utilisateur</a>
</p>

```
### Code de la vue qui crée
```csharp
<!----- Déclaration du modèle : cette vue attend un objet de type Utilisateur----->
@model MonApplication.Models.Utilisateur 

@{
    // Titre de la page (utilisé dans le layout)
    ViewData["Title"] = "Créer un utilisateur"; 
}

<h2>Créer un utilisateur</h2>

<!----- Formulaire pour la création ----->
<form method="post" asp-action="Create" asp-controller="AdminUtilisateur">
    @Html.AntiForgeryToken()

    <!-----  RÔLE  ----->
    <div>
        <label for="IdRole">Rôle</label> 
        <!-----Logiquement ASP NET CORE propose un truc natif pour ça : asp-for mais
         j'ai pas réussi à comprendre pourquoi ça fonctionnait pas donc j'ai mis les labels manuellement ----->
        <select id="IdRole" name="IdRole" class="form-control">
            <option value="">Sélectionnez un rôle</option> 
            <!----- Option vide par défaut qui sert à déclencher la validation Required ----->

            @foreach (var item in (List<SelectListItem>)ViewBag.Roles)
            {
                // On génère dynamiquement les options depuis ViewBag.Roles
                <option value="@item.Value">@item.Text</option> 
            }
        </select>

        <span asp-validation-for="IdRole" class="text-danger"></span> 
        <!----- Affiche les messages d’erreur liés à IdRole si la validation échoue ----->
    </div>

    <!-----  LOCALITÉ , même principe que pour les rôles ----->
    <div>
        <label for="IdLocalite">Localité</label>

        <select id="IdLocalite" name="IdLocalite" class="form-control">
            <option value="">Sélectionnez une localité</option>
            @foreach (var item in (List<SelectListItem>)ViewBag.Localites)
            {
                <option value="@item.Value">@item.Text</option>
            }
        </select>

        <span asp-validation-for="IdLocalite" class="text-danger"></span>
    </div>

    <!-----  NOM  ----->
    <div>
        <label for="Nom">Nom</label>
        <input type="text" name="Nom" id="Nom" class="form-control" />
        <span asp-validation-for="Nom" class="text-danger"></span>
    </div>

    <!-----  PRÉNOM  ----->
    <div>
        <label for="Prenom">Prénom</label>
        <input type="text" name="Prenom" id="Prenom" class="form-control" />
        <span asp-validation-for="Prenom" class="text-danger"></span>
    </div>

    <!-----  ADRESSE  ----->
    <div>
        <label for="Adresse">Adresse</label>
        <input type="text" name="Adresse" id="Adresse" class="form-control" />
        <span asp-validation-for="Adresse" class="text-danger"></span>
    </div>

    <!-----  EMAIL  ----->
    <div>
        <label for="Email">Adresse Mail</label>
        <input type="email" name="Email" id="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <!-----  MOT DE PASSE  ----->
    <div>
        <label for="MotDePasse">Mot de passe</label>
        <input type="password" name="MotDePasse" id="MotDePasse" class="form-control" autocomplete="new-password" />
        <span asp-validation-for="MotDePasse" class="text-danger"></span>
    </div>

    <!-----  BOUTON  ----->
    <button type="submit">Créer l'utilisateur</button> 
    <!----- Bouton d’envoi du formulaire ----->
</form>

<!----- Lien pour revenir à la liste des utilisateurs ----->
<a href="/admin/utilisateurs/list">← Retour à la liste</a>

<!----- Inclusion des scripts nécessaires à la validation côté client (à compléter) ----->
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}

```
