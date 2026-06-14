# Analyse des Avis et Alertes ANSSI — EFREI 2026

## Description
Pipeline complet de veille cybersécurité automatisé basé sur les bulletins
de sécurité publiés par l'ANSSI (CERT-FR). Le projet couvre l'extraction,
l'enrichissement, l'analyse, la modélisation ML et la génération d'alertes.

## Structure du projet
Projet_final/

├── data/

│   └── data.zip              # Données ANSSI + MITRE + EPSS

├── notebooks/

│   ├── 01_analyse_finale_anssi.ipynb   # Notebook principal

│   └── 01_analyse_finale_anssi.html    # Export HTML

├── outputs_anssi/

│   ├── df_final.csv                    # DataFrame consolidé (125 936 lignes)

│   ├── alertes_critiques.csv           # 129 CVE critiques détectés

│   ├── email_alerte.txt                # Template email généré

│   └── viz_*.png                       # Visualisations

└── README.md

## Résultats clés
- **4 103** bulletins ANSSI traités (2021-2026)
- **37 287** CVE uniques enrichis via API MITRE et EPSS
- **125 936** lignes dans le DataFrame final
- **129** CVE critiques détectés (CVSS ≥ 9.0 ET EPSS ≥ 50%)

## Pipeline

### Étape 1 — Extraction
Lecture des bulletins ANSSI (alertes + avis) depuis le zip local.
Extraction du titre, date, lien et liste des CVE pour chaque bulletin.

### Étape 2 — Enrichissement
Pour chaque CVE identifié :
- **API MITRE** → score CVSS, type CWE, description, vendor, produit, versions
- **API EPSS (FIRST)** → probabilité d'exploitation

37 279 CVE enrichis depuis les fichiers locaux, 8 via appel API direct.

### Étape 3 — Consolidation
DataFrame pandas de 125 936 lignes × 20 colonnes.
Une ligne par couple (bulletin, CVE).

### Étape 4 — Visualisations (13 graphiques)
- Distribution des scores CVSS et EPSS
- Évolution temporelle des bulletins
- Top vendors et produits (Alertes vs Avis séparés)
- Scatter CVSS vs EPSS
- Heatmap des corrélations
- Boxplot CVSS par vendor
- Top CWE, courbe cumulative...

### Étape 5 — Machine Learning

**Modèle non supervisé — K-Means (K=2)**
- Cluster 0 (16 957 CVE) : profil standard, CVSS=6.65, EPSS=0.02
- Cluster 1 (106 CVE) : profil dangereux, CVSS=8.46, EPSS=0.56
- Score silhouette : 0.81

**Modèle supervisé — Random Forest**
- Cible : CVE critique (CVSS ≥ 9)
- V1 avec CVSS : F1=1.00 (data leakage détecté et corrigé)
- V2 sans CVSS : F1=0.33 (modèle réaliste, dataset déséquilibré 5.7% critiques)
- Feature la plus importante : epss_percentile (0.35)

### Étape 6 — Alertes email
Détection automatique des CVE critiques (CVSS ≥ 9.0 ET EPSS ≥ 0.5).
Génération d'un email structuré avec résumé exécutif et liste complète.

## Installation

```bash
pip install pandas numpy matplotlib seaborn scikit-learn feedparser requests jupyter
```

## Utilisation

### Sur PyCharm / Local
```bash
# Ouvrir et exécuter
notebooks/01_analyse_finale_anssi.ipynb
```

### Sur Google Colab
Remplacer dans la première cellule :
```python
base = "../"
# par :
from google.colab import drive
drive.mount('/content/drive')
base = "/content/drive/MyDrive/"
```

## Sources de données
- **ANSSI CERT-FR** : https://www.cert.ssi.gouv.fr/
- **API MITRE CVE** : https://cveawg.mitre.org/api/cve/
- **API EPSS FIRST** : https://api.first.org/data/v1/epss