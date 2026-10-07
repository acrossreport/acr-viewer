# ACR Viewer

[English](README.md) | [日本語](README.ja.md) | Français

ACR Viewer est une application résidant dans la barre d'état système, qui imprime et enregistre automatiquement (PDF/PNG) les données de rapports ACR (AcrossReport) reçues via un dossier surveillé ou par HTTP.

## Fonctionnalités

- Réside dans la barre d'état système (aucune fenêtre nécessaire)
- **Dossier surveillé** : déposez simplement un fichier de données pour le rendu, l'impression et l'enregistrement automatiques
- **API HTTP** : envoi de données de rapport depuis d'autres systèmes pour impression et enregistrement
- Sortie : PDF / PNG / les deux, et impression automatique
- Paramètres modifiables dans une page de réglages accessible depuis le navigateur (japonais / English / Français)
- Suppression automatique des données reçues et des fichiers de sortie après une durée de conservation

## Configuration requise

| OS | Statut |
|---|---|
| Windows x64 | Pris en charge |
| Linux x64 | Pris en charge |
| macOS (Apple Silicon) | Prévu |
| macOS (Intel) | Prévu |

### Windows

- Versions de Windows prises en charge : Windows 11 ou version ultérieure
- L'impression automatique nécessite une application capable d'ouvrir les PDF (application PDF par défaut). L'impression se fait sur l'imprimante par défaut de l'OS

### Linux

- Pris en charge : Ubuntu 24.04 ou version ultérieure (x64)
- Les bibliothèques suivantes sont nécessaires. Si elles ne sont pas installées, installez-les avec :

```
sudo apt install libgtk-3-0 libxdo3 libayatana-appindicator3-1
```

- L'impression utilise le système d'impression de l'OS (CUPS, commande `lp`). Si aucune imprimante par défaut n'est définie dans l'OS, saisissez le nom de l'imprimante dans « Nom de l'imprimante » de la page de réglages. Vous pouvez vérifier les noms des imprimantes enregistrées avec `lpstat -p`
- L'icône apparaît dans la barre supérieure de l'écran. Sur les bureaux GNOME autres qu'Ubuntu, l'extension AppIndicator est nécessaire. Même si l'icône n'apparaît pas, ACR Viewer fonctionne et vous pouvez ouvrir la page de réglages à l'adresse `http://localhost:8765/` dans le navigateur

## Téléchargement

Téléchargez le fichier correspondant à votre OS depuis les [Releases](https://github.com/acrossreport/acr-viewer/releases).

- Windows x64 : `acr_viewer-v0.1.0-win-x64.zip`
- Linux x64 : `acr_viewer-v0.1.0-linux-x64.zip`

## Installation et lancement

### Windows

1. Extrayez le fichier zip téléchargé dans le dossier de votre choix
2. Lancez `acr_viewer.exe`. Une icône apparaît dans la barre d'état système
3. Choisissez « Ouvrir les paramètres Web » dans le menu de l'icône pour ouvrir la page de réglages dans le navigateur (`http://localhost:8765/`)

### Linux

1. Extrayez le fichier zip téléchargé dans le dossier de votre choix
2. Dans un terminal, placez-vous dans ce dossier et lancez `./acr_viewer`. Une icône apparaît dans la barre supérieure de l'écran
3. Cliquez sur l'icône et choisissez « Ouvrir les paramètres Web » dans le menu pour ouvrir la page de réglages dans le navigateur (`http://localhost:8765/`)

### Dossiers

Au premier lancement, les dossiers suivants sont créés automatiquement sous « Documents » (Linux : `~/Documents`). Ils sont modifiables dans la page de réglages.

| Dossier | Rôle |
|---|---|
| `AcrViewer/Watch` | Dossier surveillé (déposez les fichiers ici) |
| `AcrViewer/Templates` | Dossier des définitions |
| `AcrViewer/Output` | Sortie PDF / PNG |
| `AcrViewer/Processed` | Fichiers traités |
| `AcrViewer/Error` | Données n'ayant pas pu être traitées |

## Menu de la barre d'état système

| Élément du menu | Description |
|---|---|
| Impression automatique | Active/désactive l'impression automatique |
| Enregistrer en PDF | Active/désactive l'enregistrement en PDF |
| Ouvrir les paramètres Web (dossiers, imprimante, port) | Ouvre la page de réglages dans le navigateur |
| Quitter | Quitte AcrViewer |

La langue d'affichage suit celle choisie dans la page de réglages (日本語 / English / Français). Le changement s'applique après redémarrage d'AcrViewer.

## Utilisation

### Méthode 1 : indiquer la définition dans le fichier de données (recommandée)

1. Placez le fichier de définition (ex. `invoice.json`) dans le **dossier des définitions**
2. Indiquez le nom du fichier de définition dans l'en-tête (`Parameters`) du fichier de données

```json
{
  "Parameters": {
    "TemplateFile": "invoice.json",
    "...": "...",
    "Data": [ ... ]
  }
}
```

3. Déposez ces données dans le **dossier surveillé** sous le nom `nom.data.json` ; le rendu, l'impression et l'enregistrement sont automatiques
4. Les données traitées sont déplacées vers le dossier des fichiers traités ; celles dont la définition est introuvable ou dont le rendu échoue sont déplacées vers le dossier d'erreur

N'utilisez pas `TemplateFile` comme nom de champ du rapport (nom réservé).

### Méthode 2 : déposer la définition et les données par paire

Déposez une paire portant le même nom dans le dossier surveillé. Elle est traitée dès que les deux fichiers sont présents.

```
nom.template.json   … définition
nom.data.json       … données
```

Si un `.template.json` du même nom se trouve dans le dossier surveillé, la méthode 2 est prioritaire.

### Nom du fichier de sortie

```
yyyymmddhhmmss_nomdefinition.pdf   … PDF
yyyymmddhhmmss_nomdefinition.zip   … PNG (toutes les pages dans un seul ZIP)
```

La sortie PNG est enregistrée dans un seul fichier ZIP au format ACR-PNG-PACKAGE (identique à ACR CLI). Le ZIP contient `manifest.json` et `pages/001.png`, `pages/002.png`, etc.

### API HTTP

- `POST /api/print` : envoyez `{ "template": {...}, "data": {...}, "design_name": "..." }` pour le rendu, l'enregistrement et l'impression
  - Si `template` est omis, la définition est lue dans le dossier des définitions à partir de `Parameters.TemplateFile` dans `data`

Le serveur HTTP écoute sur `0.0.0.0`. Utilisez-le dans un environnement non accessible depuis l'extérieur.

## Diagnostic (journaux)

Les journaux ne sont pas produits par défaut. Pour les activer lors d'un diagnostic, quittez ACR Viewer, remplacez `"debug_log": false` par `"debug_log": true` dans le fichier de réglages ci-dessous, puis relancez ACR Viewer. Sous Linux, lancez-le depuis un terminal ; les journaux s'affichent dans ce terminal. Une fois le diagnostic terminé, remettez la valeur à `false`.

| OS | Fichier de réglages |
|---|---|
| Windows | `%APPDATA%\AcrViewer\config.json` |
| Linux | `~/.config/AcrViewer/config.json` |

Si `"debug_log"` n'existe pas dans le fichier, ajoutez-le.

## À propos de la sortie

Si la licence n'est pas enregistrée, la sortie (PDF, PNG, impression) comporte un filigrane. Enregistrez votre adresse e-mail et votre clé de licence dans « Enregistrement de la licence » de la page de réglages pour obtenir une sortie sans filigrane. Consultez le [site officiel](https://acrossreport.com) pour plus de détails.

## Liens

- ACR Designer : https://github.com/acrossreport/acr-designer
- ACR Generator : https://github.com/acrossreport/acr-generator
- Spécification ACR (modèle JSON) : https://github.com/acrossreport/acr-spec
- Site officiel : https://acrossreport.com

## Licence

Le code source de ce logiciel n'est pas public. Veuillez consulter le fichier [LICENSE](LICENSE) pour les conditions d'utilisation.

## Contact

across.support@gmail.com

---

© Across Systems Corporation
L'architecture d'instructions de dessin intermédiaires d'ACR fait l'objet d'une demande de brevet.
