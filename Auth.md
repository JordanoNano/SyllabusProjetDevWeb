Dans le program.cs (point d'entrée de notre app)
// IMPORTANT : placer UseAuthentication avant UseAuthorization
// ce sont des middleware -> s'exécute à chaque requête HTTP
app.UseAuthentication();
app.UseAuthorization();

cshtml 
```bash
@{
    ViewData["Title"] = "Connexion";
}

<h2>Connexion</h2>

<form method="post">
    <div class="mb-3">
        <label for="email" class="form-label">Adresse email</label>
        <input type="email" name="email" class="form-control" required />
    </div>
    <div class="mb-3">
        <label for="password" class="form-label">Mot de passe</label>
        <input type="password" name="password" class="form-control" required />
    </div>
    <button type="submit" class="btn btn-primary">Se connecter</button>

    @if (ViewBag.Erreur != null)
    {
        <div class="alert alert-danger mt-3">@ViewBag.Erreur</div>
    }
</form>
```


controller : 
```bash
// Importation des outils nécessaires au MVC (Models, Controllers, Views)
using System.Security.Claims; // Permet de créer des informations d'identité liées à un utilisateur (claims)
using Microsoft.AspNetCore.Authentication; // Fournit les méthodes pour gérer l'authentification (connexion/déconnexion)
using Microsoft.AspNetCore.Authentication.Cookies; // Utilisation de l'authentification basée sur les cookies
using Microsoft.AspNetCore.Mvc; // Base des contrôleurs et vues dans une appli ASP.NET MVC

// Importation Entity Framework Core (requêtes, Include, SaveChanges,...)
using Microsoft.EntityFrameworkCore;

// Importation du DbContext de l'application
using MonApplication.Data;

public class AuthController : Controller // Ce contrôleur gère tout ce qui touche à l'auth
{
    private readonly ECommerceDbContext _context; // Accès à la base de données
    private readonly ILogger<AuthController> _logger; // Outil de log (utile pour le debug, erreurs, etc.)

    // Constructeur avec injection de dépendances
    // => le framework nous injecte automatiquement une instance de ECommerceDbContext et ILogger<AuthController>
    public AuthController(ECommerceDbContext context, ILogger<AuthController> logger)
    {
        _context = context;
        _logger = logger;
    }

    // Méthode GET qui affiche simplement le formulaire de connexion (Login.cshtml)
    public IActionResult Login()
    {
        return View();
    }

    // Méthode POST appelée après soumission du formulaire de connexion
    [HttpPost]
    public async Task<IActionResult> Login(string email, string password)
    {   //async et await permettent d'attendre le résultat d'une opération sans bloquer l'exécution du programme, en reprenant le code une fois la réponse reçue.
        // On recherche l'utilisateur dans la base de données via son email
        // Include(u => u.Role) permet de charger en même temps les infos du rôle (relation EF Core)
        var utilisateur = await _context.Utilisateur
            .Include(u => u.Role) // On inclut le rôle pour l'utiliser ensuite dans les claims
            .FirstOrDefaultAsync(u => u.Email == email); // Ne ramène qu'un utilisateur ou null

        // Si l'utilisateur n'existe pas ou que son mot de passe n'est pas bon (comparé via BCrypt), on affiche une erreur
        if (utilisateur == null || !BCrypt.Net.BCrypt.Verify(password, utilisateur.MotDePasse))
        {
            ViewBag.Erreur = "Identifiants invalides"; // Variable utilisée dans la vue pour afficher un message
            return View(); // On réaffiche la page de login
        }

        // Si tout est bon, on prépare les "claims"
        // Un claim est une information sur l'utilisateur (nom, email, rôle...)
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.Name, utilisateur.Prenom + " " + utilisateur.Nom), // Nom complet
            new Claim(ClaimTypes.Email, utilisateur.Email), // Email (utile à afficher ou comme identifiant)
            new Claim(ClaimTypes.Role, utilisateur.Role.IntituleRole), // Le rôle de l'utilisateur (Admin, Client, etc.)
            new Claim("IdUtilisateur", utilisateur.IdUtilisateur.ToString()) // Claim personnalisé pour stocker l'ID (ex : lien avec panier, commandes, etc.)
        };

        // On crée une identité avec tous ces claims, liée au type d'authentification utilisé (cookie ici)
        var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);

        // Le ClaimsPrincipal représente l'utilisateur connecté avec son identité (il peut en avoir plusieurs en théorie)
        var principal = new ClaimsPrincipal(identity);

        // On signe l'utilisateur, ce qui génère un cookie d'authentification stocké côté client
        await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, principal);

        // Redirection vers la liste des utilisateurs après connexion (page d'admin)
        return Redirect("/admin/utilisateurs/list");
    }

    // Méthode POST pour déconnecter l'utilisateur
    [HttpPost]
    public async Task<IActionResult> Logout()
    {
        // Supprime le cookie d’authentification pour cet utilisateur
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);

        // Redirige vers la page de login après déconnexion
        return RedirectToAction("Login");
    }

    // Page d'accès refusé (si l'utilisateur est connecté mais n'a pas les droits nécessaires)
    public IActionResult Denied()
    {
        // Pas de vue dédiée ici, juste du texte
        return Content("Accès refusé");
    }
}
```
