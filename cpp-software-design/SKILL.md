---
name: cpp-software-design
description: Design C++ code for change by managing dependencies and abstractions, following Klaus Iglberger's "C++ Software Design" (SOLID principles, value semantics, Visitor with std::variant, Strategy and Command with std::function, Type Erasure, Bridge and Pimpl, Decorator, CRTP, Observer). Use when designing or reviewing C++ class hierarchies, interfaces, extension points or module boundaries, or when the user says "software design", "design pattern", "SOLID", "type erasure", "decouple" or "replace inheritance".
---

# Projekt oprogramowania w C++ według Iglbergera

## Cel

Skill prowadzi projekt kodu C++, który przyjmuje zmiany wymagań. Iglberger opisuje projekt oprogramowania jako zarządzanie zależnościami oraz abstrakcjami. Lista funkcji programu nie stanowi projektu.

## Kiedy użyć

Użyj skillu, gdy zachodzi co najmniej 1 przypadek:

- Agent projektuje hierarchię klas, interfejs, punkt rozszerzenia lub granicę modułu.
- Agent robi review zmiany, która dodaje klasę bazową, funkcję wirtualną lub wzorzec projektowy.
- Zmiana 1 wymagania wymusza edycję plików w co najmniej 2 modułach.

## Słownik

| Termin | Definicja | Słowa zakazane |
|---|---|---|
| zależność | Relacja, w której zmiana modułu A wymusza zmianę lub ponowną kompilację modułu B. | — |
| sprzężenie sztuczne | Zależność bez powodu w wymaganiach, na przykład zależność klienta od funkcji, których klient nie wywołuje. | — |
| moduł nadrzędny | Moduł z logiką biznesową, który wywołuje moduł podrzędny przez abstrakcję. | — |
| semantyka wartości | Zasada, według której kopia obiektu jest niezależna od oryginału. | — |
| type erasure | Typ wartości, który przechowuje obiekt każdego typu spełniającego wymagania, bez wspólnej klasy bazowej, na przykład `std::function`. | — |

## Kroki przy pisaniu kodu

Wykonaj każdy krok w tej kolejności:

1. Wypisz osie zmian. Oś zmiany oznacza rodzaj zmiany, który wymagania zapowiadają: nowy typ, nowa operacja, nowa implementacja operacji.
2. Wydziel każdą oś zmiany do osobnej abstrakcji (wytyczna 2, projekt pod zmianę).
3. Wybierz kierunek rozszerzenia (wytyczna 15):
   - Jeżeli wymagania dodają typy, to użyj klasy bazowej z funkcjami wirtualnymi.
   - Jeżeli wymagania dodają operacje, to użyj `std::variant` z `std::visit` (wytyczne 16 oraz 17).
4. Podziel interfejs tak, żeby klient zależał tylko od funkcji, które klient wywołuje (wytyczna 3, zasada segregacji interfejsów).
5. Ogranicz wymagania szablonu do operacji, których szablon używa (wytyczna 3).
6. Umieść abstrakcję w module nadrzędnym (wytyczna 9, własność abstrakcji). Moduł podrzędny implementuje abstrakcję.
7. Przechowuj każdą strategię oraz każdą komendę przez wartość, na przykład w `std::function` (wytyczne 22 oraz 23).
8. Jeżeli klasa pochodna dziedziczy tylko implementację, to zastąp dziedziczenie składową (wytyczna 20, kompozycja).
9. Jeżeli typy obywają się bez wspólnej klasy bazowej, to zastąp hierarchię klasą type erasure (wytyczna 32).
10. Jeżeli zmiana implementacji wymusza ponowną kompilację klientów, to ukryj implementację za Pimpl albo za Bridge (wytyczna 28).
11. Wybierz polimorfizm czasu kompilacji (szablon, CRTP) tylko dla zbioru typów znanego w czasie kompilacji (wytyczne 26 oraz 36).
12. Napisz test dla każdej abstrakcji przez podmianę implementacji na test double.

## Lista kontrolna do review

- Klient interfejsu wywołuje każdą funkcję interfejsu, od którego klient zależy (wytyczna 3).
- Concept szablonu wymienia tylko operacje, których szablon używa (wytyczna 3).
- Moduł nadrzędny nie dołącza nagłówków modułu podrzędnego. Abstrakcja leży w module nadrzędnym (wytyczna 9).
- Każda funkcja `override` wykonuje kontrakt funkcji bazowej. Puste ciało `override` oraz wyjątek typu „not implemented” łamią zasadę podstawienia Liskov.
- Klasa, która dziedziczy publicznie, spełnia relację „jest” wobec klasy bazowej. Klasa, która dziedziczy dla ponownego użycia kodu, ma składową zamiast bazy (wytyczna 20).
- Klasa, która przyjmuje strategię, przechowuje strategię przez wartość, na przykład w `std::function`. Zgłoś surowy wskaźnik do strategii jako finding (wytyczne 22 oraz 23).
- Wizytator `std::visit` ma osobną gałąź dla każdej alternatywy wariantu (wytyczna 17).
- Klasa type erasure z własnością kopiuje obiekt wewnętrzny przez wirtualną funkcję `clone` (wytyczna 30). Przeniesienie ma `noexcept`.
- Klasa Pimpl z `std::unique_ptr<Impl>` deklaruje destruktor w nagłówku oraz definiuje destruktor w pliku `.cpp` (wytyczna 28).
- Klasa bazowa CRTP ma chroniony konstruktor lub chroniony destruktor (wytyczna 26).
- Destruktor obserwatora usuwa rejestrację obserwatora u podmiotu, na przykład przez obiekt RAII (wytyczna 25).
- Kod pobiera singleton przez parametr funkcji, a nie przez wywołanie `instance()` w ciele funkcji (wytyczna 38).

## Pułapki

- Nazwa wzorca opisuje intencję, a nie implementację z książki „Design Patterns” (GoF). Przykład: parametr `std::function` realizuje wzorzec Strategy.
- `std::variant` zamyka zbiór typów. Nowy typ wymusza zmianę każdego wizytatora oraz ponowną kompilację każdego użytkownika wariantu.
- Gałąź `auto` w wizytatorze przyjmuje nowy typ bez błędu kompilacji.
- Acyclic Visitor wykonuje `dynamic_cast` przy każdym wywołaniu (wytyczna 18). Zmierz koszt przed wyborem.
- Type erasure z własnością alokuje pamięć dla każdego obiektu. Small Buffer Optimization usuwa alokację dla obiektu, który mieści się w buforze (wytyczna 33).
- Type erasure bez własności przechowuje wskaźnik (wytyczna 34). Obiekt źródłowy MUSI żyć dłużej niż obiekt type erasure.
- Bridge oraz Pimpl dodają 1 alokację na obiekt oraz 1 pośrednie wywołanie na funkcję (wytyczna 29).

## Źródła

- Klaus Iglberger, „C++ Software Design: Design Principles and Patterns for High-Quality Software”, O'Reilly Media, 2022, ISBN `9781098113162`.
- Strona wydawcy: https://www.oreilly.com/library/view/c-software-design/9781098113155/ (sprawdzone 2026-11-15). Strona odrzuciła odczyt automatyczny kodem HTTP 403. Tytuły wytycznych 1, 2, 3, 9, 10, 20, 21, 22 oraz 32 pochodzą z wyników wyszukiwania dla tej strony.
- Kod przykładów autora: https://github.com/igl42/cpp_software_design (sprawdzone 2026-11-15). Nazwy plików podają numer wytycznej oraz temat, na przykład `G32_Type_Erasure.cpp`.
- Skill opiera się na wiedzy modelu i na źródłach publicznych, a nie na pełnym tekście książki.
