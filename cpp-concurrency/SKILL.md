---
name: cpp-concurrency
description: Write and review multithreaded C++ (threads, mutexes, condition variables, futures, atomics and memory orders, lock-based and lock-free data structures, parallel algorithms) following Anthony Williams's "C++ Concurrency in Action" (2nd edition) and the CP rules of the C++ Core Guidelines. Use when code shares data between threads, when a data race, deadlock or hang is suspected, or when the user says "concurrency", "multithreading", "thread safety", "race condition", "deadlock", "atomic" or "lock-free".
---

# Współbieżność w C++ według Williamsa

## Cel

Skill prowadzi pisanie kodu wielowątkowego w C++ oraz review kodu wielowątkowego. Celem jest kod bez data race, bez deadlocka oraz bez wątku, który przeżywa właściciela wątku.

## Kiedy użyć

Użyj skillu, gdy zachodzi co najmniej 1 przypadek:

- Kod współdzieli dane przez co najmniej 2 wątki.
- Test daje różne wyniki dla tego samego kodu lub program zawiesza pracę.
- Użytkownik pyta o wątki, blokady, zmienne atomowe lub algorytmy równoległe.

## Słownik

| Termin | Definicja | Słowa zakazane |
|---|---|---|
| data race | Stan, w którym co najmniej 2 wątki używają 1 obiektu bez synchronizacji, a co najmniej 1 wątek zapisuje obiekt. | wyścig |
| sekcja krytyczna | Fragment kodu od zajęcia mutexu do zwolnienia mutexu. | — |
| deadlock | Stan, w którym co najmniej 2 wątki czekają na mutexy, które trzymają inne wątki z tej grupy. | zakleszczenie |

## Kroki przy pisaniu kodu

Wykonaj każdy krok w tej kolejności:

1. Opisz pracę jako zadania, a nie jako wątki (CP.4). Wybierz `std::async`, pulę wątków albo algorytm z `std::execution::par`.
2. Ogranicz dane współdzielone (CP.3). Przekaż do wątku kopię danych albo `std::unique_ptr` (CP.31).
3. Zdefiniuj mutex obok chronionych danych w 1 klasie (CP.50).
4. Zajmij mutex przez obiekt RAII z nazwą (CP.20, CP.44).
5. Zajmij co najmniej 2 mutexy przez 1 obiekt `std::scoped_lock` (CP.21).
6. Skróć sekcję krytyczną (CP.43). Wywołaj nieznany kod po zwolnieniu mutexu (CP.22).
7. Czekaj na `std::condition_variable` z predykatem: `cv.wait(lock, [&] { return ready; });` (CP.42).
8. Zwróć wynik zadania przez `std::future` (CP.60). Jeżeli zadanie MUSI działać w osobnym wątku, to podaj `std::launch::async`.
9. Uruchom wątek przez `std::jthread` (C++20) albo przez klasę z `join` w destruktorze (CP.25, CP.26).
10. Zainicjalizuj leniwie obiekt współdzielony przez lokalną zmienną `static` albo `std::call_once` (CP.110).
11. Użyj `std::atomic` z domyślnym porządkiem `memory_order_seq_cst`. Słabszy porządek zapisz tylko z parą release oraz acquire w komentarzu (CP.100).
12. Uruchom testy z ThreadSanitizer (`-fsanitize=thread`). Powtórz test współbieżny w pętli, bo 1 zielony przebieg nie dowodzi braku data race.

## Lista kontrolna do review

- Każdy zapis do danych współdzielonych zachodzi pod mutexem albo przez `std::atomic` (CP.2).
- Kod nie wywołuje bezpośrednio `lock()` ani `unlock()` na mutexie (CP.20).
- Funkcja, która zajmuje co najmniej 2 mutexy, zajmuje mutexy przez 1 obiekt `std::scoped_lock` (CP.21).
- Każdy obiekt blokady ma nazwę. Obiekt tymczasowy `std::lock_guard<std::mutex>{m};` zwalnia mutex na końcu wyrażenia (CP.44).
- Każde wywołanie `wait` na `std::condition_variable` ma predykat (CP.42).
- Kod pod blokadą nie wywołuje callbacku, funkcji wirtualnej klienta ani `std::function` (CP.22).
- Żadna funkcja publiczna nie zwraca wskaźnika ani referencji do danych, które chroni mutex.
- Kod nie wywołuje `detach()` (CP.26). Wątek `std::thread` ma `join` na każdej ścieżce wyjścia, także na ścieżce wyjątku.
- Kod nie używa `volatile` do synchronizacji (CP.8).
- Każdy argument `std::memory_order` inny niż `seq_cst` ma komentarz z parą release oraz acquire.
- Lambda przekazana do wątku przechwytuje przez referencję tylko obiekty, które żyją dłużej niż wątek.
- Interfejs kolejki współbieżnej łączy sprawdzenie z działaniem w 1 funkcji, na przykład `try_pop`, zamiast pary `empty` oraz `pop`.

## Pułapki

- Konstruktor `std::thread` kopiuje każdy argument do pamięci wątku. Parametr `const T&` wskazuje kopię, a parametr `T&` wymaga `std::ref`.
- Destruktor obiektu `std::thread` w stanie `joinable` wywołuje `std::terminate`.
- Destruktor `std::future` z `std::async` czeka na koniec zadania. Wywołanie `std::async` bez zapisu wyniku działa synchronicznie.
- Domyślna polityka `std::async` zezwala na wykonanie odroczone. Wtedy zadanie startuje dopiero w `get` albo w `wait`.
- Gdy funkcja elementu rzuci wyjątek, algorytm z `std::execution::par` wywołuje `std::terminate`.
- Licznik referencji `std::shared_ptr` jest atomowy. Obiekt wskazywany przez `std::shared_ptr` wymaga osobnej synchronizacji.
- Struktura lock-free wymaga obsługi problemu ABA oraz bezpiecznego zwalniania pamięci, na przykład przez hazard pointers. Zmierz strukturę lock-free względem wersji z mutexem przed wyborem (CP.100).

## Źródła

- Anthony Williams, „C++ Concurrency in Action”, wydanie 2, Manning Publications, luty 2019, ISBN `9781617294693`.
- Strona wydawcy: https://www.manning.com/books/c-plus-plus-concurrency-in-action-second-edition (sprawdzone 2026-11-15).
- Tytuły 11 rozdziałów: https://livebook.manning.com/book/c-plus-plus-concurrency-in-action-second-edition/table-of-contents oraz strony rozdziałów od 3 do 11 włącznie w serwisie livebook.manning.com (sprawdzone 2026-11-15).
- C++ Core Guidelines, sekcja CP, wersja z 2026-06-14: https://github.com/isocpp/CppCoreGuidelines (sprawdzone 2026-11-15).
- Skill opiera się na wiedzy modelu i na źródłach publicznych, a nie na pełnym tekście książki.
