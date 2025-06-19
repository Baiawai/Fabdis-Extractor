# 🏗️ Fabdis Extractor – Python

Un extracteur modulaire pour lire les fichiers Fab-Dis (catalogues produits normalisés) dans divers formats (`.xlsx`, `.csv`, `.xml`) et versions (`2.0`, `2.1`, `2.2`...), avec sortie JSON normalisée.

---

## ✅ Objectif

* Lire un **fichier Fab-Dis** fourni par un fabricant ou distributeur.
* Identifier **automatiquement** sa version (2.1, 2.2, etc.).
* Extraire les **produits** (référence, désignation, marque, prix, etc.).
* Générer un **JSON uniforme**, prêt pour intégration Laravel/API.

---
 
## 📂 Formats supportés

| Format | Extensions | Méthode                               |
| ------ | ---------- | ------------------------------------- |
| Excel  | `.xlsx`    | via `openpyxl` ou `pandas`            |
| CSV    | `.csv`     | via `csv.DictReader`                  |
| XML    | `.xml`     | via `xml.etree.ElementTree` ou `lxml` |

---

## 🔀 Versions supportées

| Version  | Type        | Description technique                                                                           |
| -------- | ----------- | ----------------------------------------------------------------------------------------------- |
| `2.0`    | Standard    | Ancienne version partielle. Champs peu structurés, souvent sans onglets explicites.             |
| `2.1`    | Standard    | Format tabulaire simplifié, colonnes principales : `Reference`, `Designation`, `PrixHT`.        |
| `2.2`    | Standard    | Structure multi-onglets (`Produits`, `Tarifs`, `Marques`...), meilleure séparation des données. |
| `3.x`    | XML + XSD   | Format XML strict avec schéma XSD. Utilisé dans les systèmes modernes (EDI/API).                |
| `custom` | Fournisseur | Variantes spécifiques (ex : Trenois, CEDEO, Rexel) avec noms d'onglets ou colonnes différents.  |

> ⚠️ Les versions `custom` nécessitent l'utilisation du moteur de règles (`RuleEngine`) pour réécrire dynamiquement les noms de champs, onglets, ou corriger les écarts structurels.

---

## 🔧 Architecture

Le projet suit une architecture modulaire et extensible en 4 couches principales :

### 1️⃣ `detectors/` – Détection du format et de la version

Chaque détecteur hérite d'une interface `BaseDetector` et se charge d’analyser un fichier brut pour identifier sa **version Fab-Dis** et son **type de format** :

```
detectors/
├── base.py            # Interface commune `BaseDetector`
├── manager.py         # `DetectorManager` pour orchestrer le processus
├── xlsx_detector.py   # Détection des fichiers .xlsx
├── xml_detector.py    # Détection des fichiers .xml
└── csv_detector.py    # Détection des fichiers .csv
```

> Le `DetectorManager` choisit dynamiquement le détecteur selon l'extension et délègue la détection de version.

---

### 2️⃣ `rules/` – Moteur de réécriture (`RuleEngine`)

Permet de corriger ou adapter dynamiquement :

* Les noms d'onglets (`ArticlesFabdis` → `Produits`)
* Les noms de colonnes (`libelle`, `nom` → `name`)
* Les valeurs mal formatées (prix, références, etc.)

```
rules/
├── engine.py          # Moteur central
├── presets/
│   ├── default.yaml   # Règles générales
│   └── cedeo.yaml     # Règles spécifiques fournisseur
```

---

### 3️⃣ `parsers/` – Extraction spécifique à chaque version

Chaque parser implémente `BaseParser` et applique les règles si besoin :

```
parsers/
├── base.py            # Interface commune `BaseParser`
├── parser_21.py       # Parser pour la version 2.1
├── parser_22.py       # Parser pour la version 2.2
├── parser_30.py       # Parser XML Fab-Dis 3.x
└── utils.py           # Fonctions communes (lecture xlsx, csv, etc.)
```

> Le parser extrait les données, applique les règles, puis les normalise vers un format standard JSON.

---

### 4️⃣ `exporters/` – Sortie des données

Module responsable de transformer les résultats en fichiers de sortie :

* JSON
* CSV (si demandé)
* YAML ou autres à l’avenir

```
exporters/
├── json_exporter.py
├── csv_exporter.py
└── factory.py         # Choisit l’exporteur en fonction des options CLI
```

---

### 5️⃣ `cli/` – Interface en ligne de commande

Point d’entrée principal du programme, gère :

* Lecture des arguments
* Chargement du fichier
* Sélection des détecteurs/parsers/exporteurs
* Logging et rapports

```
cli/
├── main.py
└── utils.py
```

---

### 🧲 Tests

Des jeux de tests par version et fournisseur seront stockés dans :

```
tests/
├── fixtures/
│   ├── v21.xlsx
│   ├── v22.xlsx
│   ├── cedeo_custom.xml
│   └── ...
├── test_detectors.py
├── test_parsers.py
├── test_rules.py
└── test_end_to_end.py
```

> L’architecture permet d’ajouter de nouveaux formats ou fournisseurs sans modifier le cœur du code, simplement en ajoutant un nouveau `Detector`, `Parser`, ou `RuleSet`.


## ▶️ Utilisation

* à venir

## 🔍 Exemple de sortie JSON

* à venir



## 🔌 Intégration Laravel

Ce JSON peut être :

* Importé directement dans un job Laravel (`ProductImporterJob`)
* Utilisé via une API POST (upload fichier → analyse Python → JSON → import Laravel)

---

## 📌 TODO

### 🧠 Détection & Parsing (architecture modulaire & tolérante)

- [ ] ✅ **Auto-détection du format de fichier** (`.xlsx`, `.xml`, `.csv`)
- [ ] 📦 **Système de détection modulaire basé sur des classes `Detector`**
  - [ ] `XlsxDetector`: détecte la version via onglets, colonnes... avec fallback si les noms varient (`Produits`, `Articles`, etc.)
  - [ ] `XmlDetector`: détecte via namespaces/balises, accepte les variantes de schéma
  - [ ] `CsvDetector`: analyse des en-têtes (`libellé`, `ref_produit`, etc.), tolère les renommages
  - [ ] `DetectorManager`: classe orchestratrice, choisit et exécute le bon détecteur
- [ ] 🧪 **Validation XSD automatique** pour fichiers XML, avec gestion des schémas partiels
- [ ] ⚠️ **Détection d’incohérences / anomalies** :
  - Version introuvable
  - Fichier mal structuré
  - Colonnes obligatoires absentes
- [ ] 🧠 **Moteur de règles adaptatives (`RuleEngine`)** :
  - [ ] Réécriture des noms d’onglets (`ProduitsFabdis` → `Produits`)
  - [ ] Mapping flexible des colonnes (`libelle`, `nom`, `designation` → `name`)
  - [ ] Correction automatique de certains champs mal formés (prix `3,99 €` → `3.99`)

---

### 🛠️ Extraction & Mapping (parsers unifiés et résilients)

- [ ] 🧩 **Parsers spécialisés par version** : `Parser_21`, `Parser_22`, `Parser_30`, etc.
  - [ ] Capables d’utiliser des stratégies alternatives si la structure standard est absente
  - [ ] Intègrent le `RuleEngine` pour normaliser les entrées
- [ ] 🔌 **Interface commune `BaseParser`** pour unifier les retours
  - Signature : `.extract() -> List[Dict[str, Any]]`
- [ ] 🔄 **Mapping final vers un format pivot JSON** :
  ```json
  {
    "reference": "1234",
    "name": "Produit X",
    "brand": "MarqueY",
    "price": 4.99
  }

### 🔧 Interface & Logs
- [ ] 📜 **Logger CLI avec niveaux** (`info`, `warn`, `error`)
- [ ] 🧪 **CLI interactive** avec options `--file`, `--force-version`, `--output-format`
- [ ] 📈 **Statistiques sur les lignes extraites** (valides, ignorées, erreurs)

### 🧩 Intégration & Compatibilité
- [ ] 🏭 **Support multi-fournisseurs** (Trenois, Rexel, CEDEO, etc.)
- [ ] 🌐 **Préparation à une API REST** ou mode `import/export` via HTTP
- [ ] 🧪 **Tests unitaires et jeux de données d'exemple** (fixtures par version/fournisseur)



---

## 📘 Références

* [Site officiel FAB-DIS](https://fabdis.fr/)
* [Documentation des versions (2.1, 2.2)](https://www.fab-dis.fr/telechargements/)
