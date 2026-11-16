---
name: math-thinking
description: "Math-first problem solving: model, invariants, edge cases, expected value, convexity, graph, numeric check. Use when a task compares models or variants, estimates a rate from a sample, or picks a sample size. Use it for an optimisation, a link or dependency graph, and a loop that needs a correctness argument. Triggers: \"math-thinking\", \"is this difference significant\", \"confidence interval\", \"policz to matematycznie\"."
---

# Myślenie matematyczne

Skill prowadzi agenta od problemu do wyniku, który agent sprawdził liczbą. Agent wykonuje 7 kroków i listę kontrolną przed raportem. Skill opiera się na 4 książkach z sekcji „Źródła”.

## Słownik

| Termin | Definicja | Słowa zakazane |
|---|---|---|
| model | Zapis problemu jako zmienne z jednostkami, relacje między zmiennymi i pytanie w postaci liczby. | „intuicja”. |
| niezmiennik | Warunek na stan, który po każdym przejściu jest prawdziwy, jeżeli był prawdziwy przed przejściem. | „reguła stała”. |
| przypadek skrajny | Wejście na granicy dziedziny: zbiór pusty, 1 element, wartości równe, wartość 0, granica parametru. | „corner case”. |
| przedział 95 % | Przedział z procedury, która obejmuje prawdziwą wartość w 95 % powtórzeń. | „95 % szans”. |
| sprawdzenie liczbą | Obliczenie wyniku drugą, niezależną drogą i porównanie 2 liczb z tolerancją. | „weryfikacja na oko”. |

## Kroki

Wykonaj każdy krok w tej kolejności. Każdy krok kończy się wynikiem, który agent zapisuje w notatce roboczej.

1. **Model.** Zapisz każdą zmienną z jednostką. Zapisz pytanie jako liczbę albo jako zdanie z odpowiedzią „tak” albo „nie”.
   - Wynik: 1 zdanie z pytaniem oraz lista zmiennych.
2. **Niezmienniki.** Wypisz każdy warunek, który pętla, kolejka lub protokół ma zachować. Sprawdź każdy warunek w stanie początkowym. Sprawdź każdy warunek w każdym typie przejścia, także w przejściu awaryjnym.
   - Wynik: lista niezmienników; przy każdym niezmienniku przejście, które łamie warunek, albo słowo „brak”.
3. **Przypadki skrajne.** Ułóż tabelę: przypadek skrajny, oczekiwany wynik, wynik programu. Tabela zawiera zbiór pusty, 1 element, wartości równe oraz granicę każdego parametru.
   - Wynik: tabela z co najmniej 4 wierszami.
4. **Wartość oczekiwana.** Jeżeli wynik pochodzi z próby, to podaj liczność próby, średnią i przedział 95 %. Wybierz narzędzie z listy:
   - Suma zdarzeń: liniowość wartości oczekiwanej, także dla zdarzeń zależnych.
   - Odsetek z próby: przedział Wilsona albo ograniczenie Czebyszewa z sekcji „Wzory”.
   - Stopa sukcesu z próby o liczności mniejszej niż 100: rozkład beta a posteriori z sekcji „Wzory”.
   - 2 warianty na tych samych danych: porównanie par, czyli liczba par niezgodnych w każdym kierunku.
   - Czas wykonania: mediana obok średniej.
   - Wynik: każda liczba z próby ma liczność i przedział 95 %.
5. **Wypukłość.** Jeżeli zadanie to minimalizacja, to sprawdź wypukłość celu oraz każdego ograniczenia. Jeżeli funkcja powstaje z funkcji wypukłych przez nieujemną sumę ważoną, maksimum lub złożenie z funkcją afiniczną, to funkcja jest wypukła. Problem wypukły ma każde minimum lokalne równe minimum globalnemu. Jeżeli dane mają wartości odstające, to wybierz karę $\lvert u \rvert$ zamiast kary $u^2$.
   - Wynik: zdanie „problem wypukły” albo „problem niewypukły” z regułą, która rozstrzyga.
6. **Graf.** Jeżeli obiekty mają relacje (linki, zależności, wywołania), to zapisz graf. Policz wierzchołki $n$, krawędzie $m$, składowe spójne oraz wierzchołki izolowane. Spójna składowa z $n_r$ wierzchołkami i $n_r - 1$ krawędziami jest drzewem. Dla zależności zadań sprawdź brak cyklu. Przy nieograniczonej liczbie wykonawców najkrótszy harmonogram ma tyle kroków, ile zadań ma najdłuższy łańcuch. Porównaj gęstość grup z modelem zerowym.
   - Wynik: liczby $n$, $m$, liczba składowych, liczba wierzchołków izolowanych, długość najdłuższego łańcucha.
7. **Sprawdzenie liczbą.** Policz wynik drugą drogą. Przykłady (lista niepełna):
   - Wzór zamknięty wobec symulacji Monte Carlo.
   - Nierówność (Jensen, Czebyszew, log-sum-exp) wobec wyniku programu.
   - Przypadek z co najwyżej 3 elementami, policzony ręcznie, wobec programu.

   Zapisz obliczenie jako skrypt z asercją. Liczenie w pamięci daje błędy.
   - Wynik: 2 liczby, różnica oraz tolerancja.

## Lista kontrolna

Przed raportem spełnij każdy warunek:

- [ ] Każda zmienna ma jednostkę albo zapis „bez jednostki”.
- [ ] Pytanie ma postać liczby albo zdania z odpowiedzią „tak” albo „nie”.
- [ ] Każdy niezmiennik ma sprawdzenie w każdym typie przejścia.
- [ ] Tabela przypadków skrajnych zawiera zbiór pusty oraz 1 element.
- [ ] Każda liczba z próby ma liczność próby oraz przedział 95 %.
- [ ] Porównanie 2 wariantów na tych samych danych podaje liczbę par niezgodnych.
- [ ] Stopa z próby o liczności mniejszej niż 100 ma rozkład a posteriori.
- [ ] Średni czas ma obok medianę.
- [ ] Każda minimalizacja ma zdanie o wypukłości.
- [ ] Każdy graf ma liczbę składowych spójnych oraz liczbę wierzchołków izolowanych.
- [ ] Wynik ma drugą drogę obliczenia; różnica dróg jest mniejsza niż tolerancja.
- [ ] Raport opisuje poziom ufności jako cechę procedury, a nie jako prawdopodobieństwo stałej wartości.

## Wzory

Ograniczenie Czebyszewa dla odsetka z $n$ prób:

$$\Pr\left(\left|\frac{S_n}{n} - p\right| \ge \varepsilon\right) \le \frac{1}{4 n \varepsilon^2}$$

Rozkład a posteriori stopy sukcesu po $k$ sukcesach w $N$ próbach:

$$\theta \mid k, N \sim \mathrm{Beta}(1 + k, 1 + N - k)$$

Wariancja różnicy 2 wyników:

$$\mathbb{V}[x - y] = \mathbb{V}[x] + \mathbb{V}[y] - 2\,\mathrm{Cov}[x, y]$$

Nierówność Jensena dla funkcji wypukłej $f$:

$$f(\mathbb{E} x) \le \mathbb{E} f(x)$$

Ograniczenie funkcji log-sum-exp dla $n$ składników:

$$\max_i x_i \le \log \sum_{i=1}^{n} e^{x_i} \le \max_i x_i + \log n$$

Symbole, każdy bez jednostki:
- $S_n$ to liczba sukcesów w $n$ próbach.
- $p$ to prawdziwa stopa sukcesu.
- $\varepsilon$ to połowa szerokości przedziału.
- $\theta$ to stopa sukcesu.
- $k$ to liczba sukcesów w $N$ próbach.
- $x$ oraz $y$ to wyniki 2 wariantów na tym samym elemencie próby.

## Przykład

Repo `pk-rewriter` podaje miarę M równą 41 z 44 tekstów dla 2 modeli. Krok 4 daje przedział Wilsona od 81,8 % do 97,7 % włącznie. Porównanie par daje liczbę par niezgodnych równą 1 w każdym kierunku. Wniosek: 44 teksty nie rozróżniają tych 2 modeli.

## Źródła

- M. P. Deisenroth, A. A. Faisal, C. S. Ong, „Mathematics for Machine Learning”, 2020: podrozdział 6.4.4 (wariancja różnicy), podrozdział 6.6.1 (rozkład beta). PDF: https://mml-book.github.io/book/mml-book.pdf.
- E. Lehman, F. T. Leighton, A. R. Meyer, „Mathematics for Computer Science”, 2015: Definition 5.4.5 (niezmiennik), Theorem 9.5.4 (porządek topologiczny), Corollary 18.5.3 (liniowość), Theorem 19.4.1 (próbkowanie). PDF: https://ocw.mit.edu/courses/6-042j-mathematics-for-computer-science-spring-2015/mit6_042js15_textbook2.pdf.
- S. Boyd, L. Vandenberghe, „Convex Optimization”, 2004: podrozdział 3.1.5 (log-sum-exp), podrozdział 3.1.8 (Jensen), podrozdział 3.2.1 (sumy nieujemne), podrozdział 6.1.2 (kary). PDF: https://web.stanford.edu/~boyd/cvxbook/bv_cvxbook.pdf.
- R. Diestel, „Graph Theory”, wydanie „Free Preview”: rozdział 1 (składowe, drzewa), Lemma 11.1.4 (wartość oczekiwana w $G(n, p)$). Strona: https://diestel-graph-theory.com/basic.html.
