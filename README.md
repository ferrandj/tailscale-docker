# Tailscale pour NAS (Docker Compose)

Ce dépôt contient un `docker-compose.tailscale.yml` qui connecte un NAS (ou tout autre
hôte Docker) à un réseau Tailscale (tailnet). **Aucun secret n'est dans ce fichier** :
la clé d'authentification se met dans un fichier `.env` séparé, qui ne doit jamais être
commité.

## Avant de démarrer

1. **Génère une clé d'authentification Tailscale**
   - Va sur la [console Tailscale](https://login.tailscale.com/admin) -> *Settings* -> *Keys*
     -> *Generate auth key*.
   - Laisse **Ephemeral décoché** (sinon la machine disparaît du tailnet à chaque arrêt).
   - Copie la clé (`tskey-auth-...`) : elle ne s'affiche qu'une seule fois.

2. **Crée le fichier `.env`** à côté de `docker-compose.tailscale.yml` (jamais dans le
   compose lui-même, et jamais commité) :

   ```
   TS_AUTHKEY=tskey-auth-xxxxxxxx
   TS_HOSTNAME=nas
   ```

   | Variable      | Obligatoire | Description                                                                 |
   |---------------|-------------|-------------------------------------------------------------------------------|
   | `TS_AUTHKEY`  | Oui         | La clé générée à l'étape 1 (`tskey-auth-...`).                               |
   | `TS_HOSTNAME` | Non         | Le nom que la machine aura dans ton tailnet. Par défaut : `nas`.             |

3. **Vérifie le support de `/dev/net/tun`** sur ton NAS (nécessaire pour le mode
   « noyau » utilisé par défaut). S'il n'existe pas, voir la section
   [Mode userspace](#mode-userspace-si-devnettun-nexiste-pas) ci-dessous.

## Déploiement

### Synology (Container Manager)

1. Crée le dossier `/volume1/docker/tailscale`.
2. Mets-y `docker-compose.tailscale.yml` et ton fichier `.env`.
3. Dans Container Manager : *Projet* -> *Créer* -> choisis ce dossier.

Ou en SSH :

```bash
cd /volume1/docker/tailscale
docker compose -f docker-compose.tailscale.yml up -d
```

### Mise à jour

```bash
docker compose -f docker-compose.tailscale.yml pull
docker compose -f docker-compose.tailscale.yml up -d
```

## Après le premier démarrage

Dans la [console Tailscale](https://login.tailscale.com/admin), onglet *Machines*,
menu « ... » de la machine -> **Disable key expiry**. Sinon, la machine sera
déconnectée du tailnet quand sa clé expirera.

## Mode userspace (si `/dev/net/tun` n'existe pas)

Si ton NAS n'a pas le périphérique `/dev/net/tun`, modifie
`docker-compose.tailscale.yml` :

- passe `TS_USERSPACE` à `"true"` ;
- commente (ou supprime) les blocs `devices:` et `cap_add:`.

## Partager le réseau local (optionnel)

Pour joindre aussi tes autres appareils de la maison depuis l'extérieur, décommente
la ligne `TS_ROUTES` dans le compose (adapte le sous-réseau, ex. `192.168.1.0/24`),
puis active la route dans la console Tailscale (*Machines* -> *Edit route settings*).

## Sécurité

- `docker-compose.tailscale.yml` ne contient **aucun secret** : il référence
  uniquement des variables d'environnement.
- Le fichier `.env` contient la clé d'authentification : garde-le hors de tout
  dépôt Git (ajoute-le à `.gitignore` si ce n'est pas déjà fait) et ne le partage
  jamais.
