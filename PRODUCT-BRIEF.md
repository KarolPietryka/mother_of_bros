# Brogress — krótki opis produktu

Ten dokument jest **głównym opisem produktu** (cel, użytkownik, core flow, stack, domena, UI, ficzery, scenariusze). 

## Cel

Aplikacja do rejestrowania treningów siłowych i śledzenia progresu na wykresach.

## Docelowy użytkownik

Osoba trenująca metodą **specialization cycles** (cykli specjalizacji):

- w danym cyklu (~6 tygodni) intensywnie rozwija **jedną partię mięśniową** (partia główna),
- pozostałe partie ćwiczy pomocniczo, bez nacisku na progres.

**Wartość w jednym zdaniu:** **progres na wierzchu**, **wpis na sali w locie**, **następny trening startuje od ostatniego zestawu** (waga + powtórzenia), żeby poprawiać tylko to, co realnie poszło lepiej.

## Rdzeń produktu

Trzy filary opisane w roadmapie jako **Rdzeń** — tutaj w skrócie, żeby brief i backlog mówiły tym samym językiem:

1. **Cykl specjalizacji** — metoda użytkownika; w apce **fokus wynika z ćwiczeń** i z sensu treningu widocznego m.in. na wykresie. **Bez** osobnej encji produktowej „blok cyklu”.
2. **Prefill i kolejka** — podpowiedź z **ostatniej zapisanej sesji** (w wariancie prostym **bez** filtrowania po `bodyPart`), **ta sama kolejność serii** oraz waga/repy. **Brak wcześniejszej sesji w historii** → brak planu / pusty prefill. **Ten sam dzień, kontynuacja treningu:** istniejący workout **nie zeruje** podpowiedzi — użytkownik dostaje **następny krok** z kolejki (seria po serii / ćwiczenie po ćwiczeniu), a nie „cały zestaw naraz” w jednym rzucie w głównym flow. W **Add exercise popup** widać **aktualny trening** (wykonane serie dzisiaj) **oraz** propozycję kolejnych pozycji z **ostatniego treningu**. **Cena prostoty:** po **zmianie cyklu / fokusu** ostatnia sesja globalnie może dotyczyć **innej partii** — **„Wyczyść prefill / plan”** usuwa kolejkę; **opcjonalnie później** (patrz ficzery / roadmap): ostatni trening **tej** partii.
3. **Wykres (MVP)** — **pierwszy ekran**, jedna czytelna seria (ćwiczenie lub partia), **jeden** główny CTA; lustro progresu, **nie** zastępuje wpisywania na sali.

## Core flow (kluczowy UX)

1. **Pierwszy ekran:** wykres progresu (MVP) + jeden główny przycisk — progres ma być widoczny od razu; trening nie jest rozjeżdżany z wieloma równorzędnymi wejściami.
2. Użytkownik zapisuje ćwiczenia z wagami i powtórzeniami **podczas trwania treningu** (na bieżąco, seria po serii).
3. **Prefill z ostatniej sesji** (kolejność + waga + repy) — szczegóły **tego samego dnia vs nowa wizyta** i kolejka kroków: **Rdzeń produktu** powyżej oraz (*Prefill*). W typowym scenariuszu **kolejnej wizyty** (np. następnego dnia) użytkownik dostaje **gotowy zestaw do korekty**, zamiast składać listę od zera.
4. Użytkownik zmienia tylko to, co udało mu się zrobić lepiej (więcej kg / więcej powtórzeń).

**Cykl specjalizacji** jest rdzeniem produktu (razem z podpowiedziami i wykresami), nie funkcją dodatkową.

---

## Dla kogo dokładniej (persona)


|                                |                                                                                             |
| ------------------------------ | ------------------------------------------------------------------------------------------- |
| **Cel**                        | Rozwój wybranej partii w bloku ~6 tygodni; pozostałe partie bez presji progresu.            |
| **Kontekst**                   | Trening na żywo — telefon jako narzędzie między seriami, nie „planowanie wieczorem”.        |
| **Frustracja bez apki**        | Rozproszone notatki, brak jednego obrazu progresu, za każdym razem składanie listy od zera. |
| **Czego oczekuje od Brogress** | Jedno miejsce na dane, wykres jako motywacja, mniej klikania przy powtarzalnym planie.      |


---

## Stack

- **Backend:** Java / Spring Boot (`brogres/`)
- **Frontend:** React (`brogress_F/`)
- **Deploy:** Render (Docker, PostgreSQL)

## Encje domenowe (skrót produktowy)

- `**Workout`** — sesja treningowa (data, partia główna).
- `**Exercise`** — ćwiczenie w ramach sesji (nazwa, sets, reps, weight).

*Uwaga:* w kodzie wiersz wykonania (waga, powtórzenia, status serii) jest odrębny od katalogu nazw — patrz słownik oraz encje w `brogres/`.

---

## Słownik (UI + domena)

Terminy poniżej opisują **widok i zachowanie**; gdzie ma sens, dopisany jest kontrakt z backendem (ścieżki względem bazy API treningów, tak jak w kliencie).


| Termin                            | Opis                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Ekran Home**                    | Podsumowanie ostatnich treningów: widać, jakie ćwiczenia były wykonywane. Dane listy sesji pochodzą z `**GET /workout`** (`WorkoutController#listWorkouts`) — zwraca listę podsumowań dni z zestawami / wierszami treningu. **Najnowsza sesja:** klik w kartę → **Add exercise popup** (`WorkoutModal`). **Starsze sesje:** jeśli w skrócie widać kafel **„…”** (overflow), pierwszy klik **rozwija** kartę na pełną listę na Home; gdy karta jest już rozwinięta (albo całość mieściła się bez **„…”**), klik w kartę → ten sam modal; zapis edycji: `**PUT /workout/{id}`**. *Dlaczego `PUT`:* `**POST /workout`** dotyczy tylko **dzisiejszej** sesji (`LocalDate.now()`). |
| **Widok Brogress**                | W shellu użytkownika (**„Your Brogress”** / panel **„Current series”**) pokazywany jest **progres bieżącej serii specjalizacji**: backend zwraca punkty dla treningów **skupionych na aktualnej partii** (wolumen zagregowany **per dzień treningu**). UI: **wykres liniowy** oraz **tabela** z **datą treningu** i **wolumenem**; **wiersze tabeli można sortować** po dacie (rosnąco / malejąco). Źródło: `**GET /brogres/graph`** — `{ workoutDay, volume }`.                                                                                                                                                                                                              |
| **Summary**                       | Widok, który **zbiera historyczne treningi** (sesje zapisane w czasie), zwykle pogrupowane lub prezentowane **per dzień**. Na Home w praktyce to ta sama warstwa danych co lista z `GET /workout`. **Edycja (ficzer #4):** zależnie od stanu karty — patrz **Starsze treningi** / **Kafel „…”**.                                                                                                                                                                                                                                                                                                                                                                              |
| **Ostatni trening (na liście)**   | Najnowsza sesja jest pokazywana **w całości** — wszystkie wiersze widoczne; klik w kartę od razu otwiera **Add exercise popup**.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **Starsze treningi**              | Sesje wcześniejsze niż ostatnia: domyślnie **zwinięty** skrót siatki; gdy jest **kafel „…”**, pierwszy klik na karcie **rozwija** pełną listę na Home; przycisk **Collapse** zwija z powrotem. Gdy skrót mieści **wszystkie** wiersze (bez **„…”**), klik od razu otwiera edycję (modal). Po rozwinięciu klik w kartę otwiera **ten sam** modal co **Add exercise**.                                                                                                                                                                                                                                                                                                          |
| **Kafel „…”**                     | Na karcie **starszej** sesji, gdy **nie wszystkie** ćwiczenia mieszczą się w skróconej siatce: ostatni slot to **„…”**; obecność tego kafelka oznacza, że **pierwszy** klik na karcie **rozwija** podgląd, a **dopiero po rozwinięciu** kolejny klik wchodzi w edycję treningu.                                                                                                                                                                                                                                                                                                                                                                                               |
| **Przycisk Add exercise**         | Przycisk otwierający **Add exercise popup** (modal dodawania / zarządzania ćwiczeniami bieżącej sesji).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Add exercise popup**            | Modal podzielony na dwie części: **(1)** przyklejony (sticky) wiersz *New exercise* u góry, **(2)** *Current workout exercise list* — lista ćwiczeń bieżącego treningu.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Sticky row — New exercise**     | Górny, „przyklejony” blok w popupie: dodanie nowego ćwiczenia do **bieżącego** treningu. Zawiera m.in.: **partię ciała** (dropdown), **ćwiczenie** (łączone źródła — patrz niżej), **wagę**, **powtórzenia** oraz ikonę **+** zatwierdzającą dodanie.                                                                                                                                                                                                                                                                                                                                                                                                                         |
| **Current workout exercise list** | Lista pozycji jednej sesji w dwóch stanach: **planned** i **done** (patrz *Exercise state*). Pomiędzy nimi leży **Workout progress bar** — przesuwalny separator, który wyznacza, ile ćwiczeń już zostało zrobione. **Zmiana kolejności (drag & drop):** start przeciągania wyłącznie z **uchwytu** (grip) na wierszu ćwiczenia. **Klik w wiersz nic nie robi** — oznaczanie „zrobione / do zrobienia” idzie przez bar.                                                                                                                                                                                                                                                       |
| **Workout progress bar**          | Szeroki, draggowalny wiersz wizualnie rozdzielający listę na **DONE** (nad barem) i **PLANNED** (pod barem). **Dwa gesty, jeden efekt:** użytkownik **przesuwa bar** między ćwiczeniami, albo **przeciąga ćwiczenie** nad/pod bar — w obu przypadkach wiersze nad barem są wykonane, pod barem zaplanowane. Pierwszy wiersz pod barem jest wizualnie wyróżniony („next up”), ale **nie ma osobnego statusu `NEXT`** w modelu — jest to tylko podświetlenie. Bar zawsze leży po liczbie *done*; dodanie ćwiczenia ląduje na końcu listy (pod barem), usunięcie *done* przesuwa bar w górę.                                                                                     |
| **Exercise state**                | **Planned** — wiersz pod barem: dopisany ręcznie albo z prefillu, jeszcze nie oznaczony jako wykonany. **Done** — wiersz nad barem: zapisany jako zrobiony. Stan wynika z **pozycji względem baru**, nie z klikania. (Legacy: klienci sprzed wprowadzenia baru mogli wysyłać `NEXT`; BE traktuje to jak `PLANNED`.)                                                                                                                                                                                                                                                                                                                                                           |
| **Mechanizm dodawania ćwiczeń**   | Nowa pozycja trafia na listę po **+** w sticky row *New exercise*; **dokładany jest na koniec** *Current workout exercise list*.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **Exercise (wybór w formularzu)** | Pojedyncza pozycja treningowa powiązana z katalogiem nazw. W dropdownie: **wszystkie ćwiczenia dostępne dla wybranej partii** (fixed + custom) oraz osobna opcja **Add custom** (tworzenie własnej nazwy).                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| **Fixed exercise**                | Ćwiczenia z **globalnego katalogu** w bazie: w encji `Exercise` pole użytkownika jest puste (`user == null`). Udostępniane razem z custom przez picker.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Custom exercise**               | Ćwiczenie utworzone przez użytkownika (np. ścieżka **Add custom**). Zapisywane w tabeli `**exercise`** z `**user_id`** właściciela (ta sama tabela co katalog; rozróżnienie po `user_id`). Tworzenie: `**POST /workout/exercises*`* (ciało m.in. `bodyPart`, `name`).                                                                                                                                                                                                                                                                                                                                                                                                         |
| **Lista ćwiczeń do pickera**      | Słownik nazw dla danej partii: `**GET /workout/exercises/picker?bodyPart=…`** — w odpowiedzi m.in. katalog (`catalog`) i własne (`custom`), które FE skleja w jeden dropdown (+ wiersz *add custom*).                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |


---

## Ficzery (numeracja jak w user journeys)


| #     | Nazwa                             | Opis                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ----- | --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1** | **Progress bar postępu treningu** | W *Add exercise popup* postęp treningu oznacza się **przeciągnięciem pasek-separatora** (*Workout progress bar*) między ćwiczenia — wszystko nad barem to *done*, pod barem *planned*. Alternatywnie: **przeciągnięcie pojedynczego ćwiczenia** nad lub pod bar zmienia jego stan i przesuwa bar o jedną pozycję. Klik w wiersz nic nie robi. W jednym submitcie można w ten sposób ustawić **wiele** pozycji jako wykonane. |
| **2** | **Prefill z historii**            | Użytkownik z już zapisaną historią, wchodząc w dodawanie treningu (**Add workout**), widzi plan **wypełniony z poprzedniej zapisanej sesji** — ćwiczenia, **ta sama kolejność serii**, wagi i powtórzenia do ewentualnej korekty (wariant prosty: bez filtrowania po partii; **kolejka przy kontynuacji tego samego dnia** — patrz **Rdzeń produktu**). Źródło: `**GET /workout/prefill`**.                                  |
| **3** | **Prefill „pod fokus partii”**    | Po zmianie cyklu (np. z barków na nogi) użytkownik buduje nowy zestaw pod nową partię; **kolejnego dnia** system ma **prefillować z ostatniego treningu, który realnie dotyczył tej partii** (np. nogi), a nie mieszać z ostatnią sesją na innej grupie mięśniowej. *Stan produktu:* pełna reguła „ostatnia sesja **tej** partii” jest planowana jako rozszerzenie prostszej logiki „ostatnia sesja w czasie”                |
| **4** | **Edycja sesji z summary**        | **Ostatnia sesja:** klik w kartę → `WorkoutModal` jak **Add exercise**. **Starsze:** przy kafelku **„…”** najpierw klik **rozwija** kartę na Home, potem klik **otwiera** `WorkoutModal`; bez **„…”** (całość w skrócie) jeden klik → modal. Backend: `**PUT /workout/{id}`** przy zapisie edycji istniejącej sesji; `**POST /workout`** — dzisiejsza sesja.                                                                 |
| **5** | **Zapis na drop w trybie edit**   | W trybie edycji istniejącej sesji (otwarte z karty w Summary) **każdy drop** — zarówno bara, jak i ćwiczenia — od razu wysyła `**PUT /workout/{id}`** z pełną nową listą (fire-and-forget). W trybie komponowania **nowego** treningu zmiany są lokalne i lecą na BE dopiero przy finalnym submitcie (`**POST /workout`**).                                                                                                  |


---

## User journeys

### 1. Pierwsze konto i pierwszy trening (ficzer **#1**)

Użytkownik **zakłada konto**; w Brogress nie ma jeszcze historii. Wchodzi w **trening** i **zanim pójdzie na siłownię** dodaje kilka ćwiczeń (plan na dziś) przez *sticky row — New exercise* i **+**. Na starcie wszystkie wiersze są **planned** (bar na samej górze). Na miejscu realizuje sesję: **przesuwa Workout progress bar** w dół w miarę kończenia kolejnych ćwiczeń (albo przeciąga pojedyncze ćwiczenia nad bar, jeśli robi je poza kolejnością) — ficzer #1. Po zatwierdzeniu popupa (*Add*) zestaw jest zapisany: wszystko nad barem jest *done*, pod barem *planned* (jeśli coś zostało). Na **Home / Summary** pojawia się pierwszy dzień w historii (`GET /workout`).

### 2. Kolejna wizyta z historią (ficzer **#2**)

Użytkownik **loguje się** i ma już zapisane treningi. Otwiera **Add workout**; modal pokazuje listę **prefill** z ostatniej sesji — te same ćwiczenia, wagi i powtórzenia (**ficzer #2**, `GET /workout/prefill`). Podczas treningu **co jakiś czas zmienia wagę albo liczbę powtórzeń** w wierszach, żeby odzwierciedlić to, co faktycznie zrobił; zapisuje sesję jak wcześniej (*Next* / submit).

### 3. Koniec cyklu jednej partii i start drugiej (ficzer **#3**)

Po kilku tygodniach kończy np. **blok na barki** i przechodzi **fokus na nogi**. Otwiera **Add exercise**: **czyści** propozycję (w UI — np. „wyczyść plan / prefill”), bo system nadal mógłby podpowiadać zestaw z ostatniej sesji na **barki**. **Dodaje nowy zestaw** ćwiczeń pod **nogi**, realizuje trening i zapisuje. **Następnego dnia** wraca na trening: oczekiwane zachowanie (**ficzer #3**) to **prefill z ostatniej sesji skupionej na nogach** (ostatni zapisany trening, w którym priorytetem były ćwiczenia na nogi), a nie stary plan barkowy. Dopóki backend nie filtruje wyłącznie po partii, szczegół reguły jest opisany w roadmapie.

---

## Co Brogress nie musi być (na start)

- Uniwersalnym plannerem dla każdego stylu treningu bez priorytetu „jedna partia w cyklu”.
- Zamiennikiem trenera lub programu — **wspiera wykonanie i pomiar**, nie zastępuje metodyki.

