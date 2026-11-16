# Stan projektu i plan wykorzystania laboratoriów

Stan przeglądu: 15 listopada 2026. Numery komórek poniżej są indeksami od 0, jak w komentarzach notebooka; numer widoczny w interfejsie może być o 1 większy.

## Ocena postępu

Po dodaniu kalibracji obu modeli, SHAP, LIME oraz eksperymentów wierności i stabilności szacujemy wykonanie na **50–55% całego oddania**, czyli pozostaje około **45–50%**. To ocena nakładu pracy i kompletności materiałów, nie prognoza punktów. Działający kod modeli nie oznacza ukończonego projektu: interpretowalność, wierność, stabilność i dokumentacja mają dużą wagę w wymaganiach.

Część obliczeniowa ma gotowe podstawy: EDA, preprocessing, regresję, eksperyment kalibracji regresji, drugi model, CV oraz część wyjaśnień. Zrealizowano też SHAP/LIME, eksperymenty wierności i stabilności oraz kalibrację drugiego modelu. Pozostają decyzje biznesowe, kontrfaktyk, domknięcie interpretacji, końcowy test i materiały do oddania.

Orientacyjny pozostały nakład: **8–15 roboczogodzin**, przy wykorzystaniu istniejącego kodu laboratoriów, bez większych problemów ze środowiskiem. To roboczogodziny zespołu, a nie gwarantowany czas kalendarzowy; weryfikacja znaczenia wskaźników może wymagać dodatkowej informacji od prowadzącego.

## Co już jest

| Obszar wymagań | Status | Wynik i otwarte elementy |
| --- | --- | --- |
| Dane i EDA | W dużej części | Jakość danych, braki/inf, rozkłady, korelacje, słownik 217 zmiennych. Nieustalone wzory wskaźników, horyzont defaultu i dostępność cech w momencie decyzji. |
| Walidacja | W dużej części | Stratyfikacja, surowe zapisane podziały, pipeline dopasowywany na train/foldzie, CV obu modeli. Brakuje graficznego diagramu i końcowej oceny testowej. Brak identyfikatora firmy ogranicza sprawdzenie grup. |
| Regresja logistyczna | Model bazowy gotowy | Sześć cech, metryki, CV, współczynniki/OR, VIF, bezpośrednia dekompozycja logitu dla przykładów. Kierunki płynności i rotacji wymagają analizy. |
| Black-box | Model bazowy gotowy | HistGradientBoosting z Lab 3, strojenie liczby iteracji na train, metryki train/validation i CV. Wyniki CV po wyborze hiperparametru są wstępne. |
| Interpretacja globalna | Częściowo | Współczynniki, permutacyjna ważność, PDP obu modeli, ICE i diagnostyka monotoniczności. Dodano TreeSHAP, beeswarm, dependence i porównanie rankingów. Brakuje pełnego komentarza ekonomicznego w raporcie. |
| Interpretacja lokalna | Częściowo | Dokładne wkłady regresji dla przykładów z validation. Dodano SHAP/LIME dla pięciu firm oraz dokładne wkłady końcowego łańcucha. Brakuje końcowego przypadku przy progu operacyjnym. |
| Kalibracja | Częściowo | Platt/isotonic/brak kalibracji dla regresji, CV kalibratorów, ECE/ACE/CITL/slope/reliability, dostosowanie do 4%. Analogiczny eksperyment black-box jest wykonany. Brakuje niezależnej oceny testowej obu modeli. |
| Wierność i stabilność XAI | Eksperymenty wykonane | LIME: R² na próbie syntetycznej i innych rzeczywistych firmach validation, powtórzenia pięciu seedów przy 500/2000 perturbacjach, Spearman i Jaccard. Wierność jest ograniczona; konieczne omówienie w raporcie. |
| Progi, ratingi, kontrfaktyk | Do zrobienia | Trzeba ustalić koszty, ratingi, ograniczenia zmian i porównać decyzje obu modeli. |
| Raport / Model Card / prezentacja | Do zrobienia | Są wymagania, README i źródła kodu. Nie ma kompletnych materiałów do oddania. |

## Dokładna mapa kodu z laboratoriów

| Potrzebna rzecz | Źródło i komórki | Jak wykorzystujemy / co zmienić |
| --- | --- | --- |
| Regresja, metryki i OR | Lab 1: 15, 17, 20 | Już użyte. Nasze cechy, pipeline i validation zamiast końcowego testu. |
| WoE, jeżeli potrzebne | Lab 1: 31, 33, 38 | Gotowe klasy i tabele. Nieobowiązkowe dla obecnej regresji; nie dodajemy tylko dla zwiększenia liczby metod. Selekcja IV musi być wewnątrz foldów. |
| Diagnostyka i kalibracja | Lab 2: 14, 20, 22, 24; Lab 3: 48, 50 | Już użyte dla regresji; ten sam eksperyment został przeniesiony na predykcje black-box. |
| Black-box i tuning | Lab 3: 33 | Przeniesione: HGB i siatka iteracji. Preprocessing naszego zbioru i CV na train zamiast strojenia na validation. |
| CV | Lab 3: 38 | Pętla zachowana, ale `StratifiedKFold` zamiast `TimeSeriesSplit` — dane przekrojowe. |
| Przedziały ufności i różnica modeli | Lab 3: 35 | Gotowe `bootstrap_ci` / `bootstrap_roznica_gini`; wykorzystać po zamknięciu wyboru modeli na teście. |
| VIF | Lab 3: 26 | Przeniesiona część diagnostyczna, bez automatycznego usuwania cech. |
| Dekompozycja regresji | Lab 4: 21 | Przeniesiony wzór β(z−E[z]); średnia train, bez biblioteki SHAP. Nie ma dummy variables, więc grupowanie nie jest potrzebne. |
| TreeSHAP | Lab 4: 26 | Przeniesione TreeExplainer i kontrola addytywności; validation i nasze sześć cech. |
| Globalny ranking i beeswarm | Lab 4: 36 | Przeniesione wykresy i porównanie z logitem, bez specjalnego traktowania branży. |
| SHAP dependence | Lab 4: 38 | Przeniesione: nasze cechy i próba `min(n, liczba obserwacji)` zamiast 3000. |
| Waterfall i lokalne powody | Lab 4: 29 | Dla 3–5 wspólnych przykładów. Wskazać, czy wyjaśniamy surowy model czy końcowy po kalibracji. |
| PDP i ICE | Lab 5: 8, 10, 12, 23 | Przeniesione funkcje i wykresy; nazwy cech, mniejsza próba validation, siatki z train. |
| ALE przy korelacjach | Lab 5: 32 | Gotowe; dodać, jeśli PDP tworzy mało realistyczne kombinacje. Metoda opcjonalna. |
| Monotoniczność | Lab 5: 44 | Przeniesiona funkcja `naruszenia` dla obu modeli z naszym słownikiem hipotez. |
| Ważność permutacyjna | Lab 6: 12 | Przeniesiona pętla dla obu modeli; po SHAP dopiąć porównanie rankingów. |
| Global surrogate | Lab 6: 33, 34 | Gotowe drzewo zastępcze i R². Opcjonalne, jeśli wierność zmierzymy przez LIME. Nie wybierać głębokości na teście jak w ćwiczeniu demonstracyjnym. |
| LIME i lokalna wierność | Lab 6: 40 | Przeniesione wrapper i LimeTabularExplainer, bez obsługi branży. Dodano R² na rzeczywistym validation obok e.score z próby dopasowania LIME. |
| Stabilność LIME | Lab 6: 42, 44 | Przeniesione powtórzenia i statystyki wag; dopisano Spearmana oraz Jaccarda Top-3. |
| Ratingi | Lab 3: 60 | Funkcja `rating` i skala gotowe; przedziały trzeba przyjąć jawnie i ocenić liczebności. D przypisujemy stwierdzonemu defaultowi, nie arbitralnie wysokiej PD. |
| Ocena ratingów | Lab 3: 66 | Tabela n/defaulty/PD/DR i testy gotowe; dostosować do naszej próby przekrojowej i opisać małe klasy. To nie jest backtest czasowy. |
| Koszty i próg | Lab 2: 42; Lab 3: 73 | Preferować prostą funkcję kosztów z Lab 2. Brak EAD: nie wolno podstawiać aktywów jako ekspozycji kredytowej; ewentualne jednostkowe ekspozycje oznaczyć jako scenariusz. |
| What-if / kontrfaktyk | Lab 6: 47, 50 | Gotowy początek przeszukiwania siatki. Nasze zakresy i kroki, cel poprawy ratingu. Konieczne dodatkowe ograniczenia wykonalności i zależności cech. |
| Podgrupy | Lab 3: 53 | Gotowa funkcja agregacji; ewentualnie forma własności, jeśli liczebności wystarczą. Nie udawać audytu demograficznej równości bez takich danych. |
| Monitoring i PSI | Lab 3: 90 | Gotowa funkcja PSI; opis planu monitoringu w Model Card. Brak dat uniemożliwia analizę rzeczywistego dryfu czasowego. |
| Model Card i prezentacja | Lab 3: sekcje 6.3/6.4; Lab 6: 53 | Gotowy zakres i układ materiałów; treść trzeba napisać na podstawie naszych wyników. |

## Kolejność pozostałych prac

1. **Decyzje biznesowe** — koszty, próg, ratingi, ocena klas, realistyczny kontrfaktyk i porównanie z SHAP/LIME; przypadek graniczny względem progu (około 3–5 h).
2. **Zamknięcie eksperymentów** — zamrożenie konfiguracji obu modeli, końcowa ocena na teście, przedziały ufności, tabele i wnioski (około 1–2 h).
3. **Materiały do oddania** — raport z krytycznym omówieniem LIME, Model Card, źródła, opis AI i wkładu osób, diagram walidacji, prezentacja i kontrola uruchomienia (około 4–8 h).

Kalibracja black-box, SHAP i LIME zostały wykonane. Nie należy utożsamiać działającego kodu XAI z wysoką jakością wyjaśnień: dodatkowa ocena LIME na innych firmach wykazała ograniczoną wierność.

## Czego nie kopiujemy bez zmian

- Generatorów danych i wniosków liczbowych ze zbioru syntetycznego.
- Podziałów czasowych, migracji ratingów i analiz PIT/TTC wymagających dat oraz identyfikatorów.
- `EAD`, branży i opóźnień, których nie ma w naszym zestawie sześciu cech.
- Wybierania hiperparametrów, progów lub metod na teście, nawet jeśli pojawia się w demonstracyjnym zadaniu laboratoryjnym.
- Wartości 0.75/1.0 dla płynności oraz kroków zmian wskaźników bez potwierdzenia skali naszego wariantu.
- Interpretacji surowego TreeSHAP jako automatycznie poprawnej dla modelu po nieliniowej kalibracji. Stałe przesunięcie logitu można dodać do bazy; isotonic wymaga innego podejścia.

## Dokumentacja pochodzenia i AI

Kod przenoszony z laboratoriów należy oznaczać plikiem i numerem komórki. Funkcje `log_szanse`, `siatka_kwantyli`, `pdp`, `ice`, `naruszenia` zostały przepisane z Lab 5 bez zmiany ich treści; ich otoczenie dostosowano do naszego zbioru. Pozostałe bloki są adaptacjami opisanymi w notebooku.

Do finalnego raportu trzeba dodać opis użycia AI oraz rzeczywisty wkład członków zespołu. Asystent pomagał w przenoszeniu kodu, adaptacji pipeline, testowaniu i szkicach komentarzy; nie potwierdza to samodzielnie poprawności ekonomicznej danych. Wkład osób i zrozumienie rozwiązania muszą potwierdzić autorzy projektu.
