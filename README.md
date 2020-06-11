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

