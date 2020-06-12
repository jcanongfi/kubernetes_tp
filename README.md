# Travaux pratiques pour formation docker

## TP no 05

Et pour les montées de version ?

### Tôt ou tard, il faudra monter de version

Si vous avez bien compris comment fonctionne un replicaset, il ne suffit pas de modifier la version dans le fichier web-replicaset.yaml.
En effet, il ne s'agit que d'un template qui sert à créer les nouveaux pods en cas de bseoin.

Faisons l'exercice.
Dans le fichier web-replicaset.yaml de ce TP, vous constaterez que la version du container a été mise à jour à 1.19.

Tentez d'appliquer le nouveau fichier :

```bash
kubectl apply -f web-replicaset.yaml
```

La modification est prise en compte, mais rien ne se passe (excepté le nombre de pod si vous n'étiez pas revenu à 3)

Observez les différents pod et cherchez la version de l'image, elle sera toujours à 1.18.

```bash
kubectl describe pod/web-rs-xxxxx
```

Pour forcer la prise en compte, il faut tuer ces "anciens" pods et le replicaset créera les nouveaux à partir du template, qui désormais prône la version 1.19.

```bash
kubectl delete pod/web-rs-xxxxx
```

Observez (avec describe), les nouveaux pods seront bien en version 1.19.

### Utilisons les Deployments

C'est donc pour gérer ces problématiques de montée de version qu'on été créé les Deployement.

Détruisez le replicaset précédent pour repartir sur de bonnes bases.

```bash
kubectl delete -f web-replicaset.yaml
```

Observons maintenant le fichier web-deployment.yaml. Excepté le "kind", nous sommes sur la même définition que pour un ReplicaSet.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  labels:
    truc: label-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.17
```

Créez ce Deployment :

```bash
kubectl create -f web-deployment.yaml
```

Observez le résultat :
* Le Deployement est créé (avec son nom)
* Un ReplicaSet est créé (avec pour nom replicaset.apps/<nom-deployement>-9999999999)
* Des pods sont créés (avec pour nom pod/<nom-deployement>-9999999999-abcde)

```bash
kubectl get all
```

**Note** : vous pouvez toujours vérifier qu'ils font partie des Endpoints du service (à cause du label 'app: web') et qu'ils acceptent des requêtes HTTP sur le nom DNS du loadbalancer.

Tout est au beau fixe.
Il est temps de changer de version

### Montée de version par fichier yaml

La méthode la plus simple et classique consiste à modifier le fichier web-deployement.yaml.
Mettez à jour l'image pour passer à la version 1.18 de nginx, et appliquez.

```bash
kubectl apply -f web-deployment.yaml
```

Observez les changements :
* Un second ReplicaSet a été créé
* Si vous êtes attentifs et rapides, vous aurez peut-être le temps de voir migrer les pods

```bash
kubectl get all
```

Vous pouvez constater l'état en cours de migration (ici réussie) :

```bash
kubectl rollout status deployment web-deployment
```

Vous pouvez consulter l'historique des révisions :

```bash
kubectl rollout history deployment web-deployment
```

Pour les puristes, vous voyez une colonne CHANGE-CAUSE remplie à <none> et cela vous chagrine.
Sachez que lors des changements (ici kubectl apply -f), il est possible d'enregistrer le changement avec l'option --record.
C'est déjà une bonne information (la ligne de commande saisie), mais ça ne nous donne pas le contenu du changement.

Il est possible si vous le souhaitez de le préciser librement avec une annotation :

```bash
kubectl annotate deployment/web-deployment kubernetes.io/change-cause="image mis à jour en 1.18" --record
```

Regardez, c'est quand même plus explicite !

```bash
kubectl rollout history deployment web-deployment
```

### Montée de version en ligne de commande

Pour les adeptes de la ligne de commande, tant que le changement est tracé, vous pouvez le faire de cette façon :

```bash
kubectl set image deployment/web-deployment nginx=nginx:1.19 --record
```

Observez les ressources :

```bash
kubectl get all
```

**Note** : Vous pouvez vérifier avec describe que l'image des pods est bien en version 1.19

Observez les révisions :

```bash
kubectl rollout history deployment web-deployment
```

### Je veux revenir en version 1.18 

Rien de plus facile, il suffit d'utiliser undo :

```bash
kubectl rollout undo deployment web-deployment
```

Et si on veut revenir encore plus en arrière, on utilise --to-revision

```bash
kubectl rollout undo deployment web-deployment --to-revision=1
```

### Observez un cas de blocage

Il y a des fois ou la modification n'aboutit pas.
Pour simuler facilement le cas, prenez une version d'image inexistante : par exemple 1.99

```bash
kubectl set image deployment/web-deployment nginx=nginx:1.99 --record
```

Suivez l'avancement :

```bash
kubectl rollout status deployment web-deployment
```

On voit que ça n'avance pas. Vous pouvez sortir avec un Ctrl-C

On constate aussi le problème avec un describe sur le deployement

```bash
kubectl describe deployment web-deployment
```

Ou encore sur les ressources :

```bash
kubectl get all
```

La solution reste la même => la marche arrière avec un undo :

```bash
kubectl rollout undo deployment web-deployment
```

Le TP est terminé.

