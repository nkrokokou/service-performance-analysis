# Analyse de Performance des Services Numériques (`service-performance-analysis`)

> **Contexte du projet**  
> Analyse exploratoire, audit et nettoyage d'un jeu de données opérationnel de **220 lignes** retraçant l'activité des services publics (volumes de demandes, aboutissements, échecs, délais). L'objectif est d'évaluer la performance par canal, région et période afin de formuler des préconisations stratégiques d'optimisation des services.

---

## Structure du Répertoire

```text
service-performance-analysis/
├── README.md                                    # Présentation, méthodologie & synthèses exécutives
├── requirements.txt                             # Dépendances Python (reproductibilité)
├── notebook/
│   └── analyse_performance_services.ipynb      # Notebook Jupyter exécuté de bout en bout
├── data/
│   ├── P-DA_Data_Analyst_Practical_Case_Dataset.xlsx # Jeu de données source (autonome)
│   └── P-DA_Data_Analyst_Cleaned.csv           # Jeu de données nettoyé et validé
└── outputs/                                     # Export des visualisations graphiques
    ├── evolution_temporelle.png
    └── comparaison_services_regions.png
```

---

## Guide de Lancement (Reproductibilité)

### 1. Installation des Dépendances
```bash
# Se placer à la racine du projet
cd service-performance-analysis

# Créer et activer un environnement virtuel
python -m venv .venv
source .venv/bin/activate  # Sur Windows: .venv\Scripts\activate

# Installer les packages requis
pip install -r requirements.txt
```

### 2. Exécution du Notebook
```bash
# Lancer l'interface Jupyter Notebook
jupyter notebook notebook/analyse_performance_services.ipynb
```

---

## Synthèse des Anomalies Traitées (Méthodologie de Nettoyage)

Le jeu de données présentait 5 catégories d'anomalies majeures identifiées lors de l'audit initial :

| Élément / Variable | Anomalie Identifiée | Règle / Traitement Appliqué | Justification & Métrique |
| :--- | :--- | :--- | :--- |
| **Doublons Stricts** | 10 lignes strictement identiques | Suppression via `drop_duplicates()` | Volumétrie ramenée de **220 à 210 lignes uniques** (surévaluation initiale de 4.5%). |
| **Date Invalide** | Formats mixtes + date impossible `'31/02/2026'` | Remplacement de `'31/02/2026'` par `'2026-02-28'` + conversion ISO `pd.to_datetime()` | Le 31 février n'existe pas. Convertir au 28 février préserve la ligne sans créer de `NaN`. |
| **Flux de Demandes** | $\text{Demandes reçues} \neq \text{Abouties} + \text{Échecs}$ | Recalcul strict : $\text{Reçues} = \text{Abouties} + \text{Échecs}$ | Les demandes abouties et échecs proviennent de comptages physiques. Corrige les reçues négatives (-25), l'aberration (25000) et les 0. |
| **Imputation Catégorielles** | `Région` (2 NaN), `Canal` (2 NaN) | Imputation prioritaire par le mode conditionnel selon le `Service` | Effectuée avant agrégations pour éviter l'omission de lignes dans les groupby (210 lignes conservées). |
| **Délai Moyen (sec)** | Délais = 0 s, 3600 s ou `NaN` | Imputation par la médiane conditionnelle par `(Service, Canal)` | Élimine les impossibilités (0s) et les erreurs de saisie/timeout (3600s = 1h pile). |

---

## Indicateurs Clés et Visualisations

### Indicateurs Généraux Post-Nettoyage (210 Opérations Uniques)
* **Volume total de demandes reçues** : **65 917 demandes**
* **Demandes abouties** : **55 328 demandes**
* **Échecs** : **10 589 demandes**
* **Taux de réussite global** : **83,94 %**
* **Délai moyen de traitement** : **99,3 secondes** (~1 min 39 s)

### Breakdown par Canal de Distribution
| Canal | Volume Reçues | Demandes Abouties | Échecs | Taux de Réussite (%) | Délai Moyen (sec) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Web** | 33 651 | 27 884 | 5 767 | **82,86 %** | **96,8 s** |
| **Mobile** | 25 598 | 21 992 | 3 606 | **85,91 %** | **101,8 s** |
| **Guichet assisté** | 6 668 | 5 452 | 1 216 | **81,76 %** | **100,5 s** |

### Visualisation 1 : Évolution Temporelle Mensuelle
![Évolution Temporelle](outputs/evolution_temporelle.png)

### Visualisation 2 : Performance Comparative (Services & Régions)
![Comparaison Services et Régions](outputs/comparaison_services_regions.png)

---

## Insights Clés et Recommandations Actionnables

### 1. Dominance du Canal Web et Adoption Digitale
* **Constat Chiffré** : Le canal **Web** concentre **51,1% du volume total de demandes** (33 651 / 65 917) avec un délai de traitement très performant (**96,8 secondes**).
* **Analyse** : Le Web est le canal prédominant et le plus rapide pour la prise en charge des flux d'usagers.
* **Piste d'action** : Poursuivre la dématérialisation des démarches administratives physiques et renforcer les serveurs pour absorber la charge.

### 2. Excellence de la Performance du Canal Mobile
* **Constat Chiffré** : Le canal **Mobile** enregistre le taux de réussite le plus élevé de l'organisation à **85,91%** sur un volume de 25 598 demandes (38,8% du volume global).
* **Analyse** : Les parcours simplifiés sur smartphone maximisent la complétude des dossiers.
* **Piste d'action** : Étendre l'offre de téléprocédures sur l'application mobile et ajouter un système d'alertes par notification push.

### 3. Goulot d'Étranglement et Échecs en Guichet Assisté
* **Constat Chiffré** : Le **Guichet assisté** enregistre le taux d'aboutissement le plus bas à **81,76%** avec 18,24% d'échecs (1 216 échecs sur 6 668 demandes).
* **Analyse** : Le guichet physique concentre les dossiers complexes ou incomplets nécessitant un accompagnement.
* **Piste d'action** : Implémenter un pré-contrôle numérique des pièces justificatives avant la venue de l'usager en agence.
