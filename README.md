# Rating_model

Projekt ratingu/scoringu z interpretowalnością, wyjaśnialnością i kalibracją PD do 4%. Pełne wymagania: [project_desc.md](project_desc.md).

## Uruchomienie

Z katalogu projektu:

```bash
uv sync
uv run jupyter notebook Wstępna_analiza.ipynb
```

Wybierz kernel środowiska projektu i uruchom komórki od początku. Dane wejściowe znajdują się w `data/`: `zbiór_2.csv` oraz `Dane_Opis_zmiennych.pdf`. Notebook obsługuje różne normalizacje polskich znaków w nazwie CSV na macOS.

## Przygotowanie danych

Notebook zachowuje stratyfikowany podział train/validation/test 70/15/15 i seed 42. EDA jest wykonywane na train. Pipeline obejmuje zamianę nieskończoności na braki, rozpoznanie kodów kategorii, usunięcie stałych i identycznych kolumn, capping 1%/99%, imputację i flagi braków, standaryzację oraz one-hot encoding.

Podstawą są komórki z laboratoriów:

- `labs/Lab_3_rating_projekt1.ipynb`, komórka 19 (indeks od 0): `Pipeline`, `ColumnTransformer`, imputacja, skalowanie i kodowanie kategorii.
- `labs/Lab_1_scoring.ipynb`, komórka 15: `preprocessing_ohe`; komórka 31 zawiera `WoEBinner` do wykorzystania przy modelu interpretowalnym.

Materiały źródłowe znajdują się w sąsiednim katalogu `../../XAI/labs` względem ScoringModel. Nie są potrzebne do uruchomienia notebooka.

W CV do całego pipeline’u (preprocessing + model) przekazujemy **surowe `X_train`**, a nie gotowe `Z_train`. Każda transformacja jest wtedy dopasowywana od nowa na foldzie treningowym. `Z_train` i `Z_val` służą obecnie do sprawdzenia przygotowania danych. Test pozostaje do końcowej oceny.

Notebook zapisuje surowe podziały i `splits.csv` w `artifacts/`. Pliki pickle należy wczytywać tylko z zaufanego źródła. Nie używamy wcześniejszych plików `X_temp.pkl` / `X_test.pkl` z katalogu głównego.

Modele, kalibracja, wyjaśnienia i raportowanie pozostają do realizacji.
