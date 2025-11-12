# Résumé des méthodes et fonctions utilisées dans le TP Pandas

Une brève explication : ce fichier liste de manière synthétique les principales méthodes et fonctions pandas vues dans le TP, avec quelques exemples pour réviser rapidement.

Ce README regroupe les méthodes et fonctions vues dans le notebook [backend/pandas.ipynb]


## 1. Entrée / sortie
- `import requests`, `import pandas as pd`, `import numpy as np` — import des bibliothèques utilisées dans le TP.
- `requests.get(url)` + `response.raise_for_status()` — téléchargement sécurisé du CSV distant.
- `open(path, "wb")` + `f.write(response.content)` — enregistrement binaire du fichier téléchargé.
- `pd.read_csv(path)` — lecture du CSV en `DataFrame` (ex. `nba = pd.read_csv("nba_all_elo.csv")`).
- `type(nba)` — vérifie que l’objet obtenu est bien un `DataFrame`.


## 2. Configuration d'affichage
- `pd.set_option("display.max.columns", None)` — affiche toutes les colonnes lors d’un `print(df)`.
- `pd.set_option("display.precision", 2)` — fixe le nombre de décimales affichées pour les floats (affichage uniquement).
  ```python
  pd.set_option("display.precision", 2)
  ```


## 3. Inspection rapide
- `len(df)`, `df.shape` — number of rows, shape (rows, columns).
- `df.head()`, `df.tail(n)` — preview first / last rows.
- `df.info()` — dtype and non-null summary.
- `df.describe()` — numeric summary statistics; `df.describe(include=object)` for categorical.
- `df["col"].value_counts()` — dénombrement des occurrences par valeur.
- `df["col"].nunique()` — nombre de valeurs distinctes dans une colonne.
- `df.axes`, `df.axes[0]`, `df.axes[1]` — axes give index and columns:
  - `df.axes[0]` is the Index of rows (same as `df.index`).
  - `df.axes[1]` is the Index of columns (same as `df.columns`).
  - Use `df.index` / `df.columns` for clarity.


## 4. Accès aux colonnes / Series
- `df["column"]` — returns a Series for that column.
- `df.column` — attribute-style access (works only for valid identifiers; can collide with DataFrame attributes).
- `type(df["column"])` — confirms it's a `pandas.Series`.
- `df.keys()` / `df.columns` — liste des colonnes disponibles.


## 5. Series (concise, with examples)
- Create:
  ```python
  s = pd.Series([10, 20, 30], index=["a", "b", "c"])  # create a Series with explicit index
  ```
- Main properties:
  - `s.values` — numpy array of values.
  - `s.index` — index labels.
  - `s.keys()` — same as `s.index`.
- Access:
  - `s.loc["a"]` — label-based access (use index labels).
  - `s.iloc[0]` — position-based access (use integer positions).
  - `s.iloc[1:3]` — slice by integer positions (stop exclusive).
  - `s.loc["a":"c"]` — slice by labels (label slicing is inclusive for end).
- `s.values`, `s.index`, `s.keys()` — propriétés principales (tableau numpy des valeurs + index).
- Membership:
  - `"a" in s` — checks whether label `"a"` exists in the Series index (not values).
- Créer une `Series` depuis un dict:
  ```python
  modules = pd.Series({"cyber-security": 10, "data-management": 15})
  ```


## 6. Sélection et filtres (explicite + exemples)
- `df.loc[condition, columns]` — label/boolean selection. Example:
  ```python
  df.loc[df["year"] > 2010, ["team_id", "pts"]]  # rows where year > 2010, these columns
  ```
- `df.iloc[...]` — purely positional selection. Example:
  ```python
  df.iloc[0:5, 0:3]  # first 5 rows, first 3 columns
  ```
- Boolean masks and combined conditions:
  ```python
  df[(df["A"] == x) & (df["B"] > y)]  # & and | require parentheses
  ```
- `df["col"].str.endswith("ers")`, `df["col"].str.startswith("LA")` — filtres textuels.
- `df[df["notes"].notnull()]` / `df[df["notes"].notna()]` — filtrer les lignes avec valeur non nulle.
- Null checks:
  - `df[df["col"].notnull()]` or `df[df["col"].notna()]` — keep rows where column is not null.
- String methods on Series:
  - `df["col"].str.endswith("ers")` — boolean Series for values ending with "ers".
  - `df["col"].str.startswith("LA")` — boolean Series for values starting with "LA".


## 7. Manipulation de colonnes (explications + exemples)
- `df.copy()` — make a deep-ish copy to avoid SettingWithCopyWarning.
  ```python
  new = df.copy()
  ```
- Create / compute new column:
  ```python
  df["difference"] = df["pts"] - df["opp_pts"]  # vectorized arithmetic
  ```
- Fill missing values:
  ```python
  df["col"] = df["col"].fillna(31)  # replace NaN with 31 for display/analysis
  ```
- Change dtype:
  ```python
  df["col"] = df["col"].astype("Int64")  # nullable integer dtype
  ```
- Rename columns:
  ```python
  df.rename(columns={"old_name": "new_name"}, inplace=True)
  ```
- Drop columns:
  ```python
  df.drop(["col1", "col2"], axis=1, inplace=True)
  ```
- Working with object / string columns (quick tips)
  - Object dtype often contains strings or mixed types; it's recommended to clean/convert them before analysis:
    - trim and normalize: `df["col"] = df["col"].str.strip().str.lower()`
    - missing handling: `df["col"] = df["col"].fillna("unknown")`
    - convert numeric strings: `df["col_num"] = pd.to_numeric(df["col_num"], errors="coerce")`
    - convert dates: `df["date"] = pd.to_datetime(df["date"], errors="coerce")`
    - convert to categorical for memory/performance: `df["col"] = df["col"].astype("category")`
  - Why convert?
    - smaller memory footprint, faster groupby/sorting when using `category`;
    - safer numeric/date operations after `to_numeric` / `to_datetime`;
    - consistent string operations when normalized (strip/lower).


## 8. Agrégation et groupby (clair et concis)
- Basic aggregation:
  ```python
  df.sum()   # sum for numeric columns
  df.max()   # max per column
  ```
- Group by one column:
  ```python
  df.groupby("fran_id", sort=False)["pts"].sum()  # total points per franchise
  ```
- Group by multiple columns and count:
  ```python
  df.groupby(["year_id", "game_result"])["game_id"].count()
  ```
- Aggregate multiple functions:
  ```python
  series.agg(("min", "max"))  # returns min and max for the Series
  ```
- Exemple issu du TP:
  ```python
  nba.groupby(["is_playoffs", "game_result"])["game_id"].count()
  ```


## 9. Types et conversions
- `pd.to_datetime(df["date_game"])` — convert string dates to datetime dtype.
- `df["col"] = pd.Categorical(df["col"])` — conversion en type catégoriel (réduction mémoire / meilleures perfs).
- `df["col"].astype("Int64")` — dtype entier nullable.
- `series.info()` / `series.nunique()` — inspecter une colonne individuellement.
- Use `df.info()` to check dtypes after conversions.


## 10. Exemple complet (NBA dataset)
- Lecture et vérifications rapides:
  ```python
  nba = pd.read_csv("nba_all_elo.csv")
  len(nba); nba.shape; nba.head(); nba.info(); nba.describe()
  ```
- Selection & grouping:
  ```python
  nba.iloc[-2]  # second to last row
  nba.loc[5555:5559, ["fran_id","opp_fran","pts","opp_pts"]]
  nba.groupby("fran_id", sort=False)["pts"].sum()
  ```


## 11. Nettoyage et qualité des données
- `nba.dropna()` — supprime les lignes contenant des valeurs manquantes.
- `nba.dropna(axis=1)` — supprime les colonnes contenant des valeurs manquantes.
- `df["notes"].fillna("no notes at all")` — remplace les valeurs manquantes par une valeur par défaut.
- Vérifications de cohérence:
  ```python
  nba[(nba["pts"] > nba["opp_pts"]) & (nba["game_result"] != "W")].empty
  ```
- Identifier les valeurs aberrantes:
  ```python
  nba[nba["pts"] == 0]
  ```


## 12. Combiner plusieurs jeux de données
- `pd.concat([df1, df2], axis=0)` — concaténation verticale (empiler lignes).
- `pd.concat([df1, df2], axis=1, join="inner")` — jointure sur l'index, colonnes alignées.
- `pd.merge(left, right, left_on="col", right_index=True, how="left")` — jointure explicite entre colonnes et index.
- Exemple du TP:
  ```python
  cities = pd.concat([all_city_data, city_countries], axis=1, sort=False)
  pd.merge(cities, countries, left_on="country", right_index=True, how="left")
  ```


## 13. Visualisation des données
- `%matplotlib inline` — active l'affichage des graphiques directement dans le notebook Jupyter.
- Graphiques à partir de Series/DataFrame:
  - `series.plot()` — graphique en ligne par défaut.
  - `series.plot(kind="bar")` — graphique en barres.
  - `series.plot(kind="pie")` — graphique en camembert.
- Exemples du TP:
  ```python
  # Graphique en ligne : total de points des Knicks par saison
  nba[nba["fran_id"] == "Knicks"].groupby("year_id")["pts"].sum().plot()
  
  # Graphique en barres : top 10 des franchises
  nba["fran_id"].value_counts().head(10).plot(kind="bar")
  
  # Graphique en camembert : résultats Heat 2013
  nba[(nba["fran_id"] == "Heat") & (nba["year_id"] == 2013)]["game_result"].value_counts().plot(kind="pie")
  ```

---

