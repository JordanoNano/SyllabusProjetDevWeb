# Documentation : Implémentation de l'approche Code First avec ASP.NET Core et Entity Framework

##  Introduction
Ce document explique comment mettre en place un projet **ASP.NET Core** en utilisant **Entity Framework Core (EF Core)** avec l’approche **Code First** et **SQL Server** comme base de données.

Nous allons voir comment :
- Installer les packages nécessaires.
- Créer les modèles de données.
- Configurer la connexion SQL Server.
- Générer la base de données avec les migrations.

---

##   Installation des packages NuGet
Avant de commencer, il faut installer **Entity Framework Core** pour **SQL Server** et les outils nécessaires :

```sh
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```
```sh
dotnet add package Microsoft.EntityFrameworkCore.Tools
```
Ces commandes ajoutent **Entity Framework Core** et les outils pour **générer et gérer les migrations**.


---

##   Création des modèles de données (**Code First**)
Nous allons créer les entités principales de notre application e-commerce.

 **Modèle `Utilisateur` :**
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class Utilisateur
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }
    
    [Required, StringLength(50)]
    public string Nom { get; set; }
    
    [Required, StringLength(50)]
    public string Prenom { get; set; }
    
    [Required, EmailAddress]
    public string Email { get; set; }
    
    [Required]
    public string MotDePasse { get; set; } 
    
    [Required]
    public string Role { get; set; }
    
    public string Adresse { get; set; }
}
```

 **Modèle `Produit` :**
```csharp
public class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Description { get; set; }
    public decimal Prix { get; set; }
    public int Stock { get; set; }
    public int CategorieId { get; set; }
    public Categorie Categorie { get; set; }
    public string Image { get; set; } 
}
```

 **Modèle `Catégorie` :**
```csharp
public class Categorie
{
    public int Id { get; set; }
    public string Nom { get; set; }
}
```

 **Modèle `Panier` :**
```csharp
public class Panier
{
    public int Id { get; set; }
    public int UtilisateurId { get; set; }
    public Utilisateur Utilisateur { get; set; }
}
```

 **Modèle `PanierProduit` (relation N-N)** :
```csharp
public class PanierProduit
{
    public int PanierId { get; set; }
    public Panier Panier { get; set; }
    public int ProduitId { get; set; }
    public Produit Produit { get; set; }
    public int Quantite { get; set; }
}
```

 **Modèle `Commande` :**
```csharp
public class Commande
{
    public int Id { get; set; }
    public int UtilisateurId { get; set; }
    public Utilisateur Utilisateur { get; set; }
    public DateTime DateCommande { get; set; } 
    public decimal Total { get; set; }
    public string Statut { get; set; } 
}
```

 **Modèle `CommandeProduit` (relation N-N)** :
```csharp
public class CommandeProduit
{
    public int CommandeId { get; set; }
    public Commande Commande { get; set; }
    public int ProduitId { get; set; }
    public Produit Produit { get; set; }
    public int Quantite { get; set; }
}
```

 **Modèle `Paiement` :**
```csharp
public class Paiement
{
    public int Id { get; set; }
    public int CommandeId { get; set; }
    public Commande Commande { get; set; }
    public decimal Montant { get; set; }
    public string Statut { get; set; } 
    public string Methode { get; set; }
}
```

---

##   Création du `DbContext`
Le **`DbContext`** est la classe qui permet d'interagir avec la base de données.

 **Fichier `ECommerceDbContext.cs`** :
```csharp
using Microsoft.EntityFrameworkCore;

namespace MonApplication.Data
{
    public class ECommerceDbContext : DbContext
    {
        public DbSet<Utilisateur> Utilisateurs { get; set; }
        public DbSet<Produit> Produits { get; set; }
        public DbSet<Categorie> Categories { get; set; }
        public DbSet<Panier> Paniers { get; set; }
        public DbSet<PanierProduit> PaniersProduits { get; set; }
        public DbSet<Commande> Commandes { get; set; }
        public DbSet<CommandeProduit> CommandesProduits { get; set; }
        public DbSet<Paiement> Paiements { get; set; }

        public ECommerceDbContext(DbContextOptions<ECommerceDbContext> options) : base(options) { }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<PanierProduit>().HasKey(pp => new { pp.PanierId, pp.ProduitId });
            modelBuilder.Entity<CommandeProduit>().HasKey(cp => new { cp.CommandeId, cp.ProduitId });
        }
    }
}
```

---

##   Configuration de la connexion SQL Server
Dans le fichier `appsettings.json`, ajoutez la **chaîne de connexion** :
```json
"ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=ECommerceDB;Trusted_Connection=True;MultipleActiveResultSets=true";TrustServerCertificate=True"
}
```

---



Puis dans **`Program.cs`**, ajoutez :
```csharp
using Microsoft.EntityFrameworkCore;
using MonApplication.Data;

var builder = WebApplication.CreateBuilder(args);

// Connexion SQL Server
builder.Services.AddDbContext<ECommerceDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();

var app = builder.Build();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

---

## Génération de la base de données avec les migrations
Une fois les modèles et la configuration terminés, il faut générer la base de données avec **EF Core Migrations**.

 ** Créer une première migration** :
```sh
dotnet ef migrations add InitialCreate
```
 ** Appliquer la migration pour créer la base de données** :
```sh
dotnet ef database update
```
Après cette commande, la base **`ECommerceDB`** sera créée dans SQL Server avec toutes les tables.

---


