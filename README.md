# TP-03 Shell bash

### Exercice : paramètres

Le script affiche des informations sur les paramètres passés lors de son exécution. Il indique d'abord le nombre de paramètres ($#), affiche le nom du script ($0), puis montre le troisième paramètre ($3), et enfin liste tous les paramètres fournis ($@).

```
#!/bin/bash

echo  "Bonjour, vous avez rentré $# paramètres"
echo  "Le nom du script est $0"
echo  "Le 3ème paramètre est $3"
echo  "Voici la liste des paramètres : $@"
```

- $# : Affiche le nombre total de paramètres passés au script.
- $0 : Affiche le nom du script.
- $3 : Affiche le troisième paramètre (ou rien s'il n'y a pas de troisième paramètre).
- $@ : Affiche tous les paramètres sous forme de liste.


### Exercice : vérification du nombre de paramètres

Le script vérifie si deux arguments ont été passés en ligne de commande ($# renvoie le nombre d'arguments). Si c'est le cas, il concatène les deux arguments ($1 et $2) et les affiche. Sinon, il affiche un message d'erreur indiquant qu'il faut entrer exactement deux paramètres.

```
#!/bin/bash

if [ $# -eq 2 ]
then
concat="$1$2"
echo $concat
else
echo "Erreur, vous devez rentrer 2 paramètres."
fi

```
Si on exécute le script avec deux arguments, par exemple script.sh Hello World, le résultat sera HelloWorld.

### Exercice : argument type et droits

Le script vérifie si le fichier ou le répertoire spécifié en argument ($1) existe ; s'il n'existe pas, il affiche un message d'erreur et quitte. Ensuite, il détermine le type de l'élément : si c'est un répertoire, un fichier ordinaire (vide ou non vide), ou un fichier spécial (comme un lien symbolique ou un socket). Enfin, le script évalue les permissions d'accès (lecture, écriture, exécution) pour l'utilisateur courant et affiche ces informations de manière claire.
```
#!/bin/bash

if [ ! -e "$1" ]; then
  echo "Le fichier $1 n'existe pas."
  exit 1
fi


if [ -d "$1" ]; then
  echo "$1 est un répertoire."
elif [ -f "$1" ]; then
  if [ -s "$1" ]; then
    echo "$1 est un fichier ordinaire non vide."
  else
    echo "$1 est un fichier ordinaire vide."
  fi
else
  echo "$1 est un fichier spécial."
fi

PERMISSIONS=""
[ -r "$1" ] && PERMISSIONS="lecture" || PERMISSIONS="pas de lec>
[ -w "$1" ] && PERMISSIONS="$PERMISSIONS, écriture" || PERMISSI>
[ -x "$1" ] && PERMISSIONS="$PERMISSIONS, exécution" || PERMISS>
echo "\"$1\" est accessible par $USER en $PERMISSIONS."
```

### Exercice : Afficher le contenu d’un répertoire

Le script vérifie d'abord si l'argument passé (représenté par $1) est un répertoire valide. Si ce n'est pas le cas, il affiche un message d'erreur et quitte. Si c'est un répertoire valide, il liste d'abord les fichiers dans ce répertoire (en excluant les sous-répertoires), puis affiche les répertoires présents en les distinguant avec la commande ls -p

```
#!/bin/bash

if [ ! -d "$1" ]; then
  echo "$1 n'est pas un répertoire valide."
  exit 1
fi
echo "####### Fichiers dans $1/"
ls -p "$1" | grep -v /
echo "####### Répertoires dans $1/"
ls -p "$1" | grep /
```

### Exercice : Lister les utilisateurs


Le problème avec la commande suivante :

```
#!/bin/bash
for user in $(cat /etc/passwd); do echo $user; done
```
est qu'elle lit tout le contenu du fichier ```/etc/passwd``` comme une seule chaîne de caractères et effectue une itération mot par mot, au lieu d'itérer correctement sur chaque ligne (chaque entrée utilisateur).

#### Solution avec cut
On peut utiliser ```cut``` pour extraire uniquement les champs pertinents du fichier ```/etc/passwd```. Pour rappel, le fichier ```/etc/passwd``` a plusieurs champs séparés par des :. Le 3ème champ contient l'UID (User ID) :

- Utilisons cut pour extraire l'UID et le nom d'utilisateur.
- Nous devons filtrer les utilisateurs ayant un UID supérieur à 100.

```
#!/bin/bash
while IFS=: read -r username _ uid _; do
  if [ "$uid" -gt 100 ]; then
    echo "$username"
  fi
done < /etc/passwd
```

- ```IFS```=: permet de spécifier que les champs sont séparés par des ```:```
- On utilise la commande ```read -r``` pour lire les champs dans des variables (comme username, uid, etc.).
- Le test ```[ "$uid" -gt 100 ]``` filtre les UID supérieurs à 100.


Solution avec ```awk```
```awk``` est un outil très puissant pour traiter des fichiers texte comme ```/etc/passwd```. Il permet de traiter directement les lignes et de travailler avec les champs séparés par ```:```. Voici comment faire la même chose avec ```awk``` :

```
#!/bin/bash
awk -F: '$3 > 100 { print $1 }' /etc/passwd
```

### Exercice : Mon utilisateur existe t’il ?

```
#!/bin/bash

if [[ $1 == "--login" ]]; then
    id -u "$2" 2>/dev/null

elif [[ $1 == "--uid" ]]; then
    getent passwd "$2" >/dev/null && echo "$2"
fi
```
- ```id -u <login>``` renvoie l'UID de l'utilisateur si le login existe, sinon ne renvoie rien.
- ```getent passwd <UID>``` vérifie si l'UID existe dans le fichier /etc/passwd, et s'il existe, affiche l'UID.

Pour vérifier avec un login : 
-   ```script.sh --login <login>```

Pour vérifier avec un UID : 
-  ```script.sh --uid <UID>```

### Creation utilisateur

- Dans ```man useradd```, recherchez les options ```-m```, ```-d```, ```-u```, ```-g```, ```-c``` pour savoir comment créer un utilisateur avec les options adéquates (répertoire, UID, GID, commentaires).

- Dans ```man read```, cherchez l'option ```-p``` pour afficher une invite avant la saisie des informations (login, nom, prénom, etc.).

```
#!/bin/bash

[ "$USER" != "root" ] && echo "Erreur : Vous devez être root." && exit 1

read -p "Login : " login
read -p "Nom : " nom
read -p "Prénom : " prenom
read -p "UID : " uid
read -p "GID : " gid
read -p "Commentaires : " comments

id -u "$login" >/dev/null 2>&1 && echo "Erreur : L'utilisateur $login existe déjà." && exit 1
getent passwd "$uid" >/dev/null && echo "Erreur : L'UID $uid existe déjà." && exit 1
[ -d "/home/$login" ] && echo "Erreur : Le répertoire /home/$login existe déjà." && exit 1

useradd -m -d "/home/$login" -u "$uid" -g "$gid" -c "$prenom $nom, $comments" "$login" && \
echo "L'utilisateur $login a été créé."
```

### Exercice : lecture au clavier

- La commande ```read``` permet de capturer une entrée de l'utilisateur et de l'affecter à une variable.
- La commande ```file <nom_du_fichier>``` détermine le type d'un fichier (texte, image, exécutable, etc.) en examinant son contenu.
- La commande ```more <nom_du_fichier>``` permet d'afficher le contenu d'un fichier page par page dans le terminal.

#### Questions :

- Comment quitter more ?
  - Il faut appuyer sur la touche q pour quitter more.

- Comment avancer d’une ligne ?
  - Il faut appuyez sur la touche Entrée (ou flèche vers le bas) pour avancer d'une ligne.

- Comment avancer d’une page ?
  - Il faut appuyez sur la touche Espace pour avancer d'une page complète.

- Comment remonter d’une page (avec less) ?
  - Appuyez sur la touche b (pour "back") pour remonter d'une page complète. (Cette option fonctionne avec less, mais pas avec more).

- Comment chercher une chaîne de caractères ?
  - Tapez / suivi de la chaîne que vous voulez chercher, puis appuyez sur Entrée. Par exemple, pour chercher "root", tapez /root.

- Comment passer à l’occurrence suivante ?
  - Après une recherche avec /, appuyez sur n pour passer à l'occurrence suivante de la chaîne trouvée.

```
#!/bin/bash

for fichier in "$1"/*; do
    if file "$fichier" | grep -q "text"; then
        read -p "Voulez-vous visualiser le fichier $(basename "$fichier") ? (oui/non) " reponse
        [ "$reponse" == "oui" ] && more "$fichier"
    fi
done
```

On  parcourt un répertoire spécifié en argument, identifie les fichiers texte, et propose à l'utilisateur de les visualiser avec more. Si l'utilisateur répond "oui", le fichier sera affiché page par page. Le script utilise la commande file pour vérifier le type des fichiers.

### Exercice : appréciation

```
#!/bin/bash

while true; do
    read -p "Entrez une note (ou 'q' pour quitter) : " note

    if [ "$note" == "q" ]; then
        echo "Programme terminé."
        exit 0
    elif (( $(echo "$note >= 16" | bc -l) && $(echo "$note <= 20" | bc -l) )); then
        echo "très bien"
    elif (( $(echo "$note >= 14" | bc -l) && $(echo "$note < 16" | bc -l) )); then
        echo "bien"
    elif (( $(echo "$note >= 12" | bc -l) && $(echo "$note < 14" | bc -l) )); then
        echo "assez bien"
    elif (( $(echo "$note >= 10" | bc -l) && $(echo "$note < 12" | bc -l) )); then
        echo "moyen"
    elif (( $(echo "$note < 10" | bc -l) )); then
        echo "insuffisant"
    else
        echo "Entrée invalide. Entrez une note entre 0 et 20."
    fi
done
```

- Le programme tourne en boucle jusqu'à ce que l'utilisateur entre ```q``` pour quitter.
- ```read -p``` : Demande à l'utilisateur d'entrer une note ou q pour quitter.
- Conditions ```if``` et ```elif``` : Compare la note et affiche un message correspondant selon les plages définies.
- ```bc -l``` : Utilisé pour les comparaisons de nombres décimaux.

  ### Sacha CLEMENT
