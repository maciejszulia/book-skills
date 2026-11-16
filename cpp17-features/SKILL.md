---
name: cpp17-features
description: Use C++17 language and library features correctly (structured bindings, if and switch initializers, inline variables, if constexpr, fold expressions, std::optional, std::variant, std::any, std::string_view, std::filesystem, std::from_chars, parallel algorithms, [[nodiscard]]) following Bartłomiej Filipek's "C++17 in Detail". Use when writing or reviewing C++17 code, when modernizing C++11 or C++14 code, or when the user says "C++17", "string_view", "optional", "variant", "if constexpr" or "structured bindings".
---

# C++17 w praktyce według Filipka

## Cel

Skill prowadzi użycie elementów C++17 w nowym kodzie oraz przy modernizacji kodu C++11 i C++14. Każdy krok zastępuje starszy wzorzec konstrukcją C++17. Sekcja „Pułapki” podaje błędy nowych konstrukcji.

## Kiedy użyć

Użyj skillu, gdy zachodzi co najmniej 1 przypadek:

- Projekt buduje kod z flagą `-std=c++17` lub z nowszym standardem.
- Agent modernizuje kod C++11 lub kod C++14.
- Agent robi review kodu z `std::optional`, `std::variant`, `std::string_view` lub `if constexpr`.

## Słownik

| Termin | Definicja | Słowa zakazane |
|---|---|---|
| alternatywa | Typ z listy argumentów szablonu `std::variant`. | wariant (dla typu z listy) |
| wizytator | Obiekt funkcyjny, który `std::visit` wywołuje dla bieżącej alternatywy. | visitor |
| gałąź odrzucona | Gałąź `if constexpr`, której warunek ma wartość `false` w czasie kompilacji. | — |

## Kroki przy pisaniu kodu

Wykonaj każdy krok w tej kolejności:

1. Sprawdź standard w systemie budowania: `CMAKE_CXX_STANDARD 17` lub flaga `-std=c++17`. Sprawdź wsparcie kompilatora dla nagłówków `<filesystem>`, `<charconv>` oraz `<execution>`.
2. Zwróć wynik opcjonalny jako `std::optional<T>` zamiast wskaźnika albo flagi `bool`.
3. Zastąp `union` oraz hierarchię z zamkniętym zbiorem typów przez `std::variant` z `std::visit`.
4. Przyjmij tekst tylko do odczytu jako `std::string_view` przekazany przez wartość.
5. Zastąp `std::enable_if` oraz tag dispatch instrukcją `if constexpr` (rozdział 19 książki).
6. Zastąp rekurencję szablonów variadic wyrażeniem fold, na przykład `(args + ...)`.
7. Rozpakuj `std::pair`, `std::tuple` lub strukturę przez structured binding: `for (const auto& [key, value] : map)`.
8. Ogranicz zasięg zmiennej pomocniczej przez `if` z inicjalizatorem: `if (auto it = m.find(k); it != m.end())`.
9. Oznacz atrybutem `[[nodiscard]]` każdą funkcję, której zignorowany wynik oznacza błąd (rozdział 18 książki).
10. Zdefiniuj stałą klasy w nagłówku jako `inline static constexpr`.
11. Jeżeli kod potrzebuje konwersji liczb bez alokacji oraz bez locale, to użyj `std::from_chars` oraz `std::to_chars` (rozdział 11 książki).
12. Użyj `std::filesystem` zamiast API systemu. Obsłuż błąd przez przeciążenie z `std::error_code` albo przez wyjątek `std::filesystem::filesystem_error`.
13. Uruchom algorytm z `std::execution::par` dopiero po pomiarze (skill `efficient-programs`). W GCC biblioteka libstdc++ wykonuje algorytmy równoległe przez bibliotekę TBB.

## Lista kontrolna do review

- Żaden `std::string_view` nie wskazuje obiektu tymczasowego, na przykład `std::string_view sv = a + b;` dla `std::string a, b`.
- Kod nie przekazuje `std::string_view::data()` do funkcji, która wymaga tekstu zakończonego zerem.
- Pole klasy typu `std::string_view` ma komentarz, który nazywa właściciela tekstu oraz czas życia tekstu.
- Kod czyta `std::optional` przez `*opt` tylko po sprawdzeniu `has_value()`. Funkcja `value()` rzuca `std::bad_optional_access`.
- Wizytator `std::visit` ma osobną gałąź dla każdej alternatywy.
- Wzorzec `overloaded` ma w C++17 regułę dedukcji `template<class... Ts> overloaded(Ts...) -> overloaded<Ts...>;`.
- Typ `std::variant`, który program tworzy konstruktorem domyślnym, ma na początku alternatywę z konstruktorem domyślnym, na przykład `std::monostate`.
- Structured binding, przez które kod zmienia elementy, ma postać `auto&`. Postać `auto` kopiuje obiekt.
- Wynik `std::from_chars` ma sprawdzone pole `ec`: `auto [ptr, ec] = std::from_chars(first, last, value);`.
- Każda funkcja, której zignorowany wynik oznacza błąd, ma atrybut `[[nodiscard]]`.
- Jeżeli gałąź odrzucona zawiera kod niepoprawny dla danego typu, to `if constexpr` stoi w szablonie, a warunek zależy od parametru szablonu.
- Kod nie zawiera elementów usuniętych w C++17. Przykłady (lista niepełna): `std::auto_ptr`, `std::random_shuffle`, słowo `register`, `throw(X)` w deklaracji funkcji, trigrafy.

## Pułapki

- `std::string_view` wskazuje tekst bez własności. Zniszczenie źródła unieważnia `std::string_view`.
- Gałąź `auto` w wizytatorze przyjmuje nową alternatywę bez błędu kompilacji.
- Warunek `if (opt)` dla `std::optional<bool>` sprawdza obecność wartości, a nie wartość.
- Gdy funkcja elementu rzuci wyjątek, algorytm z `std::execution::par` wywołuje `std::terminate`.
- `std::any` alokuje pamięć dla obiektu, który przekracza bufor wewnętrzny. Jeżeli zbiór typów jest zamknięty, to wybierz `std::variant`.
- C++17 ustala kolejność obliczeń tylko dla wybranych wyrażeń, na przykład `a << b` oraz `a = b` (rozdział 3 książki). Kolejność obliczania argumentów funkcji pozostaje nieustalona.

## Źródła

- Bartłomiej Filipek, „C++17 in Detail: Learn the Exciting Features of The New C++ Standard!”, Leanpub, wersja z 2019-09-12.
- Strona wydawcy ze spisem treści: https://leanpub.com/cpp17indetail (sprawdzone 2026-11-15).
- Skill opiera się na wiedzy modelu i na źródłach publicznych, a nie na pełnym tekście książki.
