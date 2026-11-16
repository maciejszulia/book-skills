---
name: efficient-programs
description: Speed up programs measurement-first, following Fedor G. Pikus's "The Art of Writing Efficient Programs" - set a target, profile and benchmark, then fix algorithms, memory access, allocations, branches, compiler-visible code and concurrency in C++, with the same measurement steps for Python and numpy. Use before any optimization, when code is too slow, or when the user says "performance", "optimize", "profile", "benchmark", "hot loop", "cache misses", "vectorize" or "make it faster".
---

# Wydajne programy według Pikusa: najpierw pomiar

## Cel

Skill przyspiesza program na podstawie pomiaru. Każda zmiana ma wynik pomiaru przed zmianą oraz wynik pomiaru po zmianie. Kroki pomiaru działają dla C++, Pythona oraz numpy.

## Kiedy użyć

Użyj skillu, gdy zachodzi co najmniej 1 przypadek:

- Program przekracza cel czasu, przepustowości lub pamięci.
- Agent planuje optymalizację kodu.
- Agent robi review zmiany, której opis obiecuje przyspieszenie.

## Słownik

| Termin | Definicja | Słowa zakazane |
|---|---|---|
| gorący punkt | Funkcja albo pętla, która ma największy udział w czasie z profilu. | wąskie gardło |
| pomiar bazowy | Wynik pomiaru przed zmianą, na danych z kroku 1 oraz na maszynie z kroku 1. | — |
| mikrobenchmark | Program, który mierzy czas 1 operacji w serii powtórzeń. | test szybkości |

## Kroki przy pisaniu kodu

### Etap 1: pomiar

OSTRZEŻENIE: Zmierz program przed pierwszą zmianą kodu. Bez pomiaru bazowego agent nie porówna wyniku zmiany.

Wykonaj każdy krok w tej kolejności:

1. Zapisz cel: metryka, jednostka, dane wejściowe, maszyna, wartość docelowa (Per.1).
2. Zbuduj mierzoną wersję jak wersję produkcyjną. C++: flagi `-O2 -g -DNDEBUG`. Python: wersja interpretera oraz wersja numpy z produkcji.
3. Zmierz cały program co najmniej 5 razy. Zapisz medianę, wynik najmniejszy oraz wynik największy.
   - C++: `perf stat -r 5 ./app`. Jeżeli narzędzie `perf` jest niedostępne, to zmierz czas przez `std::chrono::steady_clock` w programie.
   - Python oraz numpy: `python3 -m timeit -r 5 -n 100 -s "<przygotowanie>" "<wywołanie>"` dla 1 wywołania; `time python3 script.py` dla skryptu.
4. Znajdź gorący punkt przez profiler.
   - C++: `perf record -g ./app`, potem `perf report`.
   - Python: `python3 -m cProfile -s cumtime script.py`. Profil zawiera czas importu modułów, więc czytaj wiersze funkcji programu.
   - numpy: odczytaj z profilu, gdzie leży czas: w pętli Pythona albo w funkcji numpy.
5. Napisz mikrobenchmark tylko dla gorącego punktu. C++: Google Benchmark z `benchmark::DoNotOptimize(result)`. Python: `timeit.repeat(stmt, setup, number=N, repeat=5)`; porównuj wynik najmniejszy.
6. Zapisz pomiar bazowy w opisie zmiany: wartość, jednostka, liczba powtórzeń, maszyna.

### Etap 2: optymalizacja

Wykonaj każdy krok w tej kolejności. Po każdym kroku powtórz krok 3 oraz krok 5 z etapu 1.

1. Sprawdź złożoność gorącego punktu. Zmień algorytm albo strukturę danych przed pierwszą mikrooptymalizacją.
2. Ułóż dane w pamięci ciągłej. Czytaj dane w kolejności adresów (Per.19).
   - C++: `std::vector` zamiast `std::list`; struktura tablic dla pętli, która czyta 1 pole rekordu.
   - numpy: tablica w układzie C (`np.ascontiguousarray`); pętla wewnętrzna biegnie po osi o najmniejszym kroku.
3. Usuń alokację pamięci z gorącej pętli. C++: `reserve` oraz bufor wielokrotnego użytku. numpy: `np.empty` przed pętlą oraz parametr `out=`.
4. Zamień pętlę Pythona na operację wektorową numpy, na przykład `np.sum`, broadcasting, `np.einsum`. Jeżeli wektoryzacja zawodzi, to zmierz wariant w C++ przez pybind11.
5. Jeżeli `perf stat` pokazuje `branch-misses` w gorącej pętli, to zastąp rozgałęzienie obliczeniem bez skoku. Zmierz oba warianty.
6. Pokaż kompilatorowi kod gorącej pętli: definicja funkcji w nagłówku albo `-flto`. Usuń wywołanie wirtualne z gorącej pętli.
7. Sprawdź wektoryzację flagą `-fopt-info-vec-missed` (GCC). Przenieś obliczenie stałych do czasu kompilacji (Per.11).
8. Dodaj wątki dopiero po krokach od 1 do 7 włącznie. Podziel dane między wątki bez współdzielenia zapisywalnych danych (skill `cpp-concurrency`).
9. Zmierz wynik końcowy. Zapisz wartość przed zmianą, wartość po zmianie oraz jednostkę.

## Lista kontrolna do review

- Opis zmiany podaje pomiar przed zmianą oraz po zmianie: wartość, jednostka, liczba powtórzeń (Per.6).
- Benchmark mierzy build z flagą `-O2` lub `-O3`; wynik z buildu debug jest nieważny.
- Benchmark C++ przekazuje wynik do `benchmark::DoNotOptimize`; kompilator usuwa obliczenie bez użytego wyniku.
- Gorąca pętla czyta tablicę w kolejności adresów (Per.19).
- Gorąca pętla nie alokuje pamięci: brak `new`, brak `push_back` bez `reserve`, brak tymczasowego `std::string`.
- Jeżeli profil pokazuje koszt wywołania w gorącej pętli, to pętla wywołuje funkcję bez `virtual` oraz bez `std::function`.
- Zmienne atomowe, które zapisują różne wątki, leżą w osobnych liniach cache, na przykład przez `alignas(64)`.
- Python: gorąca pętla nie iteruje w Pythonie po elementach tablicy numpy.
- numpy: kod nie wywołuje `np.append` ani `np.concatenate` w pętli; tablica wyniku powstaje raz przez `np.empty`.
- numpy: typ danych jest jawny, na przykład `dtype=np.float32`; zmiana typu ma pomiar dokładności wyniku.
- Kod nie przyspiesza programu przez niezdefiniowane zachowanie; testy przechodzą z flagą `-fsanitize=undefined`.

## Pułapki

- Kompilator usuwa kod bez efektu. Benchmark bez `DoNotOptimize` mierzy pustą pętlę.
- Pierwszy przebieg działa na zimnym cache. Odrzuć wynik przebiegu rozgrzewkowego.
- Zmiana częstotliwości CPU zmienia wynik między przebiegami. Ustaw governor `performance` albo podaj rozrzut wyników.
- Moduł `timeit` wyłącza garbage collector podczas pomiaru. Kod, który tworzy obiekty, działa w programie wolniej niż w `timeit`.
- Narzut `cProfile` rośnie z liczbą wywołań funkcji. Potwierdź wynik profilu przez `timeit`.
- Mikrobenchmark ma inny stan cache niż cały program. Potwierdź zysk pomiarem całego programu (etap 1, krok 3).
- Narzut 1 wywołania funkcji numpy jest stały. Zmierz wariant wektorowy na rozmiarze danych z produkcji.

## Źródła

- Fedor G. Pikus, „The Art of Writing Efficient Programs: An advanced programmer's guide to efficient hardware utilization and compiler optimizations using C++ examples”, Packt Publishing, 2021, ISBN `9781800208117`.
- Strona wydawcy: https://www.packtpub.com/en-us/product/the-art-of-writing-efficient-programs-9781800208117 (sprawdzone 2026-11-15). Strona odrzuciła odczyt automatyczny kodem HTTP 403. Opis rozdziałów 2, 3 oraz 4 pochodzi z wyników wyszukiwania dla domeny packtpub.com.
- Opis wydawcy oraz kod przykładów: https://github.com/PacktPublishing/The-Art-of-Writing-Efficient-Programs (sprawdzone 2026-11-15).
- C++ Core Guidelines, sekcja Per, wersja z 2026-06-14: https://github.com/isocpp/CppCoreGuidelines (sprawdzone 2026-11-15).
- Dokumentacja Pythona: https://docs.python.org/3/library/timeit.html oraz https://docs.python.org/3/library/profile.html.
- Skill opiera się na wiedzy modelu i na źródłach publicznych, a nie na pełnym tekście książki.
