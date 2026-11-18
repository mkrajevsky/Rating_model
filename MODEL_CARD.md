# Model Card: Model Oceny Ryzyka Kredytowego (PD) dla Przedsiębiorstw

## 1. Przegląd Modelu (Model Details)
* **Nazwa modelu:** Credit Risk HistGradientBoosting Model (Black-Box XAI-Enhanced)
* **Typ modelu:** Nieliniowy model uczenia maszynowego (`HistGradientBoostingClassifier`) ze zintegrowanym modułem wyjaśnialności (SHAP/LIME).
* **Data realizacji:** Listopad 2026
* **Autorzy:** Michał Krajewski, Zuzanna Skałka

---

## 2. Przeznaczenie i Zakres (Intended Use)
* **Główny cel:** Szacowanie prawdopodobieństwa niewypłacalności (Probability of Default– PD) dla przedsiębiorstw w horyzoncie 12 miesięcy.
* **Zastosowanie biznesowe:** Wsparcie procesu podejmowania decyzji kredytowych, wstępna selekcja wniosków, optymalizacja kosztów weryfikacji eksperckiej oraz generowanie planów naprawczych (*What-If*) dla klientów granicznych.
* **Do czego model NIE powinien być używany:** 
  * Automatyczne, ostateczne odrzucanie wniosków bez ścieżki weryfikacji ludzkiej (*Straight-Through Processing* bez nadzoru).
  * Ocena podmiotów spoza próby treningowej (np. startupów bez historii finansowej, jednoosobowych działalności gospodarczych rozliczających się ryczałtowo, sektora publicznego).
  * Datowanie dokładnego dnia niewypłacalności (model szacuje roczne PD, a nie harmonogram bankructwa).

---

## 3. Dane i Cechy Wejściowe (Data & Features)
* **Próba badawcza:** 3 000 unikalnych obserwacji przedsiębiorstw (wyjściowo 221 cech finansowych i wskaźnikowych).
* **Podział zbiorów:** 
  * Trening (70%): uczenie modeli i transformacje.
  * Walidacja (15%): strojenie hiperparametrów i kalibracja.
  * Test (15%): niezależna, ostateczna ewaluacja jakości.
* **Polityka danych:** 
  *Braki danych- imputacja medianą (zmienne ilościowe) oraz najczęstszą wartością (jakościowe). 
  *Outliery- obcięcie cech liczbowych do kwantyli 1% i 99%.
* **Wybrany zestaw cech (6 najważniejszych wartości):**
  1. `wsk_liczba_dni_istnienia` (Wiek przedsiębiorstwa)
  2. `Aktywa` (Skala działalności)
  3. `wsk_zadluzenia` (Zadłużenie przedsiębiorstwa)
  4. `wsk_plynnosc_biez_2` (Płynność finansowa)
  5. `wsk_ROA` (Rentowność aktywów)
  6. `wsk_rotacja_aktywow_1` (Wskaźnik efektywności)
  *(Uwaga: Ograniczenie przestrzeni cech do 6 wymiarów ma na celu wyeliminowanie zmiennych zakłócających i umożlić skuteczne, kontrolowane porównanie z modelem liniowym).*

---

## 4. Wyniki Jakościowe i Kalibracja (Performance & Calibration)
* **Zdolność dyskryminacyjna (Zbiór testowy, $n=450$, rzeczywisty DR = 12.89%):**
  * **Model Black-Box:** AUC = **0.7180**, Gini = **0.4360**, KS = **0.3626**, PR-AUC = **0.2644**.
  * *(Dla porównania bazowy Logit: AUC = 0.6404, Gini = 0.2809).*
* **Kalibracja i dostosowanie do poziomu 4% PD:**
  * Model Black-Box poddano kalibracji statystycznej z wykorzystaniem danych walidacyjnych.
  * Po kalibracji średnie PD na zbiorze testowym to około 13,65%, przy rzeczywistym default rate równym 12,89%.
  * Zastosowano dodatkowo przesunięcie poziomu PD do poziomu 4% średniej wartości na zbiorze walidacyjnym (założenie scenariuszowe).
  * Po przesunięciu średnie PD na zbiorze testowym wyniosło 4,46%.

---

## 5. Wyjaśnialność i Stabilność (Explainability / XAI)
* **Analiza globalna:** 
  * Najważniejszymi cechami predykcyjnymi w modelu Black-Box są według analizy SHAP `wsk_zadluzenia` i `wsk_ROA`.
  * Globalna analiza wag cech wykazuje duże różnice względem regresji logistycznej, ze względu na wychwytywanie nieliniowych zależności przez model.
* **Analiza lokalna i wierność (Local Fidelity):**
  * Zastosowano metody LIME oraz TreeSHAP dla wybranych przypadków.
  * Wierność lokalna surogatów liniowych została zweryfikowana błędem **MAE $\approx$ 0.028** w otoczeniu wybranych firm.
* **Testy stabilności numerycznej:**
  * Przeprowadzono eksperymenty dla 500 oraz 2 000 perturbacji.
  * Wyniki potwierdziły powtarzalność wyjaśnień (średnia korelacja rang Spearmana dla wag to > 0.95 dla 2 000 perturbacji, a wskaźnik stabilności strukturalnej Jaccard Top-3 wyniósł 1.0). 

---

## 6. Wyniki Klasyfikacji i Konsekwencje Decyzyjne (Decision Performance)
  * Dla progu decyzyjnego (0,065) wyznaczonego na zbiorze walidacyjnym dokonano porównania oceny modelu przed i po przesunięciu do 4% PD na zbiorze testowym.
  * Model Black-Box bez dodatkowego dostosowania wykrył 56 z 58 przypadków defaultu, ale wygenerował 261 fałszywych alarmów. (Czułość = 0,9655 oraz Swoistość = 0,3342)
  * Model Black-Box po dopasowaniu do średniego poziomu PD 4% wykrył tylko 19 z 58 defaultów, ale wygenerował 58 fałszywych alarmów. (Czułość = 0,3276 oraz Swoistość = 0,8520)
  * Wybór wariantu powinien zależeć od polityki kredytowej oraz relacji kosztów błędów FP i FN.

---

## 7. Ograniczenia i Ryzyka (Limitations & Risks)
* **Horyzont czasowy i cykl ekonomiczny:** Model działa w ujęciu punktowym (*Point-in-Time*). Znaczy to, że nagły wstrząsu makroekonomicznego (tj. recesja) może prowadzić do gwałtownej zmiany rozkładów ryzyka i pogorszenia predykcji.
* **Wrażliwość danych:** Jakość oceny zależy w dużej mierze od rzetelności sprawozdawczości finansowej dostarczanej przez firmy.
* **Ryzyko nadmiernego polegania na algorytmie:** Przypadki graniczne wymagają weryfikacji eksperckiej i analizy z wykorzystaniem wygenerowanego "planu naprawczego" (What-If analysis).

---

## 8. Plan Monitoringu (Monitoring Plan)
Aby zapobiec degradacji jakości modelu, proponuje się następujący plan kontroli:
1. **Częstotliwość:** Kwartalny Przegląd Ryzyka Modelu (*Model Performance Review*).
2. **Kluczowe metryki monitorowane:**
   * **Stabilność populacji (PSI – Population Stability Index):** Kontrola dryfu rozkładów cech wejściowych (szczególnie zadłużenia i ROA).
   * **Jakość predykcyjna:** Bieżące śledzenie wskaźnika Brier Score oraz funkcji straty (Log-loss) na portfelu po uzyskaniu rzeczywistych wyników defaultu.
   * **Kalibracja portfela:** Porównanie rzeczywistego wskaźnika defaultów z prognozowanym średnim poziomem PD w horyzoncie.
   * **Skuteczność klasyfikacji i struktura rozkładu klas ratingowych.**
