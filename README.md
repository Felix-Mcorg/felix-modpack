# felix-modpack

Le modpack du serveur Felix : tous les fichiers qui modifient le jeu, et le `manifest.json` que lit le Felix Launcher.

## Contenu du dépôt
| Dossier / fichier | Rôle |
|---|---|
| `game/` | **Tout ce qui modifie le jeu**, toujours synchronisé chez les joueurs (voir ci-dessous) |
| `defaults/` | Fichiers **copiés une seule fois** (ex. `options.txt`, `servers.dat`) : le joueur peut ensuite les modifier |
| `launcher/` | Fichiers **du launcher** (pas du jeu) : logo, fond d'écran, thème, actualités |
| `pack.json` | Réglages : nom, version de Minecraft, loader, adresse du serveur, dossiers à importer, fichiers externes |
| `manifest.json` | Généré automatiquement (liste des fichiers avec SHA-256). Ne pas modifier à la main |
| `tools/generate_manifest.py` | Le script de génération |
| `.github/workflows/manifest.yml` | Régénère `manifest.json` à chaque modification |

### Dans `game/` (reproduit le dossier du jeu des joueurs)
```
game/
├─ mods/               les mods (.jar)
├─ config/             réglages des mods
├─ defaultconfigs/     réglages par défaut des nouveaux mondes
├─ kubejs/
│  ├─ startup_scripts/ server_scripts/ client_scripts/ assets/ data/
├─ shaderpacks/        shaders
├─ resourcepacks/      packs de ressources (logo de chargement, menus, textures...)
├─ scripts/            scripts CraftTweaker
└─ tacz/               add-ons (packs d'armes) du mod TaCZ
```
Un fichier posé **directement dans `game/`** (pas dans un sous-dossier) arrive à la **racine** du dossier du jeu, au même niveau que `options.txt`. C'est là que vont les fichiers de configuration qui ne sont pas dans `config/`.

Chaque dossier contient un `LISEZMOI.txt` (GitHub ne garde pas les dossiers vides). Ces fichiers ne sont **jamais** envoyés aux joueurs. Pour ajouter un autre dossier, crée-le dans `game/` (ou dans ton jeu local avant l'import) : rien à déclarer.

### Dans `launcher/` (prévu pour l'interface du launcher)
`logo.png`, `background.png`, `theme.json`, `news.json`. Ils apparaissent dans `manifest.json` sous `launcherAssets` ; le launcher les garde dans son propre dossier (pas dans le dossier du jeu).

## Mettre à jour le modpack
**Depuis ton dossier de jeu local** (le plus simple, comme pour ajouter un mod) :
```
python tools/generate_manifest.py --import-from "C:\Users\toi\Documents\FelixLauncher\felix-factory"
```
Le script copie en miroir **tous les dossiers** de ton dossier de jeu vers `game/` (`mods`, `config`, `kubejs`, `tacz`... et même un nouveau dossier d'add-on, sans rien déclarer), puis écrit `manifest.json`. Il affiche ce qu'il a copié et ce qu'il a ignoré.
- **Jamais envoyés** (exclus par défaut) : `saves`, `logs`, `crash-reports`, `screenshots`, `versions`, `libraries`, `assets`, `runtime`, `backups`, `cache`, `debug`, `downloads`, `natives`, `webcache`, `local`, `.felix`, `.git`...
- **Exclure d'autres dossiers** (ex. des cartes personnelles) : ajoute leurs noms dans `pack.json` > `"exclude"` (ex. `["journeymap", "xaero"]`).
- **Fichiers isolés à la racine du jeu** : liste-les dans `pack.json` > `"includeFiles"` (ex. `["tac.cfg"]`).
- **N'importer que certains dossiers** : remplace `"include": ["*"]` par une liste précise (ex. `["mods", "config"]`).
- Un dossier ou un fichier supprimé de ton jeu local est aussi retiré de `game/` (et donc chez les joueurs au lancement suivant).
- Garde-fou : si le dossier indiqué ne ressemble pas à un dossier de jeu (aucun sous-dossier), le script refuse et ne supprime rien.

Ensuite, envoie le tout sur GitHub (GitHub Desktop, ou `git add . && git commit && git push`).

**Ou à la main** : ajoute / retire des fichiers dans `game/`, `defaults/` ou `launcher/`, envoie sur GitHub : l'action automatique met `manifest.json` à jour.

## Fichiers hébergés ailleurs (mods volumineux ou licences)
Dans `pack.json`, ajoute dans `"external"` :
```json
{ "path": "mods/exemple.jar", "url": "https://cdn.modrinth.com/...", "sha256": "...", "size": 123456 }
```
GitHub refuse les fichiers de plus de 100 Mo : utilise cette méthode pour eux.

## Partager ce serveur (le hub du Felix Launcher)
Pour qu'un joueur ajoute ce serveur dans son Felix Launcher : **＋ Ajouter un serveur**, puis coller l'un de ces liens :
- le lien du dépôt : `https://github.com/Felix-Mcorg/felix-modpack`
- ou celui du manifest : `https://raw.githubusercontent.com/Felix-Mcorg/felix-modpack/main/manifest.json`

N'importe qui peut créer son propre modpack de la même façon : copier ce dépôt (modèle), changer `pack.json` (nom, version de Minecraft, serveur), ajouter ses fichiers dans `game/`, lancer le script, puis partager le lien de son dépôt.

Le serveur Felix Factory est déjà dans la liste au premier lancement (adresse réglée dans `felix-launcher/src/Felix.Launcher/AppConfig.cs`).

## Règles
- Dépôt public : aucun secret ici.
- Vérifie la licence des mods avant de les publier.
