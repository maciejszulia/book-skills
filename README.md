# book-skills

Six Agent Skills (`SKILL.md` format). Five turn C++ books into working checklists for writing and reviewing code; `math-thinking` turns four mathematics books into a checklist for solving a problem with a model, invariants and a numeric check. Each skill is a single `SKILL.md`: an English description that triggers the skill, and a body in Polish written in Controlled Polish (one meaning per sentence).

| Skill | Book |
|---|---|
| `cpp-software-design` | Klaus Iglberger, *C++ Software Design*, O'Reilly, 2022 |
| `cpp-core-guidelines` | J. Guy Davidson, Kate Gregory, *Beautiful C++: 30 Core Guidelines for Writing Clean, Safe, and Fast Code*, Addison-Wesley, 2021 |
| `efficient-programs` | Fedor G. Pikus, *The Art of Writing Efficient Programs*, Packt, 2021 |
| `cpp-concurrency` | Anthony Williams, *C++ Concurrency in Action*, 2nd edition, Manning, 2019 |
| `cpp17-features` | Bartłomiej Filipek, *C++17 in Detail*, Leanpub, 2019 |
| `math-thinking` | Deisenroth, Faisal, Ong, *Mathematics for Machine Learning*, 2020; Lehman, Leighton, Meyer, *Mathematics for Computer Science*, 2015; Boyd, Vandenberghe, *Convex Optimization*, 2004; Diestel, *Graph Theory* |

Every C++ skill has the same sections: goal, when to use it, steps for writing code, a review checklist (at least 8 items, each checkable in code), pitfalls, and sources. `efficient-programs` starts with measurement, and its measurement steps cover Python and numpy as well as C++.

## Paraphrase, not copies

The skills paraphrase ideas. They do not reproduce text from the books. Each skill is based on the model's general knowledge and on public sources: the publisher's page or public table of contents (URL and check date in the "Źródła" section), the authors' public example code, and the [C++ Core Guidelines](https://github.com/isocpp/CppCoreGuidelines). Buy the books for the full argument.

Rule IDs come from the 2026-06-14 version of the C++ Core Guidelines. The skills name each rule with a short Polish paraphrase; they do not quote rule titles.

## Install

```sh
git clone <this repository> ~/book-skills
for s in cpp-software-design cpp-core-guidelines efficient-programs cpp-concurrency cpp17-features math-thinking; do
  ln -s ~/book-skills/$s ~/.claude/skills/$s
done
```

The agent loads the skills at the start of its next session.

## License

The text of the skills is licensed under the MIT License; see [LICENSE](LICENSE).

## Author

Maciej Szulia
