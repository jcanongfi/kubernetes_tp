# Travaux pratiques pour formation docker

## TP no 04

Ajoutons de l'élasticité

### Que se passe-t-il si mon pod meurt ?

Tout d'abord, avec un pod simple tel que nous l'avons créé, il y a déjà un minimum de résilience.

Faites le test suivant :
* Executez un shell bash dans le container nginx du pod webserver
* Tuez le process 1

Vérifier l'état du pod webserver :
```bash
kubectl get pod webserver
```

Est-il toujours présent ?
Qu'indique la colonne STATUS ?
Qu'indique la colonne RESTART ?

Cela peut prendre un peu de temps, mais on constate que le container est bien mort, mais qu'il est relancé.
Ceci autant de fois qu'on le tue.

La réponse est obtenue par l'analyse du pod :

```bash
kubectl get pod webserver -o yaml
```

En effet, il existe une stratégie de redémarrage, et sa valeur par défaut est Always.

```bash
 restartPolicy: Always
```


### Mon site a du succès, je veux plusieurs pods

Actuellement, nous avons donc un pod (plutôt robuste) qui offre un serveur web au travers d'un Service de type LoadBalancer.

Puisque tout fonctionne bien, il gagne en notoriété et il a du mal seul à gérer la charge.

Nous souhaitons augmenter le nombre de container de façon simple (eviter de lancer manuellement x pod via kubectl).

Pour cela, il existe une resource prévue pour gérer autant de réplique de notre pod : le replicaset

Le TP contient un fichier web-replicaset.yaml qui décrit un replicaset
* apiVersion est positionné à apps/v1
* metadata est propre à cette ressource de type ReplicaSet
* spec contient :
  * replica qui permet de fixer le nombre de pod souhaité
  * selector qui permet de choisir les pods considérés
  * template qui précise le modèle pour les pods à créer (reprend exactement les metadata et spec d'une resource de type Pod)

Il y a donc 2 rubriques metadata et spec à des niveaux d'indentation différents. C'est normal.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
  labels:
    truc: labeldureplicaset
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
      - name: webserver
        image: nginx:1.18
```

Créez cette ressource :

```bash
kubectl create -f web-replicaset.yaml
```

Observez les pods créés :

```bash
kubectl get all
```

Avec la commande 'get all', vous voyez le replicaset ainsi que le service.

Concernant les pods, si vous avez suivi le TP, vous pouvez vérifier qu'il y en a bien 3.
Cependant, il y a votre ancien pod webserver et deux nouveaux nommé à partir du nom du replicaset web-rs-xxxxx

Effectivement, puisque votre pod webserver était présent précédemment et qu'il vérifie la condition de label, il est pris en compte.
Le replicaset n'a créé que 2 pod à partir du template.
Gardez ce phénomène à l'esprit. En effet, il se trouve que notre pod webserver est identique en tout point, mais ça aurait pu être un pod tout autre, venant parasiter notre service.

Détruisons ce pod (et éventuellement un ou plusieurs autres) :

```bash
kubectl delete pod webserver
```

Observez qu'il y a bien toujours 3 pods (des **nouveaux** venant remplacer ceux qui viennent à disparaître) :

```bash
kubectl get pods
```

Maintenant observez à nouveau le service, et en particulier la liste des Endpoints:

```bash
kubectl describe service webserver-service
```

Les 3 pods font bien partie du service.

Enfin, décidons d'augmenter (ou de diminuer) la capacité du replicaset

Soit vous modifiez le contenu du fichier et appliquez celui-ci à nouveau :

```bash
kubectl apply -f web-replicaset.yaml
```

Soit vous pouvez le faire dynamiquement (mais n'oubliez pas que le fichier yaml ne représentera plus la réalité...):

```bash
kubectl scale --replicas=4 replicaset web-rs
```

Encore un TP de terminé !  :star:




