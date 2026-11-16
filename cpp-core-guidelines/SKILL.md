---
name: cpp-core-guidelines
description: Apply the 30 C++ Core Guidelines that Guy Davidson and Kate Gregory selected in "Beautiful C++" (ownership, RAII, const, scoped enums, small scopes, struct return values, constexpr, concepts) when writing or reviewing C++ code. Use when writing new C++ code, when reviewing a C++ diff or pull request, or when the user says "core guidelines", "Beautiful C++", "clean C++" or names a rule ID such as F.21 or ES.22.
---

# C++ Core Guidelines według „Beautiful C++”

## Cel

Skill prowadzi pisanie kodu C++ oraz review kodu C++ według 30 reguł C++ Core Guidelines. Davidson i Gregory wybrali 30 reguł do książki „Beautiful C++”. Każdy punkt listy kontrolnej podaje identyfikator reguły oraz tytuł reguły z wersji wytycznych z 2026-06-14.

## Kiedy użyć

Użyj skillu, gdy zachodzi co najmniej 1 przypadek:

- Agent pisze nowy kod C++.
- Agent robi review zmiany w kodzie C++.
- Użytkownik podaje identyfikator reguły, na przykład F.21.

## Słownik

| Termin | Definicja | Słowa zakazane |
|---|---|---|
| zasób | Obiekt systemu, który program MUSI zwolnić po użyciu: pamięć, plik, gniazdo sieciowe, mutex. | — |
| właściciel | Obiekt, którego destruktor zwalnia zasób. | posiadacz |
| RAII | Wzorzec, w którym konstruktor zajmuje zasób, a destruktor zwalnia zasób. | — |

## Kroki przy pisaniu kodu

Wykonaj każdy krok w tej kolejności:

1. Ustal właściciela każdego zasobu przed napisaniem ciała funkcji. Typ `std::unique_ptr` oznacza właściciela. Typ `T*` oraz typ `T&` oznaczają dostęp bez własności.
2. Opakuj każdy zasób w typ RAII z biblioteki standardowej albo we własną klasę RAII.
3. Zadeklaruj każdą zmienną w miejscu, w którym zmienna dostaje wartość. Dodaj `const` do każdej zmiennej, której wartość pozostaje stała.
4. Jeżeli funkcja ma co najmniej 2 wyniki, to zwróć strukturę z nazwanymi polami. Po stronie wywołania rozpakuj strukturę przez structured binding.
5. Zapisz każde wyliczenie jako `enum class`. Zapisz każdą stałą czasu kompilacji jako `constexpr`.
6. Ogranicz każdy parametr szablonu przez concept (C++20).
7. Włącz ostrzeżenia `-Wall -Wextra -Wpedantic` oraz clang-tidy z grupą `cppcoreguidelines-*`.
8. Jeżeli kod narusza regułę z ważnego powodu, to zamknij naruszenie w 1 funkcji. Zapisz powód w komentarzu (reguła I.30).

## Lista kontrolna do review

Sprawdź w kodzie każdy punkt. Zgłoś każdy niespełniony punkt jako finding z identyfikatorem reguły.

Grupa 1, spory o drobiazgi (sekcja 1 książki):

- **P.2** (trzymaj się standardu ISO C++): kod przechodzi kompilację z flagą `-pedantic-errors`; każde rozszerzenie kompilatora stoi w 1 nagłówku za makrem.
- **F.51** (argument domyślny zamiast przeciążenia): zestaw przeciążeń, które różnią się tylko końcowym parametrem, ma postać 1 funkcji z argumentem domyślnym.
- **C.45** (stałe wartości pól w inicjalizatorach pól zamiast w konstruktorze domyślnym): żaden konstruktor domyślny nie przypisuje samych stałych; stałe stoją przy deklaracji pól.
- **C.131** (bez trywialnych getterów i setterów): para getter i setter bez niezmiennika (invariant) ma postać publicznego pola struktury.
- **ES.10** (1 nazwa w 1 deklaracji): każda deklaracja wprowadza dokładnie 1 nazwę; wyjątek stanowi structured binding.
- **NR.2** (wczesny `return` jest dozwolony): funkcja kończy przypadek brzegowy wczesnym `return` zamiast zmiennej wyniku w zagnieżdżonych `if`.

Grupa 2, ochrona przed własnymi błędami (sekcja 2 książki):

- **P.11** (nieporządna konstrukcja zamknięta w 1 miejscu): rzutowania, arytmetyka wskaźników oraz wywołania API C leżą w 1 funkcji opakowującej.
- **I.23** (mało parametrów funkcji): funkcja ma mniej niż 4 parametry, zgodnie z tekstem reguły; funkcja z co najmniej 4 parametrami przyjmuje strukturę parametrów.
- **I.26** (podzbiór C na granicy ABI między kompilatorami): funkcja `extern "C"` na granicy biblioteki przyjmuje tylko typy C; wyjątek C++ zostaje po stronie biblioteki.
- **C.47** (inicjalizacja pól w kolejności deklaracji): lista inicjalizacyjna konstruktora wymienia pola w kolejności deklaracji; flaga `-Wreorder` jest włączona.
- **CP.3** (ograniczone współdzielenie danych do zapisu): dane współdzielone przez co najmniej 2 wątki są `const`; zapisywalne dane współdzielone mają 1 właściciela oraz 1 mutex.
- **T.120** (metaprogramowanie szablonowe tylko z konieczności): rekurencja szablonów ma komentarz z powodem, dla którego `constexpr` albo `if constexpr` nie wystarcza.

Grupa 3, praktyki do porzucenia (sekcja 3 książki):

- **I.11** (przekazanie własności bez surowego wskaźnika, bez referencji): funkcja, która przekazuje własność, przyjmuje lub zwraca `std::unique_ptr`; każdy `T*` w interfejsie oznacza brak własności.
- **I.3** (bez singletonów): kod nie zawiera klasy z funkcją statyczną `instance()`; zależność przechodzi przez parametr konstruktora.
- **C.90** (kopiowanie przez operacje typu, bez funkcji pamięci z C): kod wywołuje `memset` lub `memcpy` tylko na typach trywialnie kopiowalnych.
- **ES.50** (rzutowanie zostawia `const`): kod zawiera `const_cast` tylko przy wywołaniu starego API, które przyjmuje wskaźnik bez `const`; komentarz podaje nazwę starego API.
- **E.28** (obsługa błędów bez stanu globalnego typu `errno`): funkcja zgłasza błąd wyjątkiem lub typem wyniku, na przykład `std::optional`, a nie zmienną globalną.
- **SF.7** (bez `using namespace` w zasięgu globalnym nagłówka): żaden plik nagłówkowy nie zawiera `using namespace` w zasięgu globalnym.

Grupa 4, nowe elementy języka (sekcja 4 książki):

- **F.21** (więcej niż 1 wynik funkcji jako struktura): funkcja z co najmniej 2 wynikami zwraca strukturę z nazwanymi polami, a nie parametry wyjściowe.
- **Enum.3** (`enum class` zamiast zwykłego `enum`): każde nowe wyliczenie ma postać `enum class`.
- **ES.5** (zasięg ograniczony do użycia): zmienna żyje w najmniejszym bloku, który obejmuje każde użycie zmiennej; `if` z inicjalizatorem zamyka zasięg zmiennej pomocniczej.
- **Con.5** (`constexpr` dla wartości znanej przy kompilacji): stała znana w czasie kompilacji ma `constexpr` zamiast `#define`.
- **T.1** (szablon podnosi poziom abstrakcji kodu): 2 funkcje, które różnią się tylko typem argumentu, mają postać 1 szablonu.
- **T.10** (concept dla każdego parametru szablonu): każdy parametr szablonu ma concept, na przykład `std::integral auto` (C++20).

Grupa 5, dobre nawyki domyślne (sekcja 5 książki):

- **P.4** (statyczne bezpieczeństwo typów jako cel): kod nie zawiera rzutowań w stylu C, `void*` w interfejsie ani `union` bez znacznika typu; `std::variant` zastępuje `union`.
- **P.10** (dane niezmienne przed danymi zmiennymi): zmienna bez zmiany po inicjalizacji ma `const`; parametr przekazany przez referencję do odczytu ma `const&`.
- **I.30** (naruszenie reguły zamknięte w 1 miejscu): każde naruszenie reguły leży w 1 funkcji z komentarzem, który podaje powód naruszenia.
- **ES.22** (deklaracja zmiennej dopiero z wartością początkową): każda zmienna dostaje wartość w deklaracji.
- **Per.7** (projekt, który pozwala na optymalizację): interfejs przekazuje typ elementów, na przykład przez szablon albo `std::span<T>`, zamiast `void*` z rozmiarem elementu.
- **E.6** (RAII przeciw wyciekom): żaden zasób nie zależy od ręcznego `delete`, `fclose` ani `unlock`; destruktor właściciela zwalnia zasób.

## Pułapki

- Tytuły reguł w książce z 2021 r. różnią się od tytułów w wersji wytycznych z 2026-06-14. Przykład: spis treści książki podaje dla F.21 strukturę albo krotkę, a wersja z 2026 r. zaleca strukturę.
- Reguła NR.2 należy do sekcji NR. Sekcja NR opisuje reguły, które wytyczne odrzucają jako mit.
- Reguła C.131 dotyczy tylko pary bez logiki. Getter z obliczeniem albo setter z walidacją spełnia regułę C.131.
- Reguła T.10 wymaga C++20. W kodzie C++17 zastąp concept warunkiem `static_assert` w ciele szablonu.
- Jeżeli clang-tidy z grupą `cppcoreguidelines-*` zgłosi naruszenie, to potwierdź naruszenie w kodzie przed wpisaniem do review.
- Reguła I.30 zezwala na naruszenie reguły. Zgłoś naruszenie bez komentarza z powodem jako finding.

## Źródła

- J. Guy Davidson, Kate Gregory, „Beautiful C++: 30 Core Guidelines for Writing Clean, Safe, and Fast Code”, Addison-Wesley Professional, 2021, ISBN `9780137647842`.
- Strona wydawcy ze spisem treści: https://www.informit.com/store/beautiful-c-plus-plus-30-core-guidelines-for-writing-9780137647842 (sprawdzone 2026-11-15).
- C++ Core Guidelines, wersja z 2026-06-14: https://github.com/isocpp/CppCoreGuidelines (sprawdzone 2026-11-15). Identyfikatory reguł pochodzą z tej wersji. Nazwa reguły w nawiasie to parafraza autora skillu, a nie cytat tytułu.
- Skill opiera się na wiedzy modelu i na źródłach publicznych, a nie na pełnym tekście książki.
