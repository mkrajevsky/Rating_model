# Projekt 1: Rating/Scoring z interpretowalnością, wyjaśnialnością i kalibracją

## 1) Cel projektu

Należy zaprojektować, zbudować i ocenić dwa modele predykcyjne oceniające ryzyko (rating/scoring) na podstawie danych finansowych:
1. Model interpretowalny (np. regresja logistyczna/scorecard/drzewo).
2. Model „black-box” (np. XGBoost/LightGBM/SVM/sieć).
Celem jest nie tylko skuteczność predykcyjna, ale przede wszystkim zrozumienie procesu decyzyjnego tych modeli, ocena jakości wyjaśnień oraz prawidłowa kalibracja prawdopodobieństw z uwzględnieniem 4% (tendencja centralna PD).
Wymagana jest również wyjaśnialność dwóch modeli oraz porównanie ich interpretowalności, jakości predykcyjnej i stabilności wyjaśnień.

## 2) Dane

- Zestawy danych (pliki zbiór_i.csv) zawierające zmienne finansowe i wskaźniki; zespoły pracują na przydzielonych zbiorach w oparciu o numer zbioru (i) na liście w danej grupie.
- Dołączony dokument opisujący zmienne.
- Cel (target): zdarzenie ryzyka w przyjętym horyzoncie, flaga default.
- Dane mają charakter przekrojowy - nie zawierają podziału czasowego.

## 3) Zakres prac (etapy i rezultaty)


### 3.1. Walidacja i podziały

- Podział stratyfikowany: train/validation/test oraz Stratified K-fold CV (preferowane min. 5 foldów, o ile pozwala na to liczba obserwacji i defaultów).
- W przypadku powtarzających się obserwacji tego samego podmiotu należy zastosować podział grupowy, zapobiegający występowaniu tego samego podmiotu w zbiorach treningowym i testowym.
- Dokumentacja schematu walidacji (diagram + opis).
- Kontrola data leakage, w szczególności podczas przygotowania cech, selekcji zmiennych, strojenia modeli i kalibracji.
- Zbiór testowy służy wyłącznie do końcowej oceny modeli.

### 3.2. Przygotowanie i EDA

- Kontrola jakości danych, braków, anomalii/obserwacji odstających, Data Quality.
- EDA: rozkłady, korelacje, zależności z targetem.
- Przygotowanie cech: binning (prefer. monotoniczny), WoE/encoding, capping/outlier policy, standaryzacja - w zależności od potrzeb i zastosowanych modeli.
- Wszystkie transformacje wymagające dopasowania do danych muszą być wyznaczane wyłącznie na odpowiednich zbiorach treningowych (lub foldach treningowych).

### 3.3. Model interpretowalny (obowiązkowo)

- Wybór algorytmu: regresja logistyczna (możliwe spline’y/ograniczenia monotoniczności) lub drzewo/scorecard.
- Ocena jakości: AUC/ROC, PR-AUC, KS, log-loss, Brier score.
- Interpretacja globalna: znaki i wielkości współczynników/log-odds (jeśli dotyczy), ważność cech, krzywe PDP/ICE, profilowanie binów/WoE. Należy dobrać metody odpowiednie do zastosowanego modelu.
- Interpretacja lokalna: dekompozycja skoru/logitu na wkłady cech lub analiza ścieżki decyzyjnej (w zależności od modelu) dla wybranych obserwacji (case studies).
- Stabilność: metryki na foldach CV; VIF/kolinearność (jeśli dotyczy); testy monotoniczności kierunków wpływu.
- Omówienie wpływu najważniejszych cech i zgodności uzyskanych zależności z intuicją ekonomiczną.

### 3.4. Model „black-box”

- Wybór algorytmu: XGBoost/LightGBM/SVM/sieć (płytka)/Random Forest.
- Tuning (Bayes/Random/Grid) z kontrolą overfittingu.
- Wyjaśnialność model-agnostic/post hoc: SHAP (summary, beeswarm, dependence), LIME (kilka obserwacji), „what-if”.
- Porównanie jakości i spójności wyjaśnień z modelem interpretowalnym.

#### Wierność wyjaśnień (obowiązkowo)

- Należy przeprowadzić przynajmniej jeden eksperyment oceniający wierność wyjaśnień (fidelity).
- Przykładowo: dla LIME porównanie predykcji lokalnego modelu zastępczego z predykcjami modelu black-box w otoczeniu wybranej obserwacji (np. lokalne R² lub błąd aproksymacji).
- Omówienie przypadków, w których wyjaśnienia mogą być niepełne lub mylące.

#### Stabilność wyjaśnień (obowiązkowo)

- Należy przeprowadzić przynajmniej jeden eksperyment oceniający stabilność wyjaśnień.
- Przykładowo: wielokrotne uruchomienie LIME dla tej samej obserwacji (różne seedy) i porównanie uzyskanych wyjaśnień.
- Ocena stabilności rankingów ważności cech (np. Spearman, Jaccard dla Top-k).
- Krótki komentarz, jakie czynniki wpływają na stabilność wyjaśnień i czy ewentualna niestabilność wynika z metody wyjaśniającej czy z zachowania samego modelu.
Należy uwzględnić ograniczenia SHAP/LIME, m.in. wpływ korelacji cech oraz fakt, że wyjaśnienia nie stanowią dowodu zależności przyczynowo-skutkowych.

### 3.5. Kalibracja probabilistyczna — obowiązkowo 4%


### A. Kalibracja statystyczna

- Diagnostyka „pre-cal”: krzywe reliability, histogram predykcji, Brier, log-loss, calibration intercept i calibration slope.
- Metody: Platt scaling (sigmoid), Isotonic regression. Należy porównać obie metody, jeśli liczebność danych pozwala na ich wiarygodne zastosowanie.
- Opcjonalnie: Beta calibration, ECE/ACE, dekompozycja Brier score.
- Walidacja „post-cal”: ponowna ocena Brier/log-loss/krzywych reliability oraz calibration intercept/slope.
- Kalibrację należy przeprowadzać na danych niewykorzystanych do dopasowania modelu bazowego (np. na predykcjach out-of-fold lub wydzielonym zbiorze kalibracyjnym), bez wykorzystywania zbioru testowego.

### B. Dostosowanie do 4% (tendencja centralna PD)

- Przyjmujemy docelową, długoterminową tendencję centralną PD = 4%.
- Należy porównać średnią PD po kalibracji statystycznej z częstością defaultów w danych oraz założonym poziomem 4%.
- Calibration-in-the-large: dostosowanie poziomu średniej przewidywanej PD do 4% (np. korekta interceptu w skali logitowej).
- Dostosowanie należy wyznaczyć na przyjętej populacji referencyjnej (bez użycia zbioru testowego).
- Walidacja „post-cal 4%”: ocena średniej PD, rozkładu PD, Brier/log-loss oraz krzywych reliability na niezależnym zbiorze testowym.
- Należy omówić wpływ dostosowania do 4% na jakość kalibracji i ranking ryzyka.
Ważne: dostosowanie średniej PD do 4% nie jest równoznaczne z poprawą kalibracji statystycznej. Jeżeli częstość defaultów w danych różni się od 4%, należy omówić konsekwencje i założenia takiego dostosowania.
Poziom 4% traktujemy jako zewnętrzne założenie biznesowe dotyczące docelowego portfela. Średnia PD na niezależnym zbiorze testowym nie musi być dokładnie równa 4%.
W przypadku sztucznego zmieniania proporcji klas (np. oversamplingu) należy uwzględnić wpływ tego zabiegu na estymowane prawdopodobieństwa.

### 3.6. Progi decyzji i mapowanie na ratingi

- Definicja funkcji kosztu/korzyści (TP/FP/FN/…), dobór progu operacyjnego (ROC/PR/cost curves).
- Mapowanie PD -> klasy ratingowe (np. AAA…CCC, a D dla stwierdzonego defaultu), z wymogiem monotoniczności PD między klasami ratingowymi.
- Tabele decyzyjne i przykład wpływu progu na wyniki biznesowe.
- Porównanie decyzji podejmowanych na podstawie modelu interpretowalnego i black-box.

#### Analiza kontrfaktyczna (obowiązkowo)

- Dla jednej wybranej obserwacji należy przeprowadzić analizę kontrfaktyczną.
- Przykład: jakie możliwie najmniejsze zmiany parametrów finansowych pozwoliłyby poprawić rating przedsiębiorstwa (np. z BB do BBB)?
- Należy uwzględnić realność proponowanych zmian oraz zależności między cechami.
- Porównanie wyników analizy z wyjaśnieniami SHAP/LIME.
- Krótki komentarz dotyczący ograniczeń analizy kontrfaktycznej (w szczególności braku gwarancji zależności przyczynowej).

### 3.7. Raport z interpretowalności/wyjaśnialności

- Global: 5-10 najważniejszych cech z komentarzem „dlaczego” + zgodność z intuicją/ekonomią.
- Lokalnie: 3-5 studiów przypadku (w tym jeden „graniczny”) z SHAP/LIME i wnioskami operacyjnymi. Dla modelu interpretowalnego należy przedstawić odpowiednie wyjaśnienia bezpośrednie.
- Wierność i stabilność: wyniki przeprowadzonych eksperymentów, komentarz dotyczący jakości wyjaśnień.
- Równość i stabilność: szybki audyt podgrup (różnice w Brier/AUC/PD - jeśli dane zawierają odpowiednie zmienne i liczebności pozwalają na wiarygodną ocenę). Nie wymaga się analizy stabilności czasowej.
- Porównanie modeli: skuteczność predykcyjna, złożoność, interpretowalność, wiarygodność wyjaśnień oraz kalibracja.
- Wnioski: czy ewentualna poprawa jakości predykcyjnej modelu black-box uzasadnia utratę bezpośredniej interpretowalności?

### 3.8. Prezentacja dla odbiorców nietechnicznych

- 10-12 slajdów: problem -> dane -> modele -> wyjaśnienia (global/local) -> wierność/stabilność -> kalibracja 4% -> progi/ratingi -> analiza kontrfaktyczna -> ryzyka/limity -> rekomendacje.
- Prezentacja powinna koncentrować się na interpretacji wyników i ich znaczeniu biznesowym, a nie na szczegółach implementacyjnych.

## 4) Metryki (minimum)

- Skuteczność: AUC, PR-AUC, KS, log-loss.
- Kalibracja: Brier, calibration-in-the-large (intercept), calibration slope, krzywe reliability (pre/post-cal), średnia PD.
- Wierność wyjaśnień: przynajmniej jedna miara (np. lokalne R² dla LIME).
- Stabilność wyjaśnień: przynajmniej jedna miara (np. Spearman/Jaccard dla rankingów ważności).
- Stabilność modeli: wariancja metryk w CV.
- Złożoność: liczba cech, ograniczenia monotoniczne, rozmiar drzewa/liczba reguł.
Opcjonalnie: ECE/ACE, dekompozycja Brier, dodatkowe metody oceny wierności i stabilności oraz analiza podgrup.

## 5) Wymagane materiały do oddania

1. Repo/projekt z pipeline’em: EDA -> cechy -> walidacja -> modele -> wyjaśnialność -> kalibracja 4% -> progi/ratingi.
2. Notebook(i) + skrypty; plik requirements.txt/environment.yml.
3. Raport techniczny (PDF/Markdown) z pełnymi wynikami i sekcją interpretowalności/wyjaśnialności, wierności i stabilności wyjaśnień oraz sekcją kalibracji 4% (krzywe, Brier, porównanie pre/post-cal).
4. Prezentacja dla osób nietechnicznych (PDF lub PPTX).
5. Plik MODEL_CARD.md: cel, dane, ograniczenia, ryzyka, wyjaśnienia global/local, wyniki kalibracji (4%), plan monitoringu.

## 6) Ramy organizacyjne

- Zespoły: 2 osoby.
- Technologie (przykładowe): Python (pandas, scikit-learn, xgboost/lightgbm, shap, lime, matplotlib).
- Kontrola wersji: krótki README z instrukcją uruchomienia.
- Replikowalność: seed, zapis podziałów, wersjonowanie cech.
- Walidacja: wszystkie decyzje dotyczące wyboru modelu, kalibracji i progów powinny być podjęte bez wykorzystania zbioru testowego.

## 7) Kryteria oceny (40 pkt)


### A. Realizacja projektu – 30 pkt

| Kryterium | Punkty |
| --- | ---: |
| Poprawność danych i EDA | 2,0 |
| Walidacja i brak leaków | 2,0 |
| Model interpretowalny – poprawność, jakość i interpretacja | 4,0 |
| Model „black-box” – poprawność, jakość i porównanie | 2,0 |
| Interpretowalność i wyjaśnialność (globalna i lokalna) obu modeli | 6,0 |
| Wierność i stabilność wyjaśnień – eksperymenty i interpretacja wyników | 4,0 |
| Kalibracja statystyczna i dostosowanie do 4% (pre/post, Brier, reliability, calibration intercept/slope) | 5,0 |
| Progi/ratingi, wpływ biznesowy i analiza kontrfaktyczna | 3,0 |
| Model Card, dokumentacja i replikowalność | 2,0 |
| Razem | 30,0 |


### B. Prezentacja i obrona projektu – 10 pkt

| Kryterium | Punkty |
| --- | ---: |
| Jasność i struktura prezentacji, dostosowanie do odbiorców nietechnicznych | 2,0 |
| Interpretacja wyników i przedstawienie ich znaczenia biznesowego | 3,0 |
| Krytyczna ocena modeli, wiarygodności wyjaśnień i ograniczeń rozwiązania | 2,0 |
| Znajomość zastosowanych metod i odpowiedzi na pytania podczas obrony | 3,0 |
| Razem | 10,0 |



### C. Zasady oceniania

- Oceniana jest przede wszystkim poprawność zastosowanych metod, jakość eksperymentów, interpretacja wyników i umiejętność uzasadnienia podjętych decyzji. Sama wysoka skuteczność predykcyjna nie gwarantuje uzyskania wysokiej liczby punktów.
- W ocenie wyjaśnialności szczególne znaczenie mają jakość wyjaśnień globalnych i lokalnych, wyniki eksperymentów dotyczących wierności i stabilności oraz świadomość ograniczeń zastosowanych metod.
- W ocenie kalibracji istotne jest prawidłowe rozróżnienie kalibracji statystycznej od dostosowania do zewnętrznie przyjętej tendencji centralnej PD = 4%. Nie wymaga się uzyskania średniej PD dokładnie równej 4% na zbiorze testowym.
- Każdy członek zespołu musi uczestniczyć w prezentacji lub obronie projektu i wykazać się znajomością zastosowanego rozwiązania. Punkty za prezentację i obronę mogą różnić się między członkami zespołu, w zależności od wykazanej wiedzy i indywidualnego wkładu.
- Zgodnie z regulaminem przedmiotu wymagane jest uzyskanie co najmniej 15/30 pkt za realizację każdego projektu. Punkty za prezentację nie zastępują tego minimum. Zaliczenie laboratorium wymaga ponadto co najmniej 40/80 pkt łącznie z obu projektów wraz z prezentacjami oraz spełnienia pozostałych warunków określonych w regulaminie.
- Korzystanie z narzędzi AI jest dozwolone zgodnie z regulaminem przedmiotu. Samo ich wykorzystanie nie obniża oceny. Student odpowiada za poprawność oddanego rozwiązania i musi rozumieć zastosowane metody.
