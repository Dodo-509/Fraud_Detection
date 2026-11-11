# Détection de fraudes bancaires : Random Forest + SMOTE

Pipeline de machine learning pour détecter des transactions bancaires frauduleuses dans un jeu de données **très déséquilibré** (environ 285 000 transactions, **0,17 % de fraudes**). Le projet suit la méthodologie CRISP-DM et compare trois modèles, du prétraitement jusqu'à une simulation de déploiement en temps réel.

> Projet d'études du Master Sécurité IT & Big Data (FST Tanger).

## Résultats

Évaluation sur un jeu de test qui conserve le ratio initial : **95 fraudes pour 56 651 transactions légitimes**. Les métriques portent sur la classe minoritaire (fraude).

| Modèle | Rappel | Précision | F1 | Fraudes détectées | Fausses alertes |
|---|---|---|---|---|---|
| **Random Forest + SMOTE** | **81 %** | **54 %** | **0,65** | 77 / 95 | 66 / 56 651 |
| Isolation Forest (non supervisé) | 26 % | 25 % | 0,26 | 25 / 95 | 74 / 56 651 |
| One-Class SVM (non supervisé) | 74 % | 23 % | 0,35 | 70 / 95 | 234 / 56 651 |

- Le **Random Forest** obtient le meilleur F1 : il intercepte 77 fraudes sur 95 (18 manquées) pour environ une vraie fraude par fausse alerte.
- Le **One-Class SVM** détecte beaucoup de fraudes, mais génère 234 fausses alertes, ce qui bloquerait trop de clients légitimes.
- L'**Isolation Forest** passe à côté de la majorité des fraudes sur ces données.

## Méthode

1. **Prétraitement** : mise à l'échelle avec `RobustScaler`, adaptée aux montants et aux valeurs aberrantes.
2. **Rééquilibrage** : `SMOTE` (sur-échantillonnage synthétique de la classe fraude) pour le modèle supervisé.
3. **Modèles comparés** : Random Forest (supervisé + SMOTE), Isolation Forest et One-Class SVM (non supervisés).
4. **Évaluation** : rapports de classification et matrices de confusion sur le jeu de test au ratio d'origine.
5. **Industrialisation** : le Random Forest et le `RobustScaler` sont sérialisés avec `joblib`. Un script simule un déploiement en temps réel : il reçoit une transaction brute, applique les mêmes transformations que lors de l'entraînement et renvoie un verdict.

## Données

Le jeu de données n'est **pas versionné** dans ce dépôt (taille). Télécharge-le depuis sa source : **[lien à compléter]**, puis place le fichier CSV dans `data/`.

## Installation

```bash
git clone https://github.com/Dodo-509/Fraud_Detection.git
cd Fraud_Detection
python -m venv .venv
# Windows : .venv\Scripts\activate    Linux/macOS : source .venv/bin/activate
pip install -r requirements.txt
```

Dépendances principales : `pandas`, `numpy`, `scikit-learn`, `imbalanced-learn` (SMOTE), `joblib`, `matplotlib`.

> Si `requirements.txt` n'existe pas encore : `pip freeze > requirements.txt` depuis ton environnement de travail.

## Utilisation

```bash
# 1. Entraînement et évaluation des trois modèles
python <script_ou_notebook_entrainement>

# 2. Simulation de décision en temps réel sur une transaction
python <script_simulation>.py
```

> Remplace les noms ci-dessus par ceux de tes fichiers.

## Structure du dépôt

> À adapter à ton arborescence réelle.

```
Fraud_Detection/
├── data/                     # jeu de données (non versionné)
├── <notebook_ou_scripts>     # exploration, entraînement, évaluation
├── <script_simulation>.py    # simulation de décision en temps réel
├── models/                   # pipeline sérialisé (joblib), si assez léger
├── requirements.txt
└── README.md
```

## Limites

- **Précision de 54 %** : une revue humaine reste nécessaire pour traiter les fausses alertes restantes.
- **Peu de fraudes en test** (95) : les métriques peuvent varier selon le découpage des données. Une validation croisée stratifiée renforcerait les conclusions.
- **Simulation de déploiement** : preuve de concept, pas un système de production (pas de supervision, pas de réentraînement automatique).

## Perspectives

- **Auto-encodeurs** (apprentissage profond) pour modéliser plus finement la signature des transactions normales.
- **Apprentissage en continu** : réentraînement périodique pour suivre l'évolution des comportements frauduleux.
- Optimisation du seuil de décision selon le coût d'une fraude manquée et d'une fausse alerte.

## Auteur

Adonaï Clervil, Master Sécurité IT & Big Data, FST Tanger.
