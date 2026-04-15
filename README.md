# #️⃣ Hashing Algorithms — Presentation

An interactive slide deck covering hash functions, collision resolution, open addressing, consistent hashing, cryptographic hashes, and perfect hashing. Aimed at mid-level software engineers.

## ▶ [Open Presentation](https://brendanjameslynskey.github.io/Hashing_Algorithms/index.html)

## 📄 [Markdown Version](presentation.md)

---

## Contents

| # | Topic |
|---|-------|
| 01 | Title slide |
| 02 | Hash function properties — deterministic, uniform distribution, efficiency |
| 03 | The division method |
| 04 | Multiplication method and universal hashing |
| 05 | Collision resolution — chaining |
| 06 | Open addressing — linear probing |
| 07 | Open addressing — quadratic probing |
| 08 | Open addressing — double hashing |
| 09 | Load factor and rehashing |
| 10 | Perfect hashing (FKS scheme) |
| 11 | Minimal perfect hashing |
| 12 | Consistent hashing |
| 13 | Consistent hashing — virtual nodes |
| 14 | Cryptographic hash functions |
| 15 | SHA-256 and BLAKE3 |
| 16 | Hash tables in practice (Python dict, Java HashMap, C++ unordered_map, Rust) |
| 17 | Cuckoo hashing |
| 18 | Robin Hood hashing |
| 19 | Locality-sensitive hashing |
| 20 | Applications — caches, deduplication, checksums, Bloom filters |
| 21 | Summary and further reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | append `?print-pdf` to URL |

---

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) (Monokai) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm.

---

## References

- Cormen, T.H. et al. *Introduction to Algorithms* (CLRS), 4th ed. MIT Press, 2022
- Knuth, D.E. *The Art of Computer Programming, Vol. 3: Sorting and Searching*. Addison-Wesley, 1998
- Karger, D. et al. "Consistent Hashing and Random Trees." *STOC*, 1997
- Mitzenmacher, M. & Upfal, E. *Probability and Computing*. Cambridge University Press, 2017
- Kulukundis, M. "Designing a Fast, Efficient, Cache-friendly Hash Table." CppCon 2017

## License

Educational use. Code examples provided as-is. Standards references are to publicly available documentation.
