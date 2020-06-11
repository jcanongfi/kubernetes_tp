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

Observons maintenant le fichier web-deployment.yaml

```yaml
```




kubectl rollout status deployment.v1.apps/nginx-deployment


kubectl rollout history deployment/frontend                      # Vérifie l'historique de déploiements incluant la révision
kubectl rollout undo deployment/frontend                         # Rollback du déploiement précédent
kubectl rollout undo deployment/frontend --to-revision=2         # Rollback à une version spécifique
kubectl rollout status -w deployment/frontend                    # Écoute (Watch) le status du rolling update du déploiement "frontend" jusqu'à ce qu'il se termine
kubectl rollout restart deployment/frontend      
