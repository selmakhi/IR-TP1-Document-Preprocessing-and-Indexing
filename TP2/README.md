# **IR-TP2-Pondération-des-termes**

**Recherche d'information – Séance de travaux pratiques 2 (Master 2, Informatique) :** pondération des termes avec TF, TF normalisé, DF, IDF-1, IDF-2 et TF-IDF, implémentée à partir de zéro et vérifiée avec scikit-learn.

## **Corpus (déjà tokenisé, sans prétraitement)**

| Document | Texte                                  |
| -------- | -------------------------------------- |
| D1       | information retrieval retrieval system |
| D2       | information system system system       |
| D3       | retrieval model system                 |
| D4       | information retrieval model model      |

Ordre fixe du vocabulaire partout : `[information, retrieval, system, model]`.

## **Exécution**

```
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace tp2_term_weighting.ipynb
# ou : jupyter notebook tp2_term_weighting.ipynb
```

Le notebook régénère tous les fichiers CSV dans le dossier `results/`.

## **Fonctions implémentées à partir de zéro**

| Fonction                            | Rôle                                                    |
| ----------------------------------- | ------------------------------------------------------- |
| `compute_tf(document)`              | TF brut (`Counter` utilisé uniquement pour le comptage) |
| `compute_normalized_tf(document)`   | tf / longueur du document                               |
| `build_matrix(...)`                 | Matrice documents × vocabulaire fixe                    |
| `most_frequent_terms(row)`          | Terme(s) le(s) plus fréquent(s), égalités incluses      |
| `compute_df(documents, vocabulary)` | DF, un comptage par document                            |
| `compute_idf(df, N)`                | IDF-1 = log10(N/df)                                     |
| `compute_idf_smooth(df, N)`         | IDF-2 = log10(1 + N/df)                                 |
| `compute_tfidf(tf, idf)`            | w = tf × idf                                            |

## **Fonctions des bibliothèques utilisées uniquement pour la vérification**

- `sklearn.feature_extraction.text.TfidfVectorizer` (`sublinear_tf=False`, `smooth_idf=False`, `norm=None`)
- `sklearn.feature_extraction.text.CountVectorizer` (vérification des comptages bruts)
- `pandas` pour les tableaux, `math.log10` pour les logarithmes, `collections.Counter` pour le comptage.

## **Résultats (**`results/`**)**

- `tf_matrix.csv`
- `normalized_tf_matrix.csv`
- `df_idf_comparison.csv`
- `tfidf_raw_matrix.csv`
- `tfidf_normalized_matrix.csv`

## **Conclusions principales**

- **DF :** `information`, `retrieval`, `system` = 3 ; `model` = 2. `model` possède l'IDF le plus élevé (IDF-1 = 0.3010, IDF-2 = 0.4771) ; les trois autres partagent la valeur la plus faible (0.1249 / 0.3680).
- **Terme ayant le TF-IDF le plus élevé par document (brut et normalisé) :** D1 `retrieval`, D2 `system`, D3 `model`, D4 `model`. La normalisation du TF modifie l'échelle des lignes, mais ne change pas le classement à l'intérieur d'un document.
- **Un TF élevé ne garantit pas un poids élevé :** D2 `system` (tf = 3, w = 0.3748) est inférieur à D4 `model` (tf = 2, w = 0.6021).
- **IDF-1 :** vaut 0 pour un terme présent dans tous les documents ; IDF-2 reste positif (log10(2)).
- **scikit-learn :** les comptages bruts sont identiques ; son IDF est `ln(N/df) + 1`. Les différences avec le cours proviennent de (1) l'utilisation du logarithme naturel au lieu de log10 et (2) du terme `+1`. Vérification : `tf * (sk_idf - 1) / ln(10)` reproduit exactement notre TF-IDF (par exemple, D2/`system` : 0.3748 contre 3.8630 avec scikit-learn brut).