# Bloom Filter

## What is it?
A **Bloom filter** is a **probabilistic data structure** introduced by **Burton Howard Bloom in 1970**.  
It is used to test whether an element **is a member** of a set, in a **fast and memory‑efficient** way.

- It never gives false negatives → if it says an element is not present, it is guaranteed to be absent.  
- It can give false positives → it may say an element exists even if it was never added.  
- It is widely used in databases, networks, and distributed systems.

---

## How it works
1. Create a bit array of length `m`, initially all zeros.  
2. Choose `k` independent hash functions.  
3. Insertion (`add`):  
   - For an element `x`, compute `h1(x), h2(x), ..., hk(x)`.  
   - Set the bits at those positions to `1`.  
4. Lookup (`check`):  
   - For an element `y`, compute `h1(y), ..., hk(y)`.  
   - If all bits are `1` → `y` might be in the set.  
   - If any bit is `0` → `y` is definitely not in the set.  

---

## Applications
- Databases and Big Data → Used in Cassandra, HBase, Redis to avoid unnecessary disk lookups.  
- Networks and Web → Spam filters, DNS caching, visited URL checks.  
- Blockchain → To verify if a transaction belongs to a block.  
- Distributed systems → Reduce network traffic by eliminating useless requests.  
- Search engines → Faster indexing, reduced redundant queries.  

---

## Example
Imagine a Bloom filter with:  
- `m = 10` (total bits)  
- `k = 3` (hash functions)  

### Steps
1. Insert `"hello"` → hash → positions `[2, 5, 8]`  
   ```
   [0][0][1][0][0][1][0][0][1][0]
   ```
2. Insert `"chat"` → hash → positions `[1, 5, 9]`  
   ```
   [0][1][1][0][0][1][0][0][1][1]
   ```
3. Check `"cat"` → hash → positions `[2, 4, 9]`  
   - Bit 2 = 1  
   - Bit 4 = 0  
   - Bit 9 = 1  
   Result: definitely not present  

---

## Pros and Cons
Advantages  
- Very low memory consumption.  
- Extremely fast queries (O(k), with small k like 3–7).  
- Perfect for very large datasets.  

Limitations  
- Can produce false positives.  
- Does not support deletions (unless using variants).  
- Requires careful tuning of `m` and `k` for optimal balance.  

---

## Evolutions
- Counting Bloom Filter → Uses counters instead of bits, allowing deletions.  
- Scalable Bloom Filter → Grows dynamically as more data is added.  
- Compressed Bloom Filter → Optimized for transmission over networks.  
- Cuckoo Filter → A modern alternative with fewer false positives and supports deletions.  

---

## ASCII Visualization

### Initial state
```
[0][0][0][0][0][0][0][0][0][0]
```

### After inserting "hello"
```
[0][0][1][0][0][1][0][0][1][0]
```

### After inserting "chat"
```
[0][1][1][0][0][1][0][0][1][1]
```

### Checking "cat"
- Checking positions `[2, 4, 9]`  
- Result → a zero is found → definitely not present  

---

## Python Example

```python
import mmh3  # MurmurHash
from bitarray import bitarray

class BloomFilter:
    def __init__(self, size=10, hash_count=3):
        self.size = size
        self.hash_count = hash_count
        self.bit_array = bitarray(size)
        self.bit_array.setall(0)

    def add(self, item):
        for i in range(self.hash_count):
            digest = mmh3.hash(item, i) % self.size
            self.bit_array[digest] = 1

    def check(self, item):
        for i in range(self.hash_count):
            digest = mmh3.hash(item, i) % self.size
            if self.bit_array[digest] == 0:
                return False
        return True

# Usage
bf = BloomFilter(10, 3)
bf.add("hello")
bf.add("chat")

print(bf.check("hello"))   # True (probably present)
print(bf.check("cat"))     # False (definitely absent)
```
