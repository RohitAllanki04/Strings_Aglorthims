# String Pattern Matching Algorithms

Notes on two classic string-matching techniques — **Naive (Brute Force)** and **KMP (Knuth-Morris-Pratt)** — implemented in Java.

## 📌 Table of Contents

- [1. Naive Pattern Matching](#1-naive-pattern-matching)
- [2. KMP Algorithm](#2-kmp-algorithm)
  - [2.1 LPS / Pi Table](#21-lps--pi-table)
  - [2.2 Search Phase](#22-search-phase)
- [3. Complexity Comparison](#3-complexity-comparison)
- [4. Full Java Code](#4-full-java-code)

---

## 1. Naive Pattern Matching

The brute-force approach checks every possible starting position in the text and compares character by character with the pattern.

### Idea
For each index `i` in `str`, try to match the entire `pattern` starting at `i`.

```java
static boolean naiveSearch(String str, String pattern) {
    int n = str.length();
    int m = pattern.length();

    for (int i = 0; i <= n - m; i++) {
        int j;
        for (j = 0; j < m; j++) {
            if (str.charAt(i + j) != pattern.charAt(j)) {
                break;
            }
        }
        if (j == m) return true; // full pattern matched
    }
    return false;
}
```

### Complexity
| Metric | Value |
|--------|-------|
| Time   | O(n·m) worst case |
| Space  | O(1) |

**Worst case example:** `str = "aaaaaaaab"`, `pattern = "aaab"` — every position requires a near-full comparison before failing.

---

## 2. KMP Algorithm

KMP avoids re-checking characters that are already known to match by precomputing a **failure function** (LPS / Pi table) from the pattern itself.

### 2.1 LPS / Pi Table

**LPS** = Longest Proper Prefix which is also a Suffix.

`lps[i]` stores the length of the longest proper prefix of `pattern[0..i]` that is also a suffix of `pattern[0..i]`.

```java
static int[] buildLPS(String pattern) {
    int m = pattern.length();
    int[] lps = new int[m];
    int len = 0; // length of previous longest prefix-suffix
    int i = 1;
    lps[0] = 0; // base case: single char has no proper prefix

    while (i < m) {
        if (pattern.charAt(i) == pattern.charAt(len)) {
            len++;
            lps[i] = len;
            i++;
        } else {
            if (len != 0) {
                len = lps[len - 1]; // fall back using table, i doesn't move
            } else {
                lps[i] = 0;
                i++;
            }
        }
    }
    return lps;
}
```

**Example:**

| index  | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|--------|---|---|---|---|---|---|---|
| char   | a | a | b | a | a | a | b |
| lps    | 0 | 1 | 0 | 1 | 2 | 2 | 3 |

**Key insight:** `i` only ever moves forward — it never resets. `len` can shrink via `lps[len-1]`, but the total work across the whole pass is still bounded by `m`, giving **O(m)** construction.

### 2.2 Search Phase

Using the LPS table, scan the text once. On a mismatch, instead of restarting from scratch, fall back using the table — the text pointer `i`(or `y`) never moves backward.

```java
static boolean kmpSearch(String text, String pattern) {
    int n = text.length(), m = pattern.length();
    if (m == 0) return false;
    int[] lps = buildLPS(pattern);

    int i = 0, j = 0; // i -> text, j -> pattern
    while (i < n) {
        if (text.charAt(i) == pattern.charAt(j)) {
            i++;
            j++;
            if (j == m) return true; // full match found
        } else {
            if (j != 0) {
                j = lps[j - 1]; // use table, don't reset i
            } else {
                i++;
            }
        }
    }
    return false;
}
```

---

## 3. Complexity Comparison

| Algorithm | Time Complexity | Space Complexity | Notes |
|-----------|-----------------|-------------------|-------|
| Naive     | O(n·m)           | O(1)              | Simple, but re-scans overlapping characters |
| KMP       | O(n + m)         | O(m)              | LPS table avoids redundant comparisons |

**Why KMP is O(n+m), not O(n·m):**
- Building `lps` takes O(m) — `i` moves forward only.
- Searching takes O(n) — the text pointer `i` never moves backward, even on mismatches; only the pattern pointer `j` falls back via the table.

---

## 4. Full Java Code

Combined driver with both algorithms (naive commented out, KMP active) — useful for quick testing via `Scanner` input.

```java
import java.util.*;

class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String str = sc.nextLine();
        String pattern = sc.nextLine();

        System.out.println(kmpSearch(str, pattern)
            ? "Pattern Matched"
            : "Pattern is not Matched in String");
    }

    // ---------- Naive Pattern Matching : O(n*m) ----------
    static boolean naiveSearch(String str, String pattern) {
        int n = str.length();
        int m = pattern.length();
        if (m == 0 || m > n) return false;

        for (int i = 0; i <= n - m; i++) {
            int j;
            for (j = 0; j < m; j++) {
                if (str.charAt(i + j) != pattern.charAt(j)) {
                    break;
                }
            }
            if (j == m) return true;
        }
        return false;
    }

    // ---------- KMP : LPS Table Construction : O(m) ----------
    static int[] buildLPS(String pattern) {
        int m = pattern.length();
        int[] lps = new int[m];
        int len = 0;
        int i = 1;
        lps[0] = 0;

        while (i < m) {
            if (pattern.charAt(i) == pattern.charAt(len)) {
                len++;
                lps[i] = len;
                i++;
            } else {
                if (len != 0) {
                    len = lps[len - 1];
                } else {
                    lps[i] = 0;
                    i++;
                }
            }
        }
        return lps;
    }

    // ---------- KMP : Search Phase : O(n) ----------
    static boolean kmpSearch(String text, String pattern) {
        int n = text.length(), m = pattern.length();
        if (m == 0) return false;
        int[] lps = buildLPS(pattern);

        int i = 0, j = 0;
        while (i < n) {
            if (text.charAt(i) == pattern.charAt(j)) {
                i++;
                j++;
                if (j == m) return true;
            } else {
                if (j != 0) {
                    j = lps[j - 1];
                } else {
                    i++;
                }
            }
        }
        return false;
    }
}
```

---

### 🎯 Interview Talking Points

- Naive is easy to explain but degrades to O(n·m) on repetitive patterns (e.g., `"aaaa...a"`).
- KMP's core trick: **never discard information** you've already matched — the LPS table encodes it so the pattern pointer can "jump back" intelligently instead of the text pointer restarting.
- Follow-up algorithms interviewers may ask about next: **Z-algorithm**, **Rabin-Karp** (rolling hash), **Boyer-Moore**.
