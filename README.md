# Unikalo MongoDB Kubernetes Deployment

Ce repository contient les configurations pour déployer MongoDB pour Unikalo en utilisant Helm et ArgoCD.

## Structure du projet

```
├── argocd/
│   └── app.yaml              # Configuration ArgoCD Application
├── helm/
│   └── mongodb/              # Chart Helm personnalisé
│       ├── Chart.yaml        # Métadonnées du chart
│       ├── values.yaml       # Valeurs par défaut
│       ├── values-unikalo.yaml # Valeurs spécifiques à unikalo
│       └── templates/        # Templates Kubernetes
└── README.md
```

## Fonctionnalités

- **Image officielle MongoDB** : Utilise l'image `mongo:7.0` officielle
- **Persistence** : Stockage persistant avec PVC
- **Authentification** : Support des secrets Kubernetes
- **Configuration** : ConfigMap pour la configuration MongoDB
- **Monitoring** : Probes de santé configurées
- **ArgoCD** : Déploiement automatisé avec GitOps

## Prérequis

1. **Secret existant** : Créer le secret `unikalo-mongodb-secret` avec la clé :
   ```bash
   kubectl create secret generic unikalo-mongodb-secret \
     --from-literal=mongodb-root-password=<root-password> \
     -n koden-clients
   ```
   Note: Avec l'image officielle MongoDB, nous utilisons le compte root. Pour créer des utilisateurs supplémentaires, connectez-vous après le déploiement.

2. **StorageClass** : Vérifier que `csi-cinder-high-speed` existe

## Déploiement

### Avec ArgoCD (recommandé)

1. Appliquer la configuration ArgoCD :
   ```bash
   kubectl apply -f argocd/app.yaml
   ```

2. ArgoCD synchronisera automatiquement les changements du repository

### Avec Helm (manuel)

```bash
# Installer/mettre à jour
helm upgrade --install unikalo-mongodb helm/mongodb \
  -f helm/mongodb/values-unikalo.yaml \
  -n koden-clients \
  --create-namespace

# Désinstaller
helm uninstall unikalo-mongodb -n koden-clients
```

## Configuration

### Variables principales (values-unikalo.yaml)

- `image.tag`: Version de MongoDB (défaut: 7.0)
- `auth.existingSecret`: Nom du secret contenant les mots de passe
- `persistence.size`: Taille du volume (défaut: 1Gi)
- `resources`: Limites CPU/mémoire

### Connexion à MongoDB

```bash
# Port-forward pour accès local
kubectl port-forward svc/unikalo-mongodb 27017:27017 -n koden-clients

# Connexion avec mongosh (nouveau client MongoDB)
mongosh mongodb://root:<root-password>@localhost:27017/meow
```

## Avantages de cette approche

1. **Simplicité** : Chart Helm minimaliste mais complet
2. **Image officielle** : Pas de dépendance à Bitnami
3. **Bonnes pratiques** : 
   - Séparation des préoccupations (Helm vs ArgoCD)
   - Gestion des secrets appropriée
   - Configuration modulaire
4. **GitOps** : Déploiement automatisé avec ArgoCD
5. **Maintenance** : Structure claire et documentée

persistence:
  enabled: true
  size: 1Gi
  storageClass: "csi-cinder-high-speed"
```

Adaptez les valeurs de `size` et `storageClass` selon vos besoins.

Vous pouvez également modifier l'image Docker utilisée en changeant les valeurs `image.registry`, `image.repository` et `image.tag`.

## Déploiement de MongoDB

Le déploiement peut être réalisé en utilisant le script `install.sh`.

Ce script va :
- créer le secret `unikalo-mongodb-secret` contenant les identifiants de connexion. (Pensez à modifier ce nom sur le déploiement Meow si besoin)
- déployer MongoDB en utilisant ArgoCD