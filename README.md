# lab-devops — Console de surveillance DevOps

Petite application web Django utilisée comme support d'un lab DevOps : conteneurisation avec Docker, déploiement sur Vercel, suivi d'erreurs avec Sentry et test de blocage (« kill switch ») destiné à une chaîne d'intégration continue.

Démo en ligne : <https://lab-devops-delta.vercel.app>

## Fonctionnalités

| Route | Description |
|---|---|
| `/` | Page « Console de Surveillance DevOps » avec le statut du système |
| `/sentry-debug/` | Déclenche volontairement une division par zéro, envoie l'exception à Sentry sans afficher de page 500, puis réaffiche la page avec un message d'erreur discret |
| `/admin/` | Interface d'administration Django |

## Stack technique

- Python 3.12 et Django 6.0
- Sentry (`sentry-sdk`) pour le suivi des erreurs
- SQLite (base par défaut de Django)
- Docker et Docker Compose
- Déploiement : Vercel

## Structure du projet

| Élément | Description |
|---|---|
| `core/` | Configuration du projet Django (`settings.py`, `urls.py`, `wsgi.py`, `asgi.py`) |
| `portal/` | Application du portail : vues, template `index.html`, tests |
| `manage.py` | Point d'entrée des commandes Django |
| `requirements.txt` | Dépendances Python |
| `Dockerfile` | Image `python:3.12-slim`, port 8000 exposé |
| `docker-compose.yml` | Service `web` (build local, port 8000, code monté en volume, `DEBUG=1`) |

## Prérequis

- Docker et Docker Compose
- (Sans Docker) Python 3.12 ou plus récent

## Lancement avec Docker

```bash
git clone https://github.com/ibralearn/lab-devops.git
cd lab-devops
docker compose up --build
```

L'application est disponible sur <http://localhost:8000>.

Pour arrêter : `docker compose down`.

## Lancement sans Docker

```bash
python -m venv .venv
source .venv/bin/activate        # Windows : .venv\Scripts\activate
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

## Tests

```bash
python manage.py test
```

Le test `KillSwitchTest` (`portal/tests.py`) vérifie que la page d'accueil répond en 200 et **ne contient pas** le mot « Attention ». S'il apparaît, le test échoue : l'idée est de bloquer un déploiement depuis une chaîne CI (par exemple GitHub Actions) tant que ce texte est présent. Aucun workflow n'est encore défini dans ce dépôt.

## Notes de configuration

- `ALLOWED_HOSTS` ne contient que `lab-devops-delta.vercel.app`. Pour un lancement local, ajouter `localhost` et `127.0.0.1` dans `core/settings.py`.
- Le conteneur lance `python manage.py runserver 0.0.0.0:8000`, le serveur de développement. Il convient pour le lab, pas pour la production (utiliser Gunicorn ou équivalent).
- `DEBUG`, `SECRET_KEY` et le DSN Sentry sont écrits en dur dans `core/settings.py`. Avant un usage réel : passer `DEBUG` à `False`, régénérer la clé secrète et lire ces valeurs depuis des variables d'environnement.
- Le fichier `5.1` à la racine est une sortie de `pip install` enregistrée par erreur ; il peut être supprimé.

## Auteur

[ibralearn](https://github.com/ibralearn)
