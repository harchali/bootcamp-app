### Bootcamp app

This app is composed of 11 microservices. We will use it to deploy a full CI/CD pipeline in a cloud environment, using DevOps tools.

### DevOps 

#### CI

Le but de notre chaîne de CI est de produire des images de conteneur, et de les pousser vers un registre d'image (ici Artifact Registry).

##### 1. Création d'un registre Docker

Commencez par activer le service Artifact Registry sur votre projet en suivant ce lien : https://console.cloud.google.com/flows/enableapi?apiid=artifactregistry.googleapis.com

Nous devons ensuite créer le dépôt qui hebergera nos images, utilisez la commande gcloud suivante (ou via l'interface [ici](https://cloud.google.com/artifact-registry/docs/docker/store-docker-container-images?hl=fr#create]) ) : 

```
gcloud artifacts repositories create bootcamp --repository-format=docker \
--location=europe-west1 --description="Repository for bootcamp app"
```

Vous devrez ensuite authentifier votre Docker CLI locale afin de pouvoir push et pull des images. Utilisez la commande suivante :

```
gcloud auth configure-docker europe-west1-docker.pkg.dev
```

Votre registre est maintenant configuré, vous pourrez l'utiliser comme par exemple `docker push europe-west1-docker.pkg.dev/<votre-id-projet-gcp>/<votre>/<image>:<tag>`

##### 2. Build d'images avec Google Cloud Build

Maintenant que nous avons un dépôt permettant de stocker nos images Docker nous allons automatiser les actions de CI, afin que chaque commit sur la branche master soit automatiquement transformé en image Docker et pushé sur le dépot.

Tout d'abord, activez l'API via ce lien : https://console.cloud.google.com/flows/enableapi?apiid=cloudbuild.googleapis.com

Ensuite, ajoutez le fichier `cloudbuild.yaml` à la racine de votre dépot. Ce fichier indique à Cloud Build les étapes à réaliser (ici, build votre image et push dans Artifact Registry).

```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'europe-west1-docker.pkg.dev/$PROJECT_ID/bootcamp/benjamin/adservice:$SHORT_SHA', 'src/adservice' ]
images:
- 'europe-west1-docker.pkg.dev/$PROJECT_ID/bootcamp/benjamin/adservice:$SHORT_SHA'
```

Vous devez ensuite autoriser Google Cloud Build à accédern à votre dépot de code source, pour cela suivez les instructions ici : https://cloud.google.com/build/docs/automating-builds/create-manage-triggers#connect_repo

Ajoutez ensuite un déclencheur (trigger) qui déclenchera automatiquement un build à chaque commit sur master (via la doc ici https://cloud.google.com/build/docs/automating-builds/create-manage-triggers#build_trigger)
Les optios importantes sont :
* Region : europe-west1
* Event : Push to a branch
* Branch : `^master$`
* Configuration > Type : Autodected
* Configuration > Location : Repository

Une fois la configuration terminée, faites un commit et push sur votre dépot de code source. Vous devriez voir apparaitre une nouvelle image dans Artifact Registry (attendez quelques minutes quand même...).

#### CD avec ArgoCD

Afin de déployer nos images dans notre cluster, nous devons utiliser un outil de Déploiement Continu (CD).
Dans ce bootcamp, nous allons nous concentrer sur ArgoCD.

Nous allons déployer ArgoCD grâce à Helm (sorte de gestionnaire de paquets pour Kubernetes).
Installez tout d'abord la CLI Helm en suivant cette doc https://helm.sh/docs/intro/install/

Une fois la commande `helm` installée, vous pouvez déployer ArgoCD via la commande suivante :

**ATTENTION : Assurez vous que Kubectl pointe bien vers le bon cluster !!**
```
helm repo add argo https://argoproj.github.io/argo-helm
helm install -n argocd argocd argo/argo-cd 
```

Une fois la commande exécutée, helm vous indique la commande pour récupérer le mot de passe admin, ainsi que la commande pour accéder à l'UI.
Essayez de vous connecter à l'interface à l'aide de ces deux informations.

Nous allons ensuite donner accès à ArgoCD à votre dépôt de code source. 
Ajoutez un dossier `argo/` dans votre dépot, il contiendra les fichiers Kubernetes propres à ArgoCD.

Dans ce dossier, ajoutez le fichier `repository.yaml` avec le contenu suivant (sans oublier de remplacer `url` et `sshPrivateKey`) :

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bootcamp-repository
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: git@github.com:myaccount/myrepository
  sshPrivateKey: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    ...
    -----END OPENSSH PRIVATE KEY-----
```

Enfin, pour déclarer votre application auprès de ArgoCD, ajoutez le fichier `bootcamp-app.yaml` suivant :

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bootcamp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: git@github.com:myaccount/myrepository
    targetRevision: HEAD
    path: kubernetes-manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: bootcamp
```

Utilisez `kubectl apply -f argo/` afin d'initialiser le déploiement de l'application.

Félicitation, le déploiement continu est en place !

#### Mettre à jour automatiquement le YAML à chaque nouvelle image

Nous avons une chaine de CD qui publie des images automatiquement, ainsi qu'une chaine de CD qui déploie automatiquement le YAML dans le cluster.
Il nous reste néanmoins un élément à automatiser : mettre à jour le YAML lorsqu'une nouvelle image est publiée.

Pour cela, nous allons utiliser un plugin d'ArgoCD : Argo Image Updater (AIU).

Installez AIU dans le cluster grâce à la commande suivante :

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```

Nous devons ensuite paramétrer AIU via des annotations sur l'objet Application qui nous intéresse.
Ouvrez le fichier `argo/bootcamp-app.yaml` et ajoutez les informations suivantes :

```yaml
...
metadata:
  annotations:
    argocd-image-updater.argoproj.io/image-list: adservice=europe-west1-docker.pkg.dev/<votre-projet>/<votre>/<image>
    argocd-image-updater.argoproj.io/adservice.update-strategy: latest
    argocd-image-updater.argoproj.io/adservice.allow-tags: regexp:^[0-9a-f]{7}$
    argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd/bootcamp-repository
    argocd-image-updater.argoproj.io/git-branch: main
...
```

Utilisez `kubectl apply -f argo/` afin de prendre en compte les modifications.

Enfin, nous devons faire quelques modifications à notre dossier `kubernetes-manifest`. Dans ce dossier, ajoutez un fichier `kustomization.yaml` avec 
le contenu suivant :

```yaml
resources:
  - adservice.yaml
  # Ajoutez tous les fichiers yaml que vous souhaitez apply
```

Commitez le changement, AIU devrait désormais mettre à jour vos fichiers YAML en commitant directement dans votre dépôt.