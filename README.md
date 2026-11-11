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
5. **Industrialisation** : le Random Forest et le `RobustScaler` sont sérialisés avec `joblib`. Une partie du notebook simule un déploiement en temps réel : elle reçoit une transaction brute, applique les mêmes transformations que lors de l'entraînement et renvoie un verdict.

## Données

Jeu de données : [Credit Card Fraud Detection](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud) (Kaggle, Machine Learning Group de l'ULB), des transactions de cartes bancaires anonymisées et étiquetées frauduleuses ou légitimes. Il contient des variables anonymisées (`V1` à `V28`), ainsi que `Time`, `Amount` et l'étiquette `Class`.

Il n'est **pas versionné** dans ce dépôt (taille). Télécharge `creditcard.csv` depuis Kaggle, puis place-le à l'emplacement attendu par le notebook (le chemin est indiqué dans la cellule de chargement des données).

## Installation

```bash
git clone https://github.com/Dodo-509/Fraud_Detection.git
cd Fraud_Detection
python -m venv .venv
# Windows : .venv\Scripts\activate    Linux/macOS : source .venv/bin/activate
pip install pandas numpy scikit-learn imbalanced-learn joblib matplotlib jupyter
```

`imbalanced-learn` fournit SMOTE et `joblib` sert à charger les modèles sérialisés.

## Utilisation

Tout le projet se fait dans le notebook `fraud_detection.ipynb` : prétraitement, entraînement et évaluation des trois modèles, sérialisation du pipeline et simulation de décision en temps réel. Ouvre-le (Jupyter, VS Code ou Google Colab) et exécute les cellules dans l'ordre.

```bash
jupyter notebook fraud_detection.ipynb
```

## Structure du dépôt

```
Fraud_Detection/
├── fraud_detection.ipynb                          # tout le projet : préparation, modèles, évaluation, simulation
├── meilleur_modele_fraude_rf.pkl                  # Random Forest entraîné (sérialisé)
├── scaler_fraude.pkl                              # RobustScaler ajusté sur les données d'entraînement
├── rapport.docx                                   # rapport complet du projet
├── Présentation - Projet Machine Learning.pptx    # présentation
├── video.mp4                                      # démonstration
└── README.md
```

### Réutiliser le modèle sérialisé

```python
import joblib

model = joblib.load("meilleur_modele_fraude_rf.pkl")
scaler = joblib.load("scaler_fraude.pkl")
```

Utilise la même version de scikit-learn que celle de l'entraînement, sinon le chargement peut échouer ou donner des résultats différents.

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
