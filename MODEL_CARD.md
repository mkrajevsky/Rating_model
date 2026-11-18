# Model Card — scoring ryzyka defaultu przedsiębiorstw

**Wersja dokumentu:** 0.1, 18 listopada 2026.  
**Status:** eksperymentalny projekt akademicki, ocena wstępna; bez dopuszczenia do zastosowań operacyjnych.  
**Zakres:** regresja logistyczna i model black-box HistGradientBoosting.  
**Wersja kodu odniesienia:** commit `3b71486`, gałąź `michał`.  
**Autorstwo i odpowiedzialność:** zespół projektowy; imienny wkład członków zespołu należy uzupełnić przed oddaniem.

Karta odpowiada wymaganiom z [project_desc.md](project_desc.md), punkt 5.5, oraz polom z Lab 3, sekcja 6.3 (komórka 94, indeks od 0). Rozróżnia wykonane eksperymenty od planowanych prac. Główny kod: [Wstępna_analiza.ipynb](Wstępna_analiza.ipynb).

## 1. Cel i zakres zastosowania

Celem jest oszacowanie prawdopodobieństwa `default=1` na podstawie danych finansowych przedsiębiorstwa oraz porównanie skuteczności, interpretowalności i kalibracji dwóch modeli. Odbiorcami są autorzy projektu i oceniający; przykładowe wykorzystanie biznesowe to wsparcie analityka ryzyka w porządkowaniu firm według ryzyka.

**Zdarzenie i horyzont:** etykieta jest dostarczoną flagą `default`. Szczegółowa definicja zdarzenia, horyzont i moment pomiaru cech nie zostały potwierdzone. Nie przypisujemy danym automatycznie horyzontu 12 miesięcy ani definicji defaultu z syntetycznych przykładów laboratoryjnych.

Model nie powinien samodzielnie rozstrzygać o finansowaniu. Wyników nie należy przenosić na nowe portfele ani interpretować jako zweryfikowanych długoterminowych PD bez dodatkowej walidacji. PD nie jest prawdopodobieństwem skutku konkretnej interwencji finansowej.

## 2. Dane i jakość

- **Źródło:** przydzielony na zajęcia zbiór [zbiór_2.csv](data/zbiór_2.csv); zewnętrzne pochodzenie i okres obserwacji nie są ustalone.
- **Wielkość:** 3000 obserwacji, 222 cechy wejściowe i target; 389 defaultów, czyli 12,97% całego zbioru. Jest to opis zbioru wejściowego, a nie wynik oceny testowej.
- **Charakter:** dane przekrojowe, bez dat pozwalających odtworzyć kolejność czasową i bez potwierdzonego identyfikatora podmiotu. Nie można wykluczyć powtarzania się firm wyłącznie na podstawie braku identycznych wierszy.
- **Opis:** [Dane_Opis_zmiennych.pdf](data/Dane_Opis_zmiennych.pdf) przypisuje zmienne do obszarów, ale nie podaje wzorów wskaźników ani terminów ich dostępności. Przepisany słownik obejmuje 217 cech: [variable_dictionary.csv](data/variable_dictionary.csv).
- **Kontrola jakości:** wykonano analizę braków, nieskończoności, zmiennych stałych, identycznych kolumn, rozkładów i korelacji. W pełnym zbiorze cech są pozycje z bardzo dużym udziałem braków i skrajnymi wartościami.
- **Wykluczenia w małym modelu bazowym:** indeksy `wsk_index_*` pominięto z powodu braku definicji i potwierdzenia dostępności. Kody PKD, form prawnych i schematy sprawozdań oraz pozostałe wskaźniki nie weszły do celowo małego, sześciowymiarowego porównania. Nie oznacza to dowodu ich nieprzydatności ani obecności leakage.
- **Dostępność cech w momencie decyzji:** wymaga potwierdzenia; niski jednowymiarowy AUC nie wyklucza leakage.

SHA-256 wejściowego CSV: `20e41fae11c93f8d3bbf964bfba78c6fc836c4bb787ac30023277fdabf5fabf9`.

## 3. Cechy i preprocessing

Oba modele używają tego samego zestawu sześciu cech. Opisy wskaźników i oczekiwane kierunki są hipotezami opartymi na nazwach; wzory wymagają potwierdzenia.

| Cecha | Rola w modelu |
| --- | --- |
| `wsk_liczba_dni_istnienia` | Wiek firmy |
| `Aktywa` | Wielkość firmy |
| `wsk_zadluzenia` | Zadłużenie |
| `wsk_plynnosc_biez_2` | Płynność bieżąca, wariant 2 |
| `wsk_ROA` | Rentowność aktywów |
| `wsk_rotacja_aktywow_1` | Rotacja aktywów, wariant 1 |

Pipeline zamienia nieskończoności na braki, usuwa kolumny stałe i identyczne, obcina cechy liczbowe do kwantyli 1%/99%, imputuje medianą, dodaje flagi braków i standaryzuje. Ogólny preprocessing obsługuje też kategorie przez imputację i one-hot encoding; w aktualnych modelach sześć cech jest liczbowych. Dla tych cech nie stwierdzono braków/inf na pierwotnym train. Nie stosowano oversamplingu ani wag klas.

Wszystkie parametry transformacji są dopasowywane wyłącznie na train, a w CV od nowa na części treningowej każdego foldu. Capping ogranicza wpływ skrajności, ale może też spłaszczać istotne zależności w ogonach. Selekcja sześciu cech jest ekspercka i eksploracyjna; nie jest dowiedzionym optimum.

## 4. Podział i walidacja

```text
3000 obserwacji — podział stratyfikowany, seed=42
├── train:      2099 — preprocessing, modele, 5-fold CV i tuning boostingu
├── validation:  451 — ocena wstępna, CV kalibratorów, wyjaśnienia
└── test:        450 — zarezerwowany do końcowej oceny
```

Proporcje wynoszą około 70%/15%/15%; odchylenie o jedną obserwację wynika z zaokrągleń podziału. Indeksy zapisano w `artifacts/splits.csv`. Zastosowano `StratifiedKFold(n_splits=5, shuffle=True, random_state=42)`. Brak dat uzasadnia odejście od podziału czasowego z laboratoriów; nie wykonano oceny out-of-time.

Validation ma 59 defaultów. Kalibratory są porównywane przez wewnętrzne 5-fold CV na predykcjach validation przy zamrożonym modelu bazowym. Predykcja OOF kalibratora dotyczy obserwacji niewykorzystanej do dopasowania tego kalibratora. Jest to etap wyboru metody, więc wyniki nie zastępują niezależnego testu. CV wybranego boostingu jest obciążone wyborem liczby iteracji na tych samych foldach. Wybór cech również poprzedza raportowane CV i był poprzedzony EDA na train.

**Końcowa ocena na teście, przedziały ufności i niezależna ocena różnicy modeli nie zostały jeszcze wykonane.** Nie używamy etykiet testowych do wyboru kalibracji, progu ani rankingu modeli.

## 5. Modele i wybór modelu końcowego

| Model | Konfiguracja |
| --- | --- |
| Regresja logistyczna | Regularyzacja L2, `C=1.0`, `max_iter=10000`, sześć cech; bez interakcji i ograniczeń monotonicznych |
| HistGradientBoosting | Wybrano 100 iteracji z siatki 100/200/400/800; `learning_rate=0.03`, `max_leaf_nodes=8`, `min_samples_leaf=50`, `l2_regularization=2.0`, `early_stopping=False`, seed 42 |

Boosting wybrano według najniższego średniego log-loss w CV na train. Oba modele dopasowano do całego train po przygotowaniu danych w pipeline. Nie ustalono jeszcze jednego modelu końcowego. Regresja jest prostszym punktem odniesienia; niewielka różnica AUC na validation nie uzasadnia sama w sobie wyboru black-box. Boosting ma niższy Brier i log-loss, ale większą różnicę train–validation.

## 6. Jakość predykcyjna — wyniki wstępne

Poniższe wyniki dotyczą **validation, nie testu**, i surowych PD przed dostosowaniem do 4%. PR-AUC jest raportowane jako average precision.

| Model | AUC | Gini | KS | PR-AUC (AP) | log-loss | Brier |
| --- | --- | --- | --- | --- | --- | --- |
| Regresja logistyczna | 0.6807 | 0.3614 | 0.3316 | 0.2661 | 0.3710 | 0.1104 |
| HistGradientBoosting | 0.6827 | 0.3654 | 0.2953 | 0.2950 | 0.3624 | 0.1065 |

Wyniki pięciu foldów na train:

| Model | AUC — średnia ± SD | Log-loss — średnia ± SD | Brier — średnia ± SD |
| --- | --- | --- | --- |
| Regresja | 0.6805 ± 0.0398 | 0.3637 ± 0.0132 | 0.1076 ± 0.0047 |
| Boosting | 0.7150 ± 0.0533 | 0.3516 ± 0.0168 | 0.1047 ± 0.0043 |

AUC na train wynosi 0,6906 dla regresji i 0,8221 dla boostingu; dla boostingu różnica względem validation wskazuje na ryzyko overfittingu. Wynik CV wybranego boostingu jest wynikiem po selekcji hiperparametru, a nie estymacją z nested CV. Nie stwierdzamy statystycznie potwierdzonej przewagi żadnego modelu. Przedziały ufności należy wyznaczyć przy końcowej ocenie.

Źródła liczb: `artifacts/models_train_validation.csv`, `logit_cv.csv`, `black_box_cv.csv`.

## 7. Kalibracja statystyczna i zewnętrzny poziom PD = 4%

Dla obu modeli porównano Platt scaling, isotonic regression i brak kalibracji. Kryterium wyboru to średni log-loss w pięciu foldach kalibratorów na validation. W obu przypadkach wybrano **brak dodatkowej kalibracji**; nie oznacza to wykazania idealnej kalibracji. Mała liczba defaultów ogranicza wiarygodność elastycznej isotonic.

Dostosowanie poziomu wykonuje osobne przesunięcie:

`PD_4 = sigmoid(logit(PD_stat) + delta)`.

| Model | Wybrana metoda statystyczna | Delta końcowe | Średnia PD na referencyjnym validation przed / po przesunięciu |
| --- | --- | --- | --- |
| Regresja | Brak dodatkowej kalibracji | -1.298298 | 12,31% / 4,00% |
| Boosting | Brak dodatkowej kalibracji | -1.313947 | 12,65% / 4,00% |

Populacją referencyjną jest validation, bez testu. Dokładne 4% na tej populacji jest sprawdzeniem dopasowania parametru, nie dowodem jakości. Poziom 4% jest zewnętrznym założeniem biznesowym dla portfela docelowego; nie został oszacowany z dostarczonej próby. Nie zweryfikowano, czy portfel o takim poziomie defaultów ma tę samą strukturę zależności cech i ryzyka.

Diagnostyka na validation; wiersze „po 4%” wykorzystują przesunięcia dopasowane osobno w foldach. Dlatego średnia OOF może nie być dokładnie 4%.

| Model | Etap | Średnia PD | DR | Brier | Log-loss | ECE | ACE | CITL | Slope |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Regresja | Surowe PD / wybrany brak kalibracji | 0.1231 | 0.1308 | 0.1104 | 0.3710 | 0.0268 | 0.0372 | 0.0743 | 0.8596 |
| Regresja | PD po dostosowaniu do 4% — OOF | 0.0400 | 0.1308 | 0.1194 | 0.4441 | 0.0985 | 0.0908 | 1.3729 | 0.8580 |
| Boosting | Surowe PD / wybrany brak kalibracji | 0.1265 | 0.1308 | 0.1065 | 0.3624 | 0.0310 | 0.0391 | 0.0416 | 0.8645 |
| Boosting | PD po dostosowaniu do 4% — OOF | 0.0401 | 0.1308 | 0.1171 | 0.4344 | 0.0908 | 0.0933 | 1.3545 | 0.8607 |

CITL ma wartość idealną 0, slope — 1. Przy około 13% obserwowanych defaultów dostosowanie do 4% **pogarsza** zgodność z wynikami obecnej próby: rosną Brier, log-loss i błędy kalibracji. Nie interpretujemy tego jako poprawy kalibracji statystycznej. Stałe końcowe przesunięcie zachowuje ranking ryzyka; niewielkie zmiany AUC w złączonych OOF wynikają z różnych parametrów między foldami. Średnia PD na niezależnym teście nie musi wynosić 4%.

Krzywe reliability w przedziałach kwantylowych i histogramy są w notebooku oraz `artifacts/*_calibration_diagnostics.png`. CITL, slope, ECE i ACE mają niepewność związaną z małą próbą; nie wyznaczono ich przedziałów ufności.

## 8. Interpretowalność i wyjaśnialność

### Globalnie

- **Regresja:** raportowane są β i `exp(β)`. Po cappingu i standaryzacji większe zadłużenie zwiększa przewidywane ryzyko, natomiast wyższe ROA, większy wiek i aktywa je zmniejszają. Dodatnie znaki płynności i rotacji są odmienne od wstępnych hipotez i wymagają weryfikacji. Regularyzowane β nie są efektami przyczynowymi ani testami istotności.
- **Black-box:** surowy TreeSHAP wskazuje jako najważniejsze zadłużenie i aktywa, następnie ROA, rotację, płynność i wiek. Wzrost zadłużenia wiąże się przeważnie ze wzrostem wkładu do ryzyka, a większe aktywa i ROA — ze spadkiem. Zależności są nieliniowe; plateau i lokalne zmiany kierunku wymagają ostrożnej interpretacji.
- **Porównanie:** ranking średnich bezwzględnych wkładów ma Spearmana około −0,09 między modelami. Wzór regresji i TreeSHAP używają różnych założeń tła; brak zgodności nie jest automatycznie błędem którejś metody.
- Dodano ważność permutacyjną obu modeli, PDP na siatkach kwantylowych train, ICE i diagnostykę kierunków. PDP może tworzyć nierealistyczne kombinacje zależnych wskaźników. Niski VIF około 1,01–1,04 po transformacjach nie wyklucza zależności nieliniowych.

### Lokalnie

Przeanalizowano obserwacje o indeksach 1228, 339, 960, 249 i 2657 z validation. Regresję wyjaśniono dokładnymi wkładami `β(z−E[z])`. Dla black-box pokazano surowy TreeSHAP w log-szansach oraz dokładne marginalne wartości Shapleya pełnego łańcucha po kalibracji i przesunięciu do 4%, z tłem 50 firm z train. Sprawdzono addytywność. LIME wyjaśnia pełny łańcuch w skali PD.

Przykład 2657: końcowe PD black-box wynosi **3.98%**. Dokładny Shapley wskazuje zadłużenie jako główny czynnik podnoszący logit, a ROA jako czynnik go obniżający. Lokalny surogat LIME daje w punkcie **9.89%**, czyli różni się o **5.91 p.p.** To konkretny przykład ograniczonej zgodności surogatu z modelem.

Surowego TreeSHAP nie należy automatycznie przypisywać do nieliniowo skalibrowanego modelu. LIME w PD i Shapley w log-szansach nie mają bezpośrednio porównywalnych wielkości wag. Obserwacja bliska 4% nie jest przypadkiem granicznym względem zatwierdzonego progu operacyjnego — takiego progu jeszcze nie ustalono.

### Wierność i stabilność wyjaśnień

Dla pięciu firm R² LIME na sztucznej próbie dopasowania wynosi około 0,21–0,70. Na innych rzeczywistych firmach validation, ważonych według odległości w binarnej przestrzeni dyskretyzacji LIME, R² wynosi około −0,42 do 0,01 i jest ujemne w czterech przypadkach. Oznacza to słabą wierność w tak zdefiniowanym otoczeniu; nie stanowi niezależnej oceny jakości modelu ryzyka ani dowodu nieskuteczności każdej konfiguracji LIME.

Dla stałej firmy i modelu wykonano pięć seedów przy dwóch liczbach perturbacji:

| Perturbacje LIME | Średni Spearman dla bezwzględnych wag | Średni Jaccard Top-3 |
| --- | --- | --- |
| 500 | 0,760 | 0,80 |
| 2000 | 0,874 | 0,65 |

Więcej perturbacji poprawiło średnią zgodność pełnych rankingów, ale nie zgodność Top-3. Zbliżone wagi mogą zmieniać kolejność. Losowość LIME nie jest tym samym co zmienność predykcji modelu. Niezależne perturbacje cech mogą dodatkowo naruszać zależności finansowe.

Źródła: `artifacts/shap_global_importance.csv`, `shap_local_cases.csv`, `lime_fidelity.csv`, `lime_stability_pairs.csv`; wykresy i tabele znajdują się w notebooku. Artefakty są generowane lokalnie i ignorowane przez Git — można je odtworzyć przez uruchomienie notebooka.

## 9. Skala ratingowa i backtest

**Niezaimplementowane w obecnej wersji.** Nie ma zatwierdzonych granic klas, rozkładu ratingów, oceny PD wobec DR w klasach ani sygnalizacji przekroczeń. Nie wolno przedstawiać aktualnych predykcji jako ukończonego systemu ratingowego.

Do wykonania: mapowanie PD na uporządkowane klasy, kontrola monotoniczności PD, liczebności i koncentracji klas oraz porównanie PD z DR wraz z niepewnością. Klasę D należy wiązać ze stwierdzonym defaultem, nie samą wysoką prognozą. Brak dat wyklucza rzeczywisty backtest czasowy i migracje kohortowe w dostępnych danych. Przyszłą ocenę przekrojową należy tak nazwać.

## 10. Próg decyzji i wpływ biznesowy

**Nie ustalono** funkcji kosztów, operacyjnego progu PD, stopy akceptacji ani oczekiwanej straty portfela. Brak udokumentowanej EAD uniemożliwia rzetelne wyliczenie strat kwotowych; aktywów firmy nie traktujemy jako ekspozycji kredytowej.

Przed oceną testową trzeba jawnie przyjąć scenariusz kosztów FP/FN lub korzyści i strat, wyznaczyć próg bez testu i porównać decyzje obu modeli. Wartość 4% jest docelową średnią portfela, nie automatycznym progiem akceptacji. Analiza kontrfaktyczna i ocena wykonalności zmian zależnych wskaźników pozostają do wykonania.

## 11. Ograniczenia i ryzyka

1. **Niepełne metadane:** brak potwierdzenia horyzontu, definicji defaultu, wzorów wskaźników i czasu dostępności cech. Weryfikacja leakage nie jest zakończona.
2. **Generalizacja:** mała próba i brak informacji o reprezentatywności, okresie i zmianach gospodarczych. Nie potwierdzono skuteczności na niezależnym teście ani w przyszłych okresach.
3. **Identyfikacja firm:** brak identyfikatora ogranicza kontrolę grupowego przecieku; podobne kombinacje PKD/wieku nie zastępują identyfikatora.
4. **Selekcja i tuning:** EDA i dobór cech poprzedzają CV; wynik wybranego boostingu korzysta z tych samych foldów co tuning. Wyniki mogą być optymistyczne.
5. **Nietypowe wartości:** duże skale i skrajności wskaźników utrudniają interpretację; capping może ukryć istotne przypadki.
6. **Kalibracja do 4%:** zakłada docelowy portfel o innym poziomie ryzyka. Samo przesunięcie średniej nie potwierdza filozofii TTC ani stabilności w cyklu. Nie ustalono empirycznie, czy model jest PIT czy TTC.
7. **Wyjaśnienia:** korelacje i nierealistyczne perturbacje utrudniają przypisanie wpływu. Wykazano ograniczoną wierność LIME. SHAP/LIME nie uzasadniają twierdzeń przyczynowych ani gwarancji skuteczności kontrfaktyku.
8. **Podgrupy i równość:** nie wykonano audytu podgrup. Nie potwierdzono obecności odpowiednich cech wrażliwych; brak ich w modelu nie dowodzi równego traktowania. Forma własności i PKD mogą być badane jako segmenty biznesowe, jeśli liczebności wystarczą.
9. **Gotowość:** brak zatwierdzonego modelu końcowego, ratingów, kosztów i monitoringu działającego produkcyjnie. Bieżąca wersja służy eksperymentom akademickim.

## 12. Plan monitoringu

Poniższe działania są **proponowanym planem**, a nie wdrożonym monitoringiem. Progi alarmowe są robocze i wymagają uzgodnienia oraz sprawdzenia przy docelowej liczebności i częstości napływu danych. Właściciel: zespół modelowy; osoba odpowiedzialna i procedura akceptacji zmian wymagają wskazania.

| Obszar | Miara i częstotliwość | Proponowany sygnał | Reakcja |
| --- | --- | --- | --- |
| Jakość wejścia | Schemat, typy, zakresy, NaN/inf; każda partia | Brak wymaganej cechy, niezgodny typ, nowa niefinitywna wartość; wzrost braków o >5 p.p. wobec referencji | Zatrzymanie błędnej partii lub kontrola źródła; brak automatycznej imputacji błędów bez raportu |
| Populacja | PSI PD / CSI cech; miesięcznie przy wystarczającej próbie | PSI/CSI 0,10–0,25: przegląd; >0,25: eskalacja | Analiza zmian miksu, źródeł i reprezentatywności; granice binów zamrożone na train |
| Ranking | AUC, PR-AUC, KS; kwartalnie po uzyskaniu dojrzałych etykiet | Spadek AUC o ≥0,05 wobec zatwierdzonej referencji, potwierdzony z uwzględnieniem niepewności | Analiza segmentów, zmian cech i możliwa przebudowa |
| Kalibracja | Brier, log-loss, ECE/ACE, CITL/slope, PD vs DR; kwartalnie po dojrzeniu etykiet | Utrzymujące się odchylenie CITL od 0 lub slope od 1; roboczo slope poza 0,8–1,2 z uwzględnieniem przedziałów | Ustalenie, czy zmienił się poziom ryzyka czy kształt zależności; rozważenie rekalibracji na nowych danych |
| Ratingi i decyzje | PD/DR i liczebności klas, akceptacja, FP/FN, koszt; po wdrożeniu skali i progu | Niemonotoniczne PD, koncentracja w małej liczbie klas lub trwałe niedoszacowanie DR | Przegląd granic klas i polityki; bez strojenia na końcowym teście |
| Wyjaśnienia | Średnie \|SHAP\|, kierunki PDP, rankingi i Jaccard Top-3; kwartalnie | Nowy dominujący czynnik, odwrócenie kierunków lub spadek zgodności rankingu | Analiza zmiany populacji/modelu; kontrola spójności z wiedzą finansową |
| LIME | Błąd surogatu, lokalne R² i stabilność; po zmianie modelu/metody | Utrzymująca się niska wierność lub duże błędy w punkcie | Ograniczenie użycia surogatu; preferowanie dokładnych wyjaśnień i jawnych zastrzeżeń |
| Poziom 4% | Porównanie populacji referencyjnej i docelowej; przed każdą zmianą założenia | Zmiana zewnętrznej tendencji lub struktury portfela | Ponowna ocena zasadności założenia i zatwierdzenie nowego przesunięcia, bez wymuszania 4% w każdej bieżącej partii |

PSI/CSI z progami 0,10 i 0,25 pochodzi z orientacyjnego przykładu Lab 3, sekcja 6.1; nie jest wiążącym kryterium zaliczenia ani regulacyjną normą. Przed uruchomieniem monitoringu jakości trzeba ustalić horyzont etykiety, minimalną liczebność i referencję z końcowej walidacji. Przy zbyt małej liczbie defaultów raportujemy niepewność i łączymy okresy zamiast wnioskować z pojedynczych wahań.

## 13. Odtwarzalność, źródła kodu i AI

Uruchomienie: instrukcja w [README.md](README.md); notebook należy wykonać od początku. Przy lokalnym aktywnym środowisku członka workspace: `uv sync --active`. Zależności opisano w `pyproject.toml` i `requirements.txt`; seed 42. JSON-y kalibracji zapisują metodę i deltę, ale nie są serializacją dopasowanych estymatorów. Nie ma jeszcze osobnego, wersjonowanego pakietu modelu do wdrożenia.

Źródła adaptacji (numery są indeksami komórek od 0):

- **Lab 1:** 15/17/20 — regresja, ocena i współczynniki.
- **Lab 2:** 14/20/22/24 — metryki i kalibracja.
- **Lab 3:** 19/26/33/38/48 — preprocessing, VIF, boosting, schemat CV i wybór kalibracji; 89/94/96 — zakres monitoringu i Model Card.
- **Lab 4:** 18/21/26/29/36/38 — dokładny Shapley, wkłady regresji, TreeSHAP i wizualizacje.
- **Lab 5:** 8/10/12/23/44 — PDP/ICE i diagnostyka monotoniczności.
- **Lab 6:** 12/40/42/44 — ważność permutacyjna, LIME i stabilność; 53 — zakres materiałów do oddania i opis użycia AI.

Nazwy laboratoriów odnoszą się do materiałów w `../../XAI/labs` względem projektu. Numery i niezbędne adaptacje są też opisane w notebooku i [project_status.md](project_status.md). Nie przenoszono definicji horyzontu i wniosków ze zbioru syntetycznego na nasze dane.

**Użycie AI:** wykorzystano asystenta OpenAI Codex do przeglądu materiałów laboratoryjnych, adaptacji kodu do zbioru, uporządkowania pipeline, przygotowania eksperymentów, uruchamiania kontroli oraz szkiców dokumentacji, w tym tej karty. Wygenerowany kod był wykonywany, a liczby w karcie odczytano z zapisanych wyników. Zgodność obliczeń nie potwierdza znaczenia ekonomicznego nieopisanych zmiennych. Autorzy odpowiadają za weryfikację, znajomość rozwiązania i imienny opis wkładu; wkład osób nie został tutaj wymyślony.

## 14. Warunki zamknięcia karty

- [ ] Potwierdzenie definicji defaultu, horyzontu, dostępności cech i możliwości powtarzania firm.
- [ ] Zatwierdzenie modelu końcowego oraz metody i populacji kalibracji.
- [ ] Ustalenie kosztów, progu, skali ratingowej i ocena klas.
- [ ] Realistyczny kontrfaktyk i przypadek przy progu operacyjnym.
- [ ] Niezależna ocena testowa obu modeli, pre/post-cal i pre/post-4%, z przedziałami ufności.
- [ ] Audyt podgrup, jeśli dane i liczebności na to pozwalają.
- [ ] Imienny wkład autorów, właściciel monitoringu i wersja końcowa dokumentacji.

Karta dokumentuje obecny stan projektu. Po wykonaniu powyższych etapów należy zastąpić oznaczenia „nie wykonano” rzeczywistymi wynikami, bez przedstawiania wyników validation jako wyników testowych.
