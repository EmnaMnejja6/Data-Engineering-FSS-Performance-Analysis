# Analyse Longitudinale de la Performance Étudiante

## 📋 Vue d'ensemble du projet

Suite complète de **notebooks Jupyter** pour l'analyse longitudinale de **143 étudiants** répartis sur **4 promotions** (P1, P2, I1, I2, I3) d'une école d'ingénierie tunisienne. Le projet couvre l'analyse des performances académiques sur **2 semestres**, portant sur **26 matières** organisées en **9 modules**.

**Objectif principal :** Identifier les patterns de performance, prédire les risques d'échec et segmenter les étudiants en profils homogènes pour améliorer le suivi pédagogique.

---

## 🎯 Questions de recherche

1. **Différences inter-promotions** : Les promotions présentent-elles des différences statistiquement significatives en termes de performance ?
2. **Structure des corrélations** : Existe-t-il des axes latents (ex: "profil mathématique" vs "profil développement") ?
3. **Segmentation naturelle** : Les clusters d'étudiants correspondent-ils aux promotions ou les transcendent-ils ?
4. **Prédiction du risque** : Peut-on identifier les étudiants en rattrapage dès le S1 ?
5. **Facteurs déterminants** : Quelles matières/modules sont les meilleurs prédicteurs de réussite ?

---

## 📊 Architecture du pipeline d'analyse

```
┌─────────────────────────────────────────────────────────────┐
│ 00_anonymisation.ipynb                                      │
│ → Anonymisation des données personnelles (SHA-256)          │
│ → Génération des IDs anonymes déterministes                 │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 01_parse_pv.ipynb                                           │
│ → Parsing des fichiers Excel bruts (I1_1.xlsx, etc.)       │
│ → Extraction des notes, matières, parcours                  │
│ → Création des CSVs parsés                                  │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 03_data_cleaning.ipynb                                      │
│ → Nettoyage et validation des données                       │
│ → Imputation des valeurs manquantes                         │
│ → Construction du panel (format large + long)              │
│ → Sortie: panel.csv, panel_clustered.csv                   │
└────────────────────┬────────────────────────────────────────┘
                     ↓
        ┌────────────┴────────────┬──────────────┐
        ↓                         ↓              ↓
   ┌─────────────┐        ┌──────────────┐  ┌──────────────┐
   │ 03_eda_     │        │ 05_         │  │ 04_profils_  │
   │ performance │        │ multivariate│  │ etudiants.   │
   │ .ipynb      │        │ .ipynb      │  │ ipynb        │
   │             │        │             │  │              │
   │ • Stats     │        │ • PCA       │  │ • Profils    │
   │   desc.     │        │ • SVD       │  │   démogr.    │
   │ • Tests     │        │ • AFC       │  │ • Répartition│
   │   stat.     │        │             │  │   genre/bac  │
   │ • Violin    │        └──────────────┘  └──────────────┘
   │   plots     │
   └─────────────┘
        ↓
   ┌─────────────────────────────────────────┐
   │ 06_clustering.ipynb                     │
   │ • K-Means (k=3)                         │
   │ • Hierarchical Agglomerative Clustering │
   │ • Silhouette analysis                   │
   │ • Profiling des clusters                │
   │ → panel_clustered.csv                   │
   └────────────────────┬────────────────────┘
                        ↓
   ┌─────────────────────────────────────────┐
   │ 07_predictions.ipynb                    │
   │ • Logistic Regression                   │
   │ • Random Forest                         │
   │ • Gradient Boosting                     │
   │ • Feature Importance (MDI + Permutation)│
   │ • Learning Curves                       │
   │ • ROC-AUC, Confusion Matrix             │
   └─────────────────────────────────────────┘
```

---

## 📁 Structure des données

### Fichiers d'entrée (`data/parsed/`)

| Fichier | Lignes | Colonnes clés | Description |
|---------|--------|---------------|-------------|
| `profils_anonymes.csv` | 143 | id_anon, Promotion, Genre, Origine, Licence, Bac | Profils démographiques anonymisés |
| `mapping_id_noms.csv` | 143 | id_anon, Nom, Prénom, Promotion | Pont d'identité (usage restreint) |
| `matieres.csv` | 26 | id_mat, nom_mat, nom_module, semestre | Référentiel des matières |
| `notes.csv` | 2600 | id, promo, id_mat, mg_mat, mg_module | Notes détaillées par matière/module |
| `parcours.csv` | 100 | id, promo, moy_s1, moy_s2, moy_ann, rang, passage | Trajectoire annuelle par étudiant |

### Fichiers de sortie (`data/clean/`)

| Fichier | Contenu |
|---------|---------|
| `panel.csv` | Panel complet (format large) avec toutes les variables |
| `panel_clustered.csv` | Panel enrichi avec assignations de clusters |
| `notes_clean.csv` | Notes nettoyées et imputées |
| `matieres_clean.csv` | Référentiel matières nettoyé |
| `missing_values.png` | Heatmap des valeurs manquantes avant/après imputation |

### Fichiers de visualisations (`figures/`)

#### EDA (`figures/eda/`)
- `dist_moy_ann_by_promo.png` — Distribution des moyennes annuelles par promotion
- `violin_promos.png` — Distributions comparées (violin plots)
- `correlation_matrix.png` — Heatmap des corrélations entre matières
- `semester_progression.png` — Progression S1 → S2
- `rank_vs_avg.png` — Rang vs moyenne annuelle

#### Analyse Multivariée (`figures/multivariate/`)
- `pca_scree.png` — Variance expliquée cumulée (scree plot)
- `pca_biplot.png` — Biplot PCA (individus + variables)
- `pca_loadings.png` — Contributions des variables aux composantes
- `pca_scores_by_promo.png` — Scores PCA colorés par promotion
- `svd_scores.png` — Scores SVD
- `afc_bac_quartile.png` — AFC bac × quartile de performance

#### Clustering (`figures/clustering/`)
- `kmeans_scatter.png` — Scatter plot K-Means (2 composantes PCA)
- `kmeans_silhouette.png` — Silhouette plot (validation)
- `hac_dendrogram.png` — Dendrogramme HAC
- `hac_scatter.png` — Scatter plot HAC
- `cluster_selection.png` — Comparaison k=2,3,4,5
- `cluster_boxplots.png` — Distributions par cluster
- `cluster_radar.png` — Profils radar des clusters

#### Prédiction (`figures/predictions/`)
- `classification_results.png` — Confusion matrix, ROC curves
- `regression_results.png` — Prédictions vs réalité
- `feat_importance_classification.png` — Feature importance (classification)
- `feat_importance_regression.png` — Feature importance (régression)
- `learning_curves.png` — Courbes d'apprentissage (train/val)

---

## 🔬 Techniques et méthodes

### Analyse Exploratoire (EDA)

| Technique | Notebook | Objectif |
|-----------|----------|----------|
| **Statistiques descriptives** | 03_eda_performance | Moyennes, écarts-types, quartiles par promotion |
| **Tests non-paramétriques** | 03_eda_performance | Kruskal-Wallis (3+ groupes), Mann-Whitney (2 groupes) |
| **Visualisations** | 03_eda_performance | Violin plots, distributions, corrélations |

### Analyse Multivariée (05_multivariate.ipynb)

| Technique | Sortie | Interprétation |
|-----------|--------|-----------------|
| **PCA** | Scree plot, biplot | Réduction dimensionnelle, axes latents |
| **SVD** | Scores, loadings | Factorisation matricielle, reconstruction |
| **AFC** | Cartes de correspondance | Associations catégories × promotions |

### Segmentation (06_clustering.ipynb)

| Algorithme | Paramètres | Sortie |
|-----------|-----------|--------|
| **K-Means** | k=3 (silhouette) | Centroïdes, assignations |
| **HAC** | linkage=ward | Dendrogramme, dendrogramme coupé |
| **Profiling** | Radar charts | Caractéristiques moyennes par cluster |

### Prédiction (07_predictions.ipynb)

| Modèle | Tâche | Métrique |
|--------|-------|----------|
| **Logistic Regression** | Classification (passage/rattrapage) | Accuracy, ROC-AUC |
| **Random Forest** | Classification + Régression | Feature importance (MDI) |
| **Gradient Boosting** | Classification + Régression | Feature importance (permutation) |
| **Learning Curves** | Diagnostic | Bias-variance tradeoff |

---

## 🛠️ Installation et configuration

### Prérequis
- Python 3.8+
- pip ou conda
- Jupyter Notebook ou JupyterLab

### Installation des dépendances

```bash
pip install -r requirements.txt
```

**Packages requis :**
- `pandas` — Manipulation de données
- `numpy` — Calculs numériques
- `matplotlib` — Visualisations statiques
- `seaborn` — Visualisations statistiques
- `scikit-learn` — ML (PCA, clustering, modèles)
- `scipy` — Tests statistiques
- `openpyxl` — Lecture fichiers Excel
- `pillow` — Traitement images
- `jupyter` — Notebooks interactifs

### Configuration du projet

Éditer `config.py` pour adapter les chemins et paramètres :

```python
# Racine du projet
ROOT = Path("/home/mayna/fss")
DATA_DIR = ROOT / "data"

# Promotion à analyser
ANNEE = "I1"   # P1 | P2 | I1 | I2 | I3
PROMO = None   # None = toutes les promos, ou ex: 1

# Seuils de qualité
SEUIL_NAN_MODULE   = 0.35   # % max de NaN par matière
SEUIL_NAN_ETUDIANT = 0.45   # % max de NaN par étudiant

# Palette de couleurs pour les visualisations
PALETTE_PASSAGE = {
    "admis":        "#2ecc71",
    "rattrapage":   "#e67e22",
    "redoublement": "#e74c3c",
    "en_cours":     "#3498db",
}
```

---

## 🚀 Exécution des notebooks

### Option A : Jupyter Lab (recommandé)

```bash
jupyter lab
```
Puis ouvrir les notebooks dans l'interface web.

### Option B : VS Code avec extension Jupyter

1. Installer l'extension Jupyter
2. Ouvrir un notebook `.ipynb`
3. Exécuter les cellules avec `Shift+Enter`

### Option C : Exécution en ligne de commande

```bash
# Exécuter un notebook spécifique
jupyter nbconvert --to notebook --execute notebooks/03_data_cleaning.ipynb

# Convertir en HTML
jupyter nbconvert --to html notebooks/03_data_cleaning.ipynb
```

---

## 📝 Ordre d'exécution recommandé

1. **00_anonymisation.ipynb** — Anonymiser les données (une seule fois)
   - Génère les IDs anonymes déterministes
   - Crée `profils_anonymes.csv` et `mapping_id_noms.csv`

2. **01_parse_pv.ipynb** — Parser les fichiers Excel bruts
   - Extrait les notes des PVs
   - Crée `matieres.csv`, `parcours.csv`, `notes.csv`

3. **03_data_cleaning.ipynb** — Nettoyer et construire le panel
   - Valide la qualité des données
   - Impute les valeurs manquantes
   - Crée `panel.csv` (format large)

4. **03_eda_performance.ipynb** — Analyse exploratoire
   - Statistiques descriptives par promotion
   - Tests statistiques (Kruskal-Wallis, Mann-Whitney)
   - Visualisations (violin plots, corrélations)

5. **05_multivariate.ipynb** — Analyse multivariée
   - PCA : réduction dimensionnelle
   - SVD : factorisation matricielle
   - AFC : correspondances catégorielles

6. **04_profils_etudiants.ipynb** — Profils démographiques
   - Répartition genre, bac, licence
   - Analyse par promotion

7. **06_clustering.ipynb** — Segmentation
   - K-Means (k=3)
   - Hierarchical Agglomerative Clustering
   - Profiling des clusters
   - Crée `panel_clustered.csv`

8. **07_predictions.ipynb** — Modèles prédictifs
   - Logistic Regression
   - Random Forest
   - Gradient Boosting
   - Feature importance
   - Learning curves

9. **07_quick_analysis.ipynb** — Analyses rapides ad-hoc
   - Explorations supplémentaires
   - Vérifications ponctuelles

---

## 🔐 Gestion de l'anonymisation

### Système d'IDs

- **`parcours.id`** et **`notes.id`** : IDs anonymes primaires (utilisés dans l'analyse)
- **`profils_anonymes.id_anon`** : Même espace d'IDs → jointure directe possible
- **`mapping_id_noms.id_anon`** : Pont d'identité (usage restreint)

### ⚠️ Règles de confidentialité

- ✅ Utiliser `parcours.id`, `notes.id`, `profils_anonymes.id_anon` dans l'analyse
- ❌ **JAMAIS** exposer `mapping_id_noms` dans les outputs d'analyse
- ❌ **JAMAIS** exporter les noms/prénoms dans les fichiers de résultats
- ✅ Vérifier que tous les outputs sont anonymisés avant partage

---

## 📊 Seuils et paramètres clés

| Paramètre | Valeur | Justification |
|-----------|--------|---------------|
| `SEUIL_NAN_MODULE` | 35% | Exclure les matières avec >35% de notes manquantes |
| `SEUIL_NAN_ETUDIANT` | 45% | Exclure les étudiants avec >45% de notes manquantes |
| K-Means k | 3 | Déterminé par silhouette score |
| HAC linkage | ward | Minimise variance intra-cluster |
| Train/test split | 80/20 | Standard ML |
| Random state | 42 | Reproductibilité |
| PCA variance | 95% | Seuil de variance expliquée |

---

## 📈 Interprétation des résultats

### Passage vs Rattrapage

- **Admis** : Moyenne annuelle ≥ seuil de passage
- **Rattrapage** : Moyenne annuelle < seuil, nécessite examen de rattrapage
- **Redoublement** : Échec au rattrapage

### Clusters

Les clusters identifiés par K-Means et HAC peuvent révéler :
- Des profils d'étudiants distincts (forts/faibles/moyens)
- Des patterns de performance par module
- Des groupes transcendant les promotions

### Feature Importance

Les matières/modules les plus importants pour prédire le passage :
- **MDI (Mean Decrease Impurity)** : Importance globale
- **Permutation Importance** : Importance en production

---

## 🐛 Troubleshooting

### Erreur : "Module not found"
```bash
pip install -r requirements.txt
```

### Erreur : "File not found" dans les notebooks
Vérifier que `config.py` pointe vers les bons chemins (adapter `ROOT`, `DATA_DIR`, etc.)

### Valeurs manquantes excessives
Vérifier les seuils dans `config.py` :
- Augmenter `SEUIL_NAN_MODULE` si trop de matières sont exclues
- Augmenter `SEUIL_NAN_ETUDIANT` si trop d'étudiants sont exclus

### Clusters non stables
Vérifier la graine aléatoire (`random_state=42`) et augmenter le nombre d'itérations K-Means

### Problèmes de mémoire
- Réduire la taille du dataset en filtrant par promotion
- Utiliser `PROMO = 1` dans `config.py` pour analyser une seule promotion

---

## 📚 Références et ressources

### Techniques statistiques
- **Tests non-paramétriques** : Kruskal-Wallis, Mann-Whitney (données non-normales)
- **PCA** : Réduction dimensionnelle, analyse de variance
- **SVD** : Factorisation matricielle, reconstruction
- **AFC** : Analyse des correspondances (données catégorielles)

### Clustering
- **K-Means** : Partitionnement, centroïdes
- **HAC** : Clustering hiérarchique, dendrogrammes
- **Silhouette score** : Validation de clusters

### Modèles prédictifs
- **Logistic Regression** : Baseline linéaire
- **Random Forest** : Ensemble non-linéaire, robuste
- **Gradient Boosting** : Meilleure performance, mais moins interprétable

---

## 📞 Support et contributions

Pour toute question ou amélioration :
1. Vérifier les logs des notebooks
2. Consulter les commentaires dans le code
3. Adapter les paramètres dans `config.py`
4. Vérifier les seuils de qualité des données

---

## 📄 Licence et confidentialité

Ce projet contient des données d'étudiants anonymisées. Respecter les règles de confidentialité :
- ✅ Partager les résultats agrégés
- ❌ Ne jamais exposer les données individuelles
- ❌ Ne jamais partager `mapping_id_noms.csv`
- ✅ Vérifier l'anonymisation avant tout partage

---

## 📋 Résumé des fichiers

### Notebooks
- `00_anonymisation.ipynb` — Anonymisation (SHA-256)
- `01_parse_pv.ipynb` — Parsing des PVs Excel
- `03_data_cleaning.ipynb` — Nettoyage et panel construction
- `03_eda_performance.ipynb` — Analyse exploratoire
- `04_profils_etudiants.ipynb` — Profils démographiques
- `05_multivariate.ipynb` — PCA, SVD, AFC
- `06_clustering.ipynb` — K-Means, HAC
- `07_predictions.ipynb` — Modèles prédictifs
- `07_quick_analysis.ipynb` — Analyses ad-hoc

### Configuration
- `config.py` — Paramètres centralisés
- `requirements.txt` — Dépendances Python

### Données
- `data/raw/` — Fichiers Excel bruts
- `data/parsed/` — CSVs parsés
- `data/clean/` — Panel nettoyé

### Visualisations
- `figures/eda/` — Analyses exploratoires
- `figures/multivariate/` — PCA, SVD, AFC
- `figures/clustering/` — Clusters
- `figures/predictions/` — Modèles

---

**Dernière mise à jour** : Avril 2026  
**Version** : 2.0  
**Auteur** : Équipe d'analyse  
**Statut** : Production
