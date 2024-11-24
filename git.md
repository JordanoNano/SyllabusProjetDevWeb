# Documentation pour l'Initialisation du Projet et la Configuration de Git.

Cette documentation couvre les étapes d'installation et de configuration nécessaires pour un projet utilisant Git, SQL Server, et Entity Framework dans un environnement Visual Studio Code.

## Prérequis

Assurez-vous que les outils suivants sont installés sur votre machine :
- **Git** : pour la gestion de version du projet.
- **Winget** : pour installer des paquets via la ligne de commande.

### 1. Installer Git et vérifier sa version

Si Git n'est pas encore installé, vous pouvez l'installer avec **Winget** :

```bash
winget install Git.Git
```
Une fois installé, vérifiez la version de Git avec la commande suivante :

bash
Copier le code
git --version
2. Configurer Git
Avant de commencer à travailler avec Git, vous devez configurer votre nom d'utilisateur et votre adresse e-mail. Ces informations seront utilisées pour lier vos commits à votre identité.

bash
Copier le code
git config --global user.name "Jordano Modesto"
git config --global user.email "jordanomodestonano@gmail.com"
3. Créer un dépôt Git sur GitHub
Créez un nouveau dépôt sur GitHub.
Copiez l'URL du dépôt, elle sera utilisée pour ajouter un remote à votre projet local.
4. Initialiser le projet Git localement
Naviguez dans le répertoire de votre projet et initialisez un dépôt Git local :
bash
git init

Ajoutez tous les fichiers au suivi Git :
git add .
Effectuez le premier commit :
git commit -m "[DEVWEB-1]Initial commit"
Ajoutez le dépôt distant (remote) GitHub :
git remote add origin https://github.com/JordanoNano/MonApplication.git
Vérifiez les branches disponibles :
git branch
Créez une nouvelle branche pour le développement :
git checkout -b dev-test
Poussez la branche dev-test sur GitHub :
git push -u origin dev-test


