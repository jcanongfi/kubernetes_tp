# Travaux pratiques pour formation kubernetes

## TP no 03

Assurer le service !

### Nous voulons rendre notre serveur web accessible

Pour l'instant, notre pod existe dans l'orchestrateur, mais n'est pas accessible.
Il dispose d'une adresse IP privée affectée de façon aléatoire et est démarré sur un noeud quelconque.

Nous avons vu qu'il existe des ressource de type Service qui permettent d'exposer un ou plusieurs pod identifiés par un sélecteur.

#### Ajout d'un label

Problème : notre pod n'a pas de label !

Soit vous disposez du fichier yaml d'origine (c'est bien!)
Soit vous le regénérez via la commande suivante :

```bash
kubectl run webserver --image=nginx --dry-run=client -o yaml >webserver-pod.yaml
```

Ajoutez un (ou plusieurs) label(s) de votre choix dans les metadata de votre pod :

```yaml
metadata:
  labels:
    app: web
```

Et **appliquez** la modification directement sur votre pod (évite la destruction/recréation) :

```bash
kubectl apply -f webserver-pod.yaml
```

Vous devriez obtenir le message "pod/webserver configured"
Vous pouvez (devriez) vérifier via une commande "get" (j'arrête de vous donnez les commandes, je vous laisse commencer à prendre votre autonomie)

#### Création du service 

Nous allons gagner du temps, voici la description du service que nous allons créer :
* Service de type LoadBalancer (géré par le cloud, ici AWS)
* Selectionne tous les pods ayant le label app: web
* Ecoute sur le port 80, redirige sur le port 80 des containers ciblés (containerPort)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webserver-service
  labels:
    info: un_label_pour_webserver-service
spec:
  type: LoadBalancer
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: web
```

Le fichier existe dans le répertoire de ce TP (webserver-service.yaml)
Vous pourriez ajouter "namespace: <namespace>" dans la partie metadata, mais l'apiserver le fera pour vous.

```bash
kubectl create -f webserver-service.yaml
```

Le service est créé. Observons ses propriétés :

```bash
kubectl describe service webserver-service
```

* On note que le Namespace est bien renseigné.
* On note que le Selector correspond à notre demande.
* On vérifie via Endpoints qu'il a bien ciblé un seul pod
  * Ce qui permet de préciser qu'il se restreint au namespace dans sa recherche, sinon il aurait trouvé les pods des collègues (sic!)
  * On pourrait (devrait) vérifier que l'adresse IP est bien celle de notre pod. N'hésitez pas à le faire.
* On trouve une adresse IP privée de service, qui ne nous intéresse pas, mais attibuée sur le service_cidr
* On trouve de façon plus intéressante l'adresse publique du loadbalancer avec **LoadBalancer Ingress**

Tentez d'accéder à votre service (le provisionnement de l'ELB peut prendre un peu de temps) :

```bash
http://xxxxxxxxxxxxxxxxxxxxxxx-xxxxxxxxxx.eu-west-3.elb.amazonaws.com
```

C'est tout pour ce TP !  :clap:

