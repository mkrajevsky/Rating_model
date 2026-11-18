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

## Bazowa regresja logistyczna

Sekcja 3 notebooka zawiera przegląd zmiennych i regresję L2 (`C=1`) z sześcioma cechami: wiek firmy, aktywa, zadłużenie, płynność, ROA i rotacja aktywów. Kod bazuje na Lab 1 (komórki 15, 17, 20) i metrykach Lab 2 (komórka 14). `data/variable_dictionary.csv` jest transkrypcją tabeli zmienna–obszar z załączonego PDF; dokument nie zawiera wzorów wskaźników.

Notebook raportuje metryki train/validation, pięć foldów CV oraz współczynniki i ilorazy szans. Test pozostaje do końcowej oceny. Raport jakości zmiennych, metryki i współczynniki są zapisywane w `artifacts/`. Ratingi, próg i kontrfaktyk, końcowa ocena testowa oraz raportowanie pozostają do realizacji.


## Kalibracja modelu bazowego

Sekcja 4 porównuje brak kalibracji, Platt scaling i isotonic w stratyfikowanym 5-fold CV na validation, przy niezmienionym modelu bazowym wyuczonym na train. Kod pochodzi z Lab 2 (komórki 14, 20, 22, 24) oraz Lab 3 (komórka 48). Raport obejmuje Brier, log-loss, ECE, ACE, CITL i slope oraz wykresy reliability i histogram PD.

Osobno wykonano przesunięcie logitu do średniej PD 4% na referencyjnym validation. Do wstępnej oceny przesunięcia użyto także wersji OOF. W tym porównaniu surowa regresja uzyskała najlepszy średni log-loss; dostosowanie do 4% pogarsza zgodność z default rate w obecnym zbiorze. Test nadal nie służy do wyboru modelu lub kalibracji.

Wyniki i wykresy są zapisywane w `artifacts/`. Funkcja `pd_logit_po_kalibracji` przygotowuje pełny łańcuch do późniejszej oceny końcowej. Sam plik konfiguracji JSON nie zawiera dopasowanych estymatorów; należy uruchomić notebook od początku.


## Bloki przeniesione z Lab 3–6

Sekcja 5 zawiera HistGradientBoosting (Lab 3, komórka 33), VIF (Lab 3, 26), dokładne wkłady regresji (Lab 4, 21), ważność permutacyjną (Lab 6, 12), PDP/ICE i diagnostykę monotoniczności (Lab 5, 8/10/12/23/44). Numery są indeksami od 0. Strojenie black-box wykonujemy na train; validation pozostaje do oceny wstępnej i kalibracji. Test nadal nie jest używany.

Szczegółowy stan realizacji, plan kolejnych etapów, szacowany nakład oraz mapa wszystkich pozostałych fragmentów laboratoriów: [project_status.md](project_status.md).


## SHAP, LIME i kalibracja black-box

Sekcje 6–8 dodają kalibrację black-box, TreeSHAP (ranking, beeswarm, dependence), dokładny Shapley końcowego łańcucha dla pięciu firm i LIME. Kod pochodzi z Lab 2/3 (kalibracja), Lab 4 (komórki 18/26/29/36/38) i Lab 6 (40/42/44); funkcje `shapley_dokladne` oraz `stabilnosc_lime` są zachowane bez zmian. Numery oznaczają indeksy od 0.

Do laboratoryjnego R² LIME dodano ocenę na innych rzeczywistych firmach validation oraz Spearmana i Jaccarda dla powtórzeń seedów. Wyniki wskazują na ograniczoną wierność LIME w tym eksperymencie; wykresy nie stanowią dowodu jakości wyjaśnień. Surowe TreeSHAP w log-szansach i LIME końcowego PD są jawnie rozdzielone.

Zależności SHAP i LIME są zapisane w `pyproject.toml` i `requirements.txt`. Przy aktywnym lokalnym środowisku projektu należącego do workspace użyj `uv sync --active` przed uruchomieniem notebooka. Dopasowane estymatory są w pamięci notebooka; do odtworzenia uruchom wszystkie komórki od początku.


## Model Card

[MODEL_CARD.md](MODEL_CARD.md) dokumentuje aktualny cel, dane, modele, wyniki validation/CV, kalibrację do 4%, wyjaśnienia, ograniczenia, plan monitoringu i użycie AI. Jest to wersja wstępna; wyniki testowe, ratingi i próg operacyjny nie zostały jeszcze uzupełnione.
