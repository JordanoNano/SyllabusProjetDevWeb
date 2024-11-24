5. Installer SQL Server et ses dépendances sur Visual Studio Code
5.1. Installer SQL Server
Si SQL Server n'est pas encore installé, vous pouvez l'installer en suivant les étapes sur la documentation officielle de SQL Server.

5.2. Installer les packages nécessaires pour SQL Server avec Entity Framework
Dans Visual Studio Code, ouvrez le terminal et assurez-vous d'être dans le répertoire de votre projet. Exécutez les commandes suivantes pour ajouter les dépendances nécessaires :

dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design

5.3. Créer les modèles et le contexte
Créez un dossier Models et placez-y vos classes de modèle.
Créez un dossier Data et placez-y la classe de contexte (DbContext), qui permettra d'interagir avec la base de données.
5.4. Adapter le fichier Program.cs
Dans le fichier Program.cs, configurez le contexte pour utiliser SQL Server. Exemple :

public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        // Configure DbContext
        builder.Services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

        var app = builder.Build();
        app.Run();
    }
}
5.5. Créer la première migration
Utilisez les outils Entity Framework pour ajouter votre première migration :

dotnet ef migrations add InitialCreate
Cette commande génère une migration basée sur l'état actuel de votre modèle de données.

5.6. Appliquer les migrations et mettre à jour la base de données
Une fois la migration générée, appliquez-la à la base de données avec la commande suivante :
dotnet ef database update
Cela créera les tables dans la base de données en fonction des migrations générées.

5.7. Ajouter des informations dans les tables
Après avoir créé les tables, vous pouvez ajouter des données dans celles-ci via votre application en utilisant des méthodes comme Add() et SaveChanges() dans votre code C#.

Exemple :

using (var context = new ApplicationDbContext())
{
    var utilisateur = new Utilisateur
    {
        Nom = "Jordano Modesto",
        Email = "jordanomodestonano@gmail.com"
    };

    context.Utilisateurs.Add(utilisateur);
    context.SaveChanges();
}
Cela ajoutera un utilisateur à votre table Utilisateurs dans la base de données.

