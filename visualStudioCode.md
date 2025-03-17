# Documentation sur l'installation de Visual Studio Code et la gestion des outils .NET

## 1) Installer Visual Studio Code via le Microsoft Store

### Étapes pour l'installation :
1. Ouvrez le **Microsoft Store** sur votre ordinateur Windows.
2. Recherchez "**Visual Studio Code**" dans la barre de recherche.
3. Cliquez sur le bouton **Installer** pour télécharger et installer Visual Studio Code (VS Code).

## 2) Installer le SDK .NET

### Téléchargement et installation du SDK .NET :
1. Allez sur le site officiel de .NET : [https://dotnet.microsoft.com](https://dotnet.microsoft.com).
2. Cliquez sur le bouton "**Download .NET**" et sélectionnez la dernière version stable du SDK .NET.

### Vérification de l'installation :
Ouvrez le terminal et tapez la commande suivante pour vérifier que le SDK .NET est installé correctement :
```bash
dotnet --version
```
## 3) Installer les extensions dans Visual Studio Code

### Extensions recommandées pour le développement C# et Entity Framework

#### 3.1) Extension C# (par Microsoft)
1. Ouvrez Visual Studio Code.
2. Allez dans l'onglet **Extensions** (ou utilisez le raccourci `Ctrl+Shift+X`).
3. Dans la barre de recherche, tapez "**C#**".
4. Sélectionnez l'extension officielle de Microsoft (généralement en haut des résultats) et cliquez sur **Install**.

#### 3.2) Extension Entity Framework (par Microsoft)
1. Toujours dans l'onglet **Extensions**, tapez "**Entity Framework**" dans la barre de recherche.
2. Installez les outils nécessaires pour la gestion des migrations et des bases de données.

## 4) Redémarrer Visual Studio Code

Après avoir installé les extensions, il est recommandé de redémarrer Visual Studio Code pour s'assurer que toutes les extensions sont correctement chargées.

## 5) Résolution de l'erreur `dotnet new webapp -n MonApplication`

Si la commande `dotnet new webapp -n MonApplication` renvoie une erreur liée à une variable d'environnement manquante, il est probable que votre machine n'ait pas correctement configuré le **.NET SDK** dans le PATH. Assurez-vous que le chemin vers le SDK .NET est correctement ajouté à la variable d'environnement **PATH** de votre système.

1. Ouvrez les **Paramètres système avancés**.
2. Cliquez sur **Variables d'environnement**.
3. Dans la section **Variables système**, recherchez la variable **Path**.
4. Ajoutez le chemin vers le SDK .NET, généralement : `C:\Program Files\dotnet`.

## 6) Vérification de la version de `winget` (gestionnaire de paquets Windows)

Pour vérifier si vous avez installé le gestionnaire de paquets **winget**, tapez la commande suivante dans le terminal :
```bash
winget --version
```
