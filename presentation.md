# Hashing Algorithms

**Computer Science Fundamentals Series**

Hash functions · Collision resolution · Open addressing · Consistent hashing · Cryptographic hashes · Perfect hashing

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [Hash Function Properties](#slide-02--hash-function-properties)
2. [The Division Method](#slide-03--the-division-method)
3. [Multiplication Method & Universal Hashing](#slide-04--multiplication-method--universal-hashing)
4. [Collision Resolution -- Chaining](#slide-05--collision-resolution--chaining)
5. [Open Addressing -- Linear Probing](#slide-06--open-addressing--linear-probing)
6. [Open Addressing -- Quadratic Probing](#slide-07--open-addressing--quadratic-probing)
7. [Open Addressing -- Double Hashing](#slide-08--open-addressing--double-hashing)
8. [Load Factor & Rehashing](#slide-09--load-factor--rehashing)
9. [Perfect Hashing](#slide-10--perfect-hashing)
10. [Minimal Perfect Hashing](#slide-11--minimal-perfect-hashing)
11. [Consistent Hashing](#slide-12--consistent-hashing)
12. [Consistent Hashing -- Virtual Nodes](#slide-13--consistent-hashing--virtual-nodes)
13. [Cryptographic Hash Functions](#slide-14--cryptographic-hash-functions)
14. [SHA-256 & BLAKE3](#slide-15--sha-256--blake3)
15. [Hash Tables in Practice](#slide-16--hash-tables-in-practice)
16. [Cuckoo Hashing](#slide-17--cuckoo-hashing)
17. [Robin Hood Hashing](#slide-18--robin-hood-hashing)
18. [Locality-Sensitive Hashing](#slide-19--locality-sensitive-hashing)
19. [Applications](#slide-20--applications)
20. [Summary & Further Reading](#slide-21--summary--further-reading)

---

## Slide 02 -- Hash Function Properties

### What is a hash function?

A hash function `h(k)` maps keys from a universe `U` to a fixed-size set of integers `{0, 1, ..., m-1}`, where `m` is the table size.

### Desirable properties

- **Deterministic** -- the same key always produces the same hash value
- **Uniform distribution** -- keys should be spread as evenly as possible across the output range to minimise collisions
- **Efficiency** -- computing the hash should be `O(1)` or near-constant time relative to key size
- **Avalanche effect** -- a small change in the input should produce a large change in the output
- **Minimised collisions** -- different keys should map to different indices as often as possible

> A *perfect* hash function maps `n` keys into `n` or more slots with zero collisions. In general, the pigeonhole principle guarantees collisions when `|U| > m`.

---

## Slide 03 -- The Division Method

### Formula

`h(k) = k mod m`

The simplest hash function. The key `k` is divided by the table size `m`, and the remainder is the hash value.

### Choosing m

- **Avoid powers of 2** -- `k mod 2^p` uses only the lowest `p` bits, ignoring high-order information
- **Avoid powers of 10** -- same issue for decimal-encoded keys
- **Prefer primes** -- a prime `m` not close to a power of 2 gives good distribution
- **Example:** for ~2000 slots, `m = 2003` (prime) is better than `m = 2048`

### Example

```
Keys: 56, 72, 129, 183, 245
m = 7

h(56)  = 56  mod 7 = 0
h(72)  = 72  mod 7 = 2
h(129) = 129 mod 7 = 3
h(183) = 183 mod 7 = 1
h(245) = 245 mod 7 = 0  ← collision with 56
```

---

## Slide 04 -- Multiplication Method & Universal Hashing

### Multiplication method

`h(k) = ⌊m × (k × A mod 1)⌋`

Where `A` is a constant in `(0, 1)`. Knuth recommends `A ≈ (√5 − 1) / 2 ≈ 0.6180339887`.

- Works well with **any** table size `m` -- no need for primes
- Extracts information from all bits of the key
- Easy to implement with fixed-point arithmetic

### Universal hashing

Choose `h` **randomly** from a family of hash functions at runtime. A family `H` is *universal* if for any two distinct keys `x ≠ y`:

`Pr[h(x) = h(y)] ≤ 1/m` when `h` is chosen uniformly from `H`

**Carter-Wegman family:**

`h_{a,b}(k) = ((a × k + b) mod p) mod m`

Where `p` is a prime larger than the universe, `a ∈ {1, ..., p-1}`, `b ∈ {0, ..., p-1}`.

> Universal hashing guarantees `O(1)` expected-time lookups regardless of the input distribution -- it defeats adversarial inputs.

---

## Slide 05 -- Collision Resolution -- Chaining

### How it works

Each table slot holds a pointer to a linked list (or other collection). All keys that hash to the same index are stored in that list.

```
Index 0: [56] → [245] → ∅
Index 1: [183] → ∅
Index 2: [72] → ∅
Index 3: [129] → ∅
Index 4: ∅
Index 5: ∅
Index 6: ∅
```

### Performance

| Operation | Average case | Worst case |
|-----------|-------------|------------|
| Search | `O(1 + α)` | `O(n)` |
| Insert | `O(1)` | `O(1)` |
| Delete | `O(1 + α)` | `O(n)` |

Where `α = n/m` is the load factor (number of elements / table size).

### Strengths and weaknesses

- Simple to implement; deletion is straightforward
- Load factor can exceed 1 -- table never "fills up"
- Cache-unfriendly due to pointer chasing across heap-allocated nodes
- Extra memory for pointers (typically 8 bytes per entry on 64-bit systems)

---

## Slide 06 -- Open Addressing -- Linear Probing

### Concept

All elements stored directly in the table array. On collision, probe the next slot in a fixed sequence until an empty slot is found.

### Probe sequence

`h(k, i) = (h'(k) + i) mod m` for `i = 0, 1, 2, ...`

### Example

```
Insert 56, 72, 129, 183, 245 into m=7 using h(k) = k mod 7

h(56)  = 0 → slot 0 ✓
h(72)  = 2 → slot 2 ✓
h(129) = 3 → slot 3 ✓
h(183) = 1 → slot 1 ✓
h(245) = 0 → slot 0 full → slot 1 full → slot 2 full
                          → slot 3 full → slot 4 ✓
```

### Primary clustering

Contiguous blocks of occupied slots grow and merge, increasing average probe lengths. A cluster of size `k` has probability `(k+1)/m` of growing on the next insert.

> Linear probing is extremely cache-friendly -- sequential memory access dominates performance. With a good hash function and `α < 0.7`, it outperforms chaining in practice.

---

## Slide 07 -- Open Addressing -- Quadratic Probing

### Probe sequence

`h(k, i) = (h'(k) + c₁·i + c₂·i²) mod m`

Common choice: `c₁ = 0, c₂ = 1`, giving `h(k, i) = (h'(k) + i²) mod m`.

### Advantages over linear probing

- Eliminates **primary clustering** -- probes jump over occupied runs
- Still has reasonable cache performance for small probe distances

### Secondary clustering

Keys that hash to the same initial slot follow the same probe sequence. This is *secondary clustering* -- less severe than primary clustering but still suboptimal.

### Caveat

Quadratic probing does **not** guarantee visiting all slots. To guarantee a full cycle:
- `m` must be prime, **or**
- `m` must be a power of 2 with `c₁ = c₂ = 1/2`

> In practice, secondary clustering is rarely a bottleneck. Quadratic probing is a solid middle ground between linear probing's cache friendliness and double hashing's uniform distribution.

---

## Slide 08 -- Open Addressing -- Double Hashing

### Probe sequence

`h(k, i) = (h₁(k) + i × h₂(k)) mod m`

Two independent hash functions. Each key gets its own step size, so different keys that collide on `h₁` almost certainly diverge on subsequent probes.

### Requirements for h₂

- `h₂(k) ≠ 0` for all `k` -- otherwise the probe sequence never advances
- `h₂(k)` must be **coprime** to `m` -- ensures the full table is probed before repeating
- Common choice: `h₂(k) = 1 + (k mod (m - 1))` with `m` prime

### Performance comparison

| Method | Expected probes (unsuccessful) | Clustering? |
|--------|-------------------------------|-------------|
| Linear probing | `½(1 + 1/(1 − α)²)` | Primary |
| Quadratic probing | `−ln(1 − α) / α` (approx) | Secondary |
| Double hashing | `1/(1 − α)` | None |

> Double hashing gives the closest approximation to *uniform probing* (the theoretical ideal). The cost is two hash computations per probe instead of one.

---

## Slide 09 -- Load Factor & Rehashing

### Load factor α = n / m

| α range | Behaviour |
|---------|-----------|
| `0.0 – 0.5` | Fast lookups, wasted memory |
| `0.5 – 0.7` | Good balance for open addressing |
| `0.7 – 0.9` | Probe lengths grow noticeably; chaining still works well |
| `> 1.0` | Only possible with chaining; performance degrades linearly |

### Rehashing

When `α` exceeds a threshold, allocate a new table (typically `2m` or next prime > `2m`), then re-insert every element.

```
Rehash triggered: α > 0.75
1. Allocate new table of size 2m
2. For each entry in old table:
     compute new_index = h(key) mod 2m
     insert into new table
3. Free old table
```

### Amortised cost

Each insert is `O(1)` amortised. A rehash copies all `n` elements (`O(n)`), but it happens only after `Θ(n)` insertions -- spreading the cost.

> Most real implementations also **shrink** the table when `α` drops below a lower threshold (e.g., 0.25) to reclaim memory.

---

## Slide 10 -- Perfect Hashing

### Definition

A hash function that maps `n` keys into a table with **zero collisions**. Works when the key set is known in advance (static dictionary).

### FKS scheme (Fredman, Komlos, Szemeredi, 1984)

Two-level hashing with `O(n)` space and `O(1)` worst-case lookup.

```
Level 1: h₁ maps n keys into m = n slots
         Some slots have collisions (bucket of size sⱼ)

Level 2: For each bucket j with sⱼ keys,
         use a secondary table of size sⱼ²
         Choose h₂ⱼ from a universal family
         → guaranteed collision-free (birthday bound)
```

### Space analysis

The total space across all secondary tables is `O(n)` in expectation when `m = n` at level 1. The `sⱼ²` sizes sound wasteful, but the sum of squares stays linear because the level-1 hash distributes keys well.

> Perfect hashing is used in compilers (keyword recognition), network routers (lookup tables), and read-only dictionaries where build time is amortised over millions of lookups.

---

## Slide 11 -- Minimal Perfect Hashing

### Definition

A perfect hash function where the table size equals exactly `n` -- every slot is occupied, zero waste, zero collisions.

`h: S → {0, 1, ..., n-1}` is a bijection

### Construction algorithms

| Algorithm | Build time | Bits per key | Lookup time |
|-----------|-----------|-------------|-------------|
| **CHD** (Belazzougui et al.) | `O(n)` | ~2.07 | `O(1)` |
| **BBHash** | `O(n)` | ~3.0 | `O(1)` |
| **RecSplit** | `O(n)` | ~1.56 (near optimal) | `O(1)` |
| **PTHash** | `O(n)` | ~2.2 | `O(1)`, fast constants |

### Practical use cases

- **Static key-value stores** -- database index for immutable data
- **Language model vocabularies** -- map tokens to embedding indices
- **Genome indexing** -- map k-mers to positions in a reference
- **Content-addressable storage** -- deduplicate by content hash

> The theoretical lower bound is ~1.44 bits per key. Modern algorithms approach this limit while remaining practical for billions of keys.

---

## Slide 12 -- Consistent Hashing

### The problem

Standard `hash(key) mod n` breaks when `n` (number of servers) changes -- nearly all keys remap, causing a *rehashing storm*.

### The ring

Map both keys and servers onto a circular hash space `[0, 2³²)`.

```
         0
        / \
   S₃ ●     ● S₁
      |       |
  k₅ ○   k₁ ○
      |       |
   S₂ ●     ○ k₂
        \ /
       k₃ ○
```

Each key is assigned to the **next server clockwise** on the ring.

### Adding or removing a node

When server `S₄` joins, only the keys between `S₃` and `S₄` (the arc it now owns) need to move. All other key-to-server mappings remain unchanged.

- **Keys remapped:** `O(K/n)` on average (where `K` = total keys, `n` = servers)
- Contrast: `hash mod n` remaps `O(K)` keys

> Consistent hashing was introduced by Karger et al. (1997) and became the backbone of distributed caches and storage: Memcached, DynamoDB, Cassandra, Akamai CDN.

---

## Slide 13 -- Consistent Hashing -- Virtual Nodes

### The balance problem

With few physical nodes, the ring partitions are uneven. One server may own 60% of the key space while another owns 5%.

### Virtual nodes (vnodes)

Each physical server is represented by `v` virtual nodes spread across the ring. More vnodes means more uniform distribution.

```
Physical server S₁ → vnodes: S₁_0, S₁_1, S₁_2, ...
Physical server S₂ → vnodes: S₂_0, S₂_1, S₂_2, ...
```

### Benefits

- **Load balance** -- with `v = 150`, standard deviation of load drops below 5%
- **Heterogeneous capacity** -- assign more vnodes to more powerful servers
- **Smoother rebalancing** -- when a node leaves, its vnodes scatter load across many surviving nodes rather than dumping everything on one neighbour

### Implementations

| System | Ring mechanism |
|--------|--------------|
| **DynamoDB** | Consistent hashing with virtual nodes and preference lists |
| **Cassandra** | Token ring with vnodes (default 256 per node) |
| **Riak** | Ring of 64/128/256 partitions with ownership transfer |
| **Akka Cluster** | Consistent hash router with virtual nodes |

---

## Slide 14 -- Cryptographic Hash Functions

### Properties beyond standard hashing

| Property | Description |
|----------|------------|
| **Pre-image resistance** | Given `h`, infeasible to find any `m` such that `H(m) = h` |
| **Second pre-image resistance** | Given `m₁`, infeasible to find `m₂ ≠ m₁` with `H(m₁) = H(m₂)` |
| **Collision resistance** | Infeasible to find any `m₁ ≠ m₂` with `H(m₁) = H(m₂)` |
| **Avalanche effect** | Flipping one input bit changes ~50% of output bits |
| **Deterministic** | Same input always gives the same output |
| **Fixed output size** | Arbitrary-length input maps to a fixed-length digest |

### Common algorithms

| Algorithm | Output size | Status |
|-----------|-----------|--------|
| MD5 | 128 bits | Broken -- collisions found in seconds |
| SHA-1 | 160 bits | Deprecated -- practical collision attack (SHAttered, 2017) |
| SHA-256 | 256 bits | Current standard; widely used |
| SHA-3 | 224-512 bits | NIST standard (Keccak); different internal structure |
| BLAKE3 | 256 bits | Fastest modern hash; tree-based parallelism |

---

## Slide 15 -- SHA-256 & BLAKE3

### SHA-256

Merkle-Damgard construction. Processes input in 512-bit blocks through 64 rounds of mixing.

- **Output:** 256 bits (32 bytes, 64 hex characters)
- **Used in:** TLS certificates, Bitcoin proof-of-work, Git commit hashes, JWT signatures
- **Throughput:** ~500 MB/s on modern x86 with hardware extensions (SHA-NI)

```
SHA-256("hello") =
2cf24dba5fb0a30e26e83b2ac5b9e29e
1b161e5c1fa7425e7304362938b62494
```

### BLAKE3

Tree-based Merkle construction. Unlimited parallelism across chunks.

- **Output:** 256 bits (extendable to any length)
- **Speed:** ~5 GB/s single-threaded on x86; scales linearly with cores
- **Used in:** Bao (verified streaming), content-addressable storage, build systems (Bazel remote cache)

### When to use which

| Use case | Recommendation |
|----------|---------------|
| Interoperability / standards compliance | SHA-256 |
| Maximum throughput (checksums, dedup) | BLAKE3 |
| Password storage | **Neither** -- use Argon2, bcrypt, or scrypt |

---

## Slide 16 -- Hash Tables in Practice

### Python dict (CPython 3.6+)

- Open addressing with **pseudo-random probing** (perturbed by higher-order hash bits)
- Compact layout: keys/values stored in a dense insertion-order array; hash table holds indices
- Resize at `α > 2/3`; table size always a power of 2
- Guaranteed insertion-order iteration since Python 3.7

### Java HashMap

- **Chaining** with linked lists; degrades to **balanced tree** (red-black) when a bucket exceeds 8 entries (treeify threshold)
- Default load factor: `0.75`; capacity always a power of 2
- Key must implement `hashCode()` and `equals()`

### C++ std::unordered_map

- Chaining (bucket array of linked lists) per the standard
- Load factor threshold: `1.0` by default
- Notoriously cache-unfriendly; real-world code often replaces with **Abseil flat_hash_map** (open addressing, SIMD-accelerated probing) or **Robin Hood** variants

### Rust HashMap

- Based on **SwissTable** (hashbrown crate) -- open addressing with SIMD metadata probing
- 1-byte control byte per slot: empty, deleted, or 7 bits of hash (H2)
- Resize at `α > 7/8`; extremely cache-efficient

---

## Slide 17 -- Cuckoo Hashing

### Concept

Two hash functions `h₁, h₂` and two tables `T₁, T₂`. Each key lives in exactly one of its two possible positions.

### Insert algorithm

```
insert(key):
  if T₁[h₁(key)] is empty:
    place key there; return
  if T₂[h₂(key)] is empty:
    place key there; return
  // Evict the occupant of T₁[h₁(key)]
  displaced = T₁[h₁(key)]
  T₁[h₁(key)] = key
  // Re-insert displaced key into its alternate position
  // Repeat until stable or cycle detected → rehash
```

### Properties

| Property | Value |
|----------|-------|
| Lookup | `O(1)` worst case -- check exactly 2 locations |
| Insert | `O(1)` amortised; `O(n)` worst case if rehash triggered |
| Space | Two tables, each ~50% full (`α ≈ 0.5` total) |
| Deletion | `O(1)` -- just clear the slot, no tombstones needed |

> Cuckoo hashing guarantees worst-case `O(1)` lookup -- no long probe chains, no degenerate linked lists. Used in network switches (TCAM replacement), GPU hash tables, and high-frequency trading systems.

---

## Slide 18 -- Robin Hood Hashing

### Concept

Open addressing with linear probing, but elements are **reordered** during insertion to equalise probe distances. "Rob from the rich, give to the poor."

### Insert rule

During an insert, if the new key's probe distance exceeds the current occupant's probe distance, **swap them** and continue inserting the displaced element.

```
insert(key):
  dist = 0
  idx = h(key)
  while table[idx] is occupied:
    if dist > probe_distance(table[idx]):
      swap(key, table[idx])
      dist = probe_distance(table[idx])
    idx = (idx + 1) mod m
    dist += 1
  table[idx] = key
```

### Benefits

- **Variance reduction** -- maximum probe distance is `O(log n)` with high probability vs `O(log n / log log n)` expected
- **Cache-friendly** -- linear probing with bounded displacement
- **Lookup optimisation** -- can terminate search early when current element's probe distance exceeds search distance

> Robin Hood hashing is the basis of Rust's original `HashMap` (pre-SwissTable) and many game engine hash maps. Its bounded probe distance makes worst-case performance predictable.

---

## Slide 19 -- Locality-Sensitive Hashing

### Concept

Standard hash functions are designed so similar inputs produce very different outputs. **LSH does the opposite:** similar items hash to the **same** bucket with high probability.

### Formal definition

A family `H` is `(d₁, d₂, p₁, p₂)`-sensitive if for any points `x, y`:
- If `dist(x, y) ≤ d₁` then `Pr[h(x) = h(y)] ≥ p₁`
- If `dist(x, y) ≥ d₂` then `Pr[h(x) = h(y)] ≤ p₂`

Where `d₁ < d₂` and `p₁ > p₂`.

### Common LSH families

| Distance metric | LSH family | Application |
|----------------|-----------|-------------|
| Jaccard similarity | **MinHash** | Near-duplicate detection, document dedup |
| Cosine similarity | **SimHash** (random hyperplanes) | Semantic similarity, recommendation |
| Euclidean distance | **Random projection** | Image search, ANN |
| Hamming distance | **Bit sampling** | Fingerprint matching |

### Applications

- Approximate nearest-neighbour search in high dimensions
- Near-duplicate web page detection (Google, Bing)
- Audio fingerprinting (Shazam uses a related technique)
- Genome sequence alignment (fast seed generation)

> LSH trades exact correctness for massive speed gains. A brute-force search in `d` dimensions is `O(n)` per query; LSH reduces this to sub-linear time with tuneable accuracy.

---

## Slide 20 -- Applications

### Caches

Hash tables power in-memory caches (Memcached, Redis). Key lookup in `O(1)` enables microsecond response times. Consistent hashing distributes keys across cache nodes.

### Deduplication

Content-addressable storage hashes each data block. If the hash already exists, the block is a duplicate -- store a reference instead. Used in backup systems (Borg, Restic), file systems (ZFS), and build caches.

### Checksums and integrity

Checksums detect corruption in data transfer and storage. `CRC32` for fast error detection; `SHA-256` for cryptographic integrity.

### Bloom filters

A space-efficient probabilistic data structure. Uses `k` hash functions to set bits in a bit array.

- **"Possibly in the set"** or **"definitely not in the set"** -- false positives, no false negatives
- Space: ~10 bits per element for 1% false positive rate
- Used in: databases (avoid disk reads for absent keys), network routers, spell checkers, Chrome safe browsing

### Other applications

- **Hash-based message authentication (HMAC)** -- authenticate messages with a shared secret
- **Distributed hash tables (DHTs)** -- Chord, Kademlia (BitTorrent, IPFS)
- **Load balancing** -- hash the request key to pick a backend server deterministically
- **Data partitioning** -- shard databases by hash of the partition key

---

## Slide 21 -- Summary & Further Reading

### Key takeaways

- Hash functions must be deterministic, efficient, and distribute keys uniformly
- The division method is simple but requires careful choice of table size; the multiplication method is more robust
- Chaining is simple and tolerates high load factors; open addressing is cache-friendly but sensitive to load
- Double hashing eliminates clustering; cuckoo hashing guarantees worst-case O(1) lookup
- Robin Hood hashing bounds probe-distance variance for predictable performance
- Consistent hashing with virtual nodes is the backbone of distributed storage and caching
- Cryptographic hash functions add pre-image and collision resistance -- essential for security, integrity, and authentication
- Perfect and minimal perfect hashing achieve zero collisions for static key sets
- LSH inverts the hash contract -- similar items hash together for approximate nearest-neighbour search
- Real-world hash tables (Python dict, Rust HashMap, Abseil flat_hash_map) combine open addressing with SIMD and careful engineering

### Recommended reading

| Source | Description |
|--------|------------|
| **CLRS** | *Introduction to Algorithms* -- Chapter 11 (Hash Tables) is the canonical reference |
| **Knuth** | *The Art of Computer Programming, Vol. 3* -- Sorting and Searching, Section 6.4 |
| **Karger et al.** | "Consistent Hashing and Random Trees" (1997) -- the original consistent hashing paper |
| **Mitzenmacher & Upfal** | *Probability and Computing* -- rigorous analysis of hashing and Bloom filters |
| **Swiss Tables talk** | CppCon 2017 -- Matt Kulukundis on Abseil's flat_hash_map design |
