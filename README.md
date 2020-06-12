# Travaux pratiques pour formation docker

## TP no 02

Premier container... Qui a dit pod ?

### Démarrons notre premier Pod

La commande 'kubectl create' existe, mais pour un nombre limité de ressources (cf documentation).

Pour exécuter un pod, il existe la commande 'kubectl run' avec laquelle on précise un nom de pod, un nom d'image,...

Nous allons prendre tout de suite les bonnes habitudes :
* utiliser un fichier yaml
* mieux encore, générer le squelette du fichier yaml  :bowtie:

Pour notre premier pod, faisons simple : un serveur web (au hasard, nginx)
* nom du pod : webserver
* nom de l'image docker : nginx:latest 
* mode dry-run
* format de sortie yaml

```bash
kubectl run webserver --image=nginx:latest --dry-run=client -o yaml
```

Vous retrouvez la formule AKMS.
Vous pouvez partir de ce fichier, il est très bien.
S'il est encore trop chargé, voici une version minimale

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  containers:
  - image: nginx:latest
    name: webserver
```

Redirigez/copiez le contenu dans un fichier (exemple pod-nginx.yaml) et créez enfin ce premier pod.

```bash
kubectl create -f pod-nginx.yaml
```

A ce stade, je dois avoir :
* un élève heureux  :smile:
* 3 élèves  :disappointed_relieved:  :angry:  :sob:

En effet, nous travaillons sur un même cluster et nous avons oublié de spécifier la notion de namespace.
Pour l'heureux élève qui aura réussi à créer son pod dans le namespace default, il faut le détruire.

```bash
kubectl delete -f pod-nginx.yaml
```


### Utilisons un namespace

Corrigeons notre erreur dans la partie metadata (remplacez jcanon par votre nom de namespace):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
  namespace : jcanon
spec:
  containers:
  - image: nginx:latest
    name: webserver
```

Créons le pod dans le bon namespace :

```bash
kubectl create -f pod-nginx.yaml
```

Vérifiez :

```bash
kubectl get pods -n <namespace>
kubectl get pods --all-namespaces
```

Si tout s'est bien passé, on voit notre pod au status RUNNING.
Si ce n'est pas le cas, prévenez-moi.

Par ailleurs, dans la colonne ready, on peut lire "1/1".
Cela correspond au nombre de container running sur le nombre de container total de notre pod.

### Et maintenant ?

C'est bien, nous avons enfin un pod. Et qu'est-ce qu'on en fait ?

Notre pod tourne correctement dans kubernetes. C'est déjà un bel objectif atteint.

A ce stade, il n'y a pas moyen de lui envoyer des requêtes depuis la VM.
Nous en verrons plus plus tard.

Nous pouvons obtenir des informations sur ce qu'il s'est passé dans le cluster via les évènements :

```bash
kubectl get events -n <namespace>
```

Nous pouvons obtenir les logs du container :

```bash
kubectl logs webserver -n <namespace>
```

Mieux encore, pourquoi ne pas entrer dans le container, comme nous le faisions avec docker ?
Voici la commande pour lancer un shell bash (Notez le -- pour séparer les options kubectl de la commande à exécuter)

```bash
kubectl exec -it webserver -n <namespace> -- bash
```

Une fois à l'intérieur, je vous propose :
* de déduire quel est l'os de base
* d'installer de quoi voir les processus internes au container (commande ps)
* de visualiser effectivement les processus internes (quels sont-ils ?)
* d'installer des outils réseau
* d'observer sur quel port et quelle interface écoute le serveur web
* d'envoyer des requêtes sur différentes urls

```bash
cat /etc/issue
apt-get update && apt-get install -y procps
ps -ef
apt install -y net-tools
netstat -plntu
curl http://localhost/
ifconfig -a
curl http://<ip_eth0>/
```

Faites moi part de toutes vos remarques ou questions, n'hésitez pas.

Pour quitter le container, utilisez 'exit' ou 'Ctrl-D'

### J'en ai marre de préciser le namespace à chaque commande !

Afin de pouvoir travailler chacun dans son propre espace, je vous ai imposé l'utilisation d'un namespace.
Dans chacun des YAML à venir, vous pourrez (devrez) ajouter la ligne "namespace: <namespace>" dans la partie metadata.
De même, sur toutes les lignes de commandes kubectl, vous devrez préciser "--namespace=<namespace>" ou "-n <namespace>"

C'est impératif et c'est à conserver en mémoire en permanence, sous peine de ne pas voir les bons objets ou de créer les objets au mauvais endroit.

Cependant, vous simplifier la vie et que vous puissiez vous focaliser sur le cours, nous allons ajouter un paramétrage de kubectl.
Ceci nous permet de découvrir une nouvelle fonctionnalité : la configuration de la CLI.

Je vous ai déjà présenté le fichier de configuration de kubectl : ~/.kube/config

C'est un fichier au format yaml :

```yaml
apiVersion: v1
kind: Config
preferences: {}
clusters: {}
users: {}
contexts: {}
```

Il possède donc un type de ressource (kind) du nom de Config et une version d'API à v1.
Comme on peut le voir, il contient :
* les informations de un ou plusieurs clusters (chez nous 1)
* les informations sur un ou plusieurs utilisateurs (chez nous 1)
* les informations sur un ou plusieurs contexts

C'est la notion de context qui nous intéresse.
Elle permet de regrouper sous un seul nom :
* le cluster sur lequel on travaille
* avec quelle identité on travaille
* dans quel namespace on travaille par défaut

C'est cette dernière notion qui nous intéresse  :wink:

Observons tout d'abord combien de contextes existent sur notre environnement.
Pour cela utilisons la commande 'config' de kubectl.
La particularité de cette commande est qu'elle ne travaille qu'en local, sur le fichier de config, et il n'y a aucun lien avec un quelconque cluster kubernetes.

```bash
kubectl config get-contexts
```

Parmi la liste des contexts existants, une astérisque nous indique le context courant.
Cette commande nous donne toutes les informations : cluster, user, namespace.
On constate que le namespace est vide.

Note : pour connaître le context courant (sans toutes les infos), il y a la commande suivante :

```bash
kubectl config current-context
```

Nous pouvons forcer notre namespace personnel via la commande suivante :

```bash
kubectl config set-context aks-00 --namespace=<namespace>
```

A partir de maintenant, le namespace par défaut sera le votre.
Mais souvenez-vous que certaines ressources sont globales et non liées à un namespace !

C'est tout pour ce TP. Félicitations !  :ok_hand:

