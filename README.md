# Travaux pratiques pour formation docker

## TP no 00

Accès à l'environnement du TP

### Consignes générales

Afin de pouvoir pratiquer sur kubernetes au travers des différents tps, nous vous avons préparé l'environnement suivant :
- [ ] un compte linux sur une VM isolée
- [ ] un cluster kubernetes de 2 noeuds

### Vérification de votre environnement

Vérifiez que votre environnement est fonctionnel :
- [ ] connectez-vous sur la VM avec les accès donnés par le formateur (avec putty, hostname, user, password)
- [ ] testez la commande CLI de kubernetes

```bash
kubectl version 
```

  Résultat attendu : la version du client ainsi que la version du cluster kubernetes
```bash
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.3", GitCommit:"2e7996e3e2712684bc73f0dec0200d64eec7fe40", GitTreeState:"clean", BuildDate:"2020-05-20T12:52:00Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"16+", GitVersion:"v1.16.8-eks-e16311", GitCommit:"e163110a04dcb2f39c3325af96d019b4925419eb", GitTreeState:"clean", BuildDate:"2020-03-27T22:37:12Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
```

- [ ] obtenez un aperçu des commandes disponibles

```bash
kubectl
```

- [ ] obtenez des informations sur votre cluster

```bash
kubectl cluster-info
```

- [ ] observez le nombre de noeuds

```bash
kubectl get nodes
```

- [ ] obtenez des informations sur les composants du master (control plane)

```bash
kubectl get componentstatuses
```

- [ ] observez tous les objets déjà créés (ou presque)

```bash
kubectl get all
```

- [ ] faites part de vos questions et observations à votre formateur

