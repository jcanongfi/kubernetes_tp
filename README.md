# Travaux pratiques pour formation docker

## TP no 01

Premier contact avec kubernetes, faire sa place

### Découvrir les namespaces

Dans le TP précédents, nous n'avions décourvert qu'une unique resource : le service kubernetes (celui qui représente l'API server)

Voyons désormais l'envers du décor : il existe des resources cachées...

```bash
kubectl get all --all-namespaces
```

Vous découvrez qu'il existe en effet d'autres resources que nous préciserons plus tard (pods, services, daemonsets, deployements, replicasets)

Note : l'option -A est la version courte de --all-namespaces

Observons la première colonne NAMESPACE.
Il s'agit comme son nom l'indique d'un espace de nommage, pour regrouper les resources.
Le service kubernetes est placé dans un namespace default
Les autres ressources (interne à kubernetes) sont placées dans un namespace kube-system.

Mais combien y en a-t-il au final ?

```bash
kubectl get namespaces
```

Nous en découvrons 4 :
* default => le namespace par défaut. Il porte bien son nom, si on ne précise pas le namespace (sur un get, sur un create,...), c'est lui qui sera choisi
* kube-node-lease => namespace récent (passé stable en version 1.17) sert pour gérer des node status. Peu utilisé, on est actuellement autorisé à l'effacer
* kube-public => namespace dédié aux ressources "publiques". Vous pouvez l'investir. Peu utilisé.
* kube-system => namespace réservé pour l'usage interne de kubernetes.

Afin d'éviter d'éviter d'avoir un tas de ressources mélangées sous peine de ne plus vous y retrouver, la bonne pratique consiste à créer des namespaces dédiés.
(Par application, par business unit, pour la supervision, ...)

### Créons notre propre namespace

Voici donc le moment de créer notre première ressource !

Je vous propose pour cette première d'utiliser 3 méthodes.
Je vous propose aussi de choisir un nom pour votre namespace. Par défaut, je vous suggère votre identifiant (exemple jcanon), mais vous êtes libres tant qu'il n'y a pas de conflit.

#### Tout en ligne de commande

```bash
kubectl create namespace <namespace>
```

C'est basique et ça fait le boulot !

Vérifions la bonne création par 2 commandes :

```bash
kubectl describe namespace <namespace>
kubectl get namespaces
```

Note : la commande describe nous montre qu'il est possible d'ajouter des labels et des annotations à un namespace (ici, il n'y en a pas).

Détruisez le namespace, nous allons le recréer avec une seconde méthode

```bash
kubectl delete namespace <namespace>
```

#### En utilisant un fichier yaml

Dans le répertoire courant, vous trouverez le fichier namespace.yaml.
Editez-le et remplacez le nom du namespace par celui que vous avez choisi.

De plus, vous pouvez en profiter pour créer un ou plusieurs labels (clef/valeur). Vous en avez deux en exemple dans le fichier.

Enfin, vous pouvez observer qu'il n'y a pas de catégorie spec pour ce yaml. Pour namespace, elle n'est pas obligatoire, mais nous verrons qu'elle existe.

Une fois le fichier satisfaisant, créez à nouveau le namespace :

```bash
kubectl create -f namespace.yaml
```

Note : pour la création avec un fichier, on ne précise pas le type de ressource (kind). En effet, tout est déjà décrit dans le fichier.

Vérifiez (y compris la présence des labels) :

```bash
kubectl describe namespace <namespace>
kubectl get namespaces
```

Vous pouvez le détruire une dernière fois, nous allons utiliser une dernière méthode.

```bash
kubectl delete namespace <namespace>
```

#### En utilisant un fichier json

Vous remarquerez que je ne vous ai pas préparé de fichier json dans ce tp.
Pour écrire un tel fichier, il faudrait revenir à la documentation, la référence API de kubernetes ou un moteur de recherche pour obtenir un exemple tout fait.

Je vais vous proposer une façon détournée assez efficace pour gagner du temps.

L'option --dry-run vous permet de jouer à blanc une commande. La stratégie "client" ne joue que côté client, tandis que la stratégie "server" ajoute les propriétés du serveur.
Essayons :

```bash
kubectl create namespace toto --dry-run=client
```

Effectivement, il n'a rien fait.

La seconde astuce, que nous allons combiner à la première, est le format de sortie du résultat via l'option -o.
Vous pouvez choisir entre yaml, json, wide ou des formats custom.
Utilisons ici la sortie json :

```bash
kubectl create namespace toto --dry-run=client -o json
```

Youpi, ça marche !
Conservons le résultat dans un fichier namespace.json.

```bash
kubectl create namespace toto --dry-run=client -o json > namespace.json
```

Editez le fichier selon vos souhaits. Notamment, remplacez "toto" par le nom de votre namespace.
Et recréez pour la dernière fois votre namespace.

```bash
kubectl create -f namespace.json
```

Le TP est terminé. On va conserver ce namespace pour les autres TP.

