Exercice un peu différent cette fois, je vous donne toutes les indications en français, à vous de traduire en code, et on se passe de l'IA cette fois (jouez le jeu, c'est pour votre bien) :)

1) quels sont les champs sur lesquels vous voulez que l'utilisateur puisse faire une recherche ?

2) dans la vue qui affiche la liste, ajouter un formulaire, on oublie pas sa methode et son action

3) on ajoute : Un champ de texte pour taper la recherche (attribut type,name, placeholder ?)
               Un bouton pour lancer la recherche (attribut type)
               Un lien pour la réinitialiser (attribut lien )

4) Dans le controller, on ajoute une méthode search, ou oublie pas l'annotation http ... (get post .. autre ? )


5) cette méthode attend un paramètre, lequel ?


6) premier logger pour vérifier si on atteint bien la méthode


7) on va rechercher le contexte avec les include, comment faire une recherche sur les rôles et localité ?


8) On peut convertir la recherche en minuscule 


9) Filtrer la liste des utilisateurs en ne gardant que ceux dont au moins un champ contient le mot-clé tapé (ici on va utiliser .Where, .Contains et la méthode pour mettre en minuscule, ainsi que le signe pour "ou")(!! c'est une lamba )
utilisateurs = utilisateurs.... où le nom en minuscule contient le mot clé tapé en minuscule OU le prenom en min.........


10) on stocke le résultat dans une liste


11) on renvoie la vue avec le résultat
