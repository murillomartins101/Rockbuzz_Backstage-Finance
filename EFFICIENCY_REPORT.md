# Efficiency Analysis Report - Rockbuzz Backstage Finance

## Overview

This report documents several areas in the codebase where efficiency improvements could be made. The analysis covers performance bottlenecks, code inefficiencies, and a critical bug that affects functionality.

---

## Issue 1: Critical Bug - Incorrect DataFrame Filtering (Line 688)

**Severity:** Critical (Bug)  
**Location:** `rockbuzz_backstage_finance.py:688`

**Current Code:**
```python
sem_data = sem_data[sem_data["categoria"]] == categoria_sel
```

**Problem:** This line has a syntax error that causes incorrect behavior. The bracket placement is wrong - it's comparing a DataFrame to a string instead of filtering rows where the "categoria" column equals `categoria_sel`.

**Fix:**
```python
sem_data = sem_data[sem_data["categoria"] == categoria_sel]
```

**Impact:** This bug prevents proper filtering of records without dates by category, potentially showing incorrect data in the "Lancamentos" view.

---

## Issue 2: Inefficient DataFrame Iteration with iterrows() (Lines 729-733)

**Severity:** Medium (Performance)  
**Location:** `rockbuzz_backstage_finance.py:729-733`

**Current Code:**
```python
lancamentos_lista = []
for idx, row in view.iterrows():
    desc = (row['descricao'][:30] + "...") if isinstance(row['descricao'], str) and len(row['descricao']) > 30 else str(row['descricao'])
    data_txt = row["data"].strftime('%d/%m/%Y') if pd.notna(row["data"]) else "-"
    texto = f"{data_txt} | {row['tipo']} | {row['categoria']} | {brl(abs(row['valor']))} | {desc}"
    lancamentos_lista.append((idx, texto))
```

**Problem:** `iterrows()` is notoriously slow for large DataFrames because it creates a Series object for each row. For datasets with thousands of records, this can cause significant UI lag.

**Recommended Fix:** Use vectorized pandas operations:
```python
desc_series = view['descricao'].astype(str).str[:30] + view['descricao'].astype(str).str.len().gt(30).map({True: '...', False: ''})
data_txt_series = view["data"].dt.strftime('%d/%m/%Y').fillna("-")
valor_fmt = view['valor'].abs().map(brl)
texto_series = data_txt_series + " | " + view['tipo'] + " | " + view['categoria'] + " | " + valor_fmt + " | " + desc_series
lancamentos_lista = list(zip(view.index, texto_series))
```

**Impact:** Could improve performance by 10-100x for large datasets.

---

## Issue 3: Inefficient groupby with apply() (Lines 457-462)

**Severity:** Medium (Performance)  
**Location:** `rockbuzz_backstage_finance.py:457-462`

**Current Code:**
```python
monthly = dfp.groupby("ano_mes", dropna=False).apply(
    lambda x: pd.Series({
        "Receitas": x.loc[x["valor"] > 0, "valor"].sum(),
        "Despesas": -x.loc[x["valor"] < 0, "valor"].sum()
    })
).reset_index()
```

**Problem:** Using `apply()` with a lambda that creates a Series is slower than using built-in aggregation methods.

**Recommended Fix:**
```python
receitas = dfp[dfp["valor"] > 0].groupby("ano_mes", dropna=False)["valor"].sum().rename("Receitas")
despesas = (-dfp[dfp["valor"] < 0].groupby("ano_mes", dropna=False)["valor"].sum()).rename("Despesas")
monthly = pd.concat([receitas, despesas], axis=1).fillna(0).reset_index()
```

**Impact:** Moderate performance improvement, especially with larger datasets.

---

## Issue 4: Inefficient Import Iteration (Lines 1042-1053)

**Severity:** Medium (Performance)  
**Location:** `rockbuzz_backstage_finance.py:1042-1053`

**Current Code:**
```python
rows = []
for _, r in parsed.iterrows():
    rows.append([
        (pd.to_datetime(r.get("data")).strftime("%Y-%m-%d") if pd.notna(r.get("data")) else ""),
        r.get("tipo", ""), 
        ...
    ])
```

**Problem:** Same `iterrows()` inefficiency as Issue 2, but in the import functionality.

**Recommended Fix:** Use vectorized operations or `to_records()` to convert the DataFrame to a list of lists more efficiently.

**Impact:** Significant improvement for large Excel imports.

---

## Issue 5: Redundant Function Calls (Lines 100-113)

**Severity:** Low (Performance)  
**Location:** `rockbuzz_backstage_finance.py:100-113`

**Current Code:**
```python
def calcular_ticket_medio(df: pd.DataFrame) -> float:
    ...
    base = df.loc[_only_shows_mask(df)].copy()
    ...
    qtd = count_shows(df)  # This calls _only_shows_mask again internally
```

**Problem:** `_only_shows_mask(df)` is called twice - once directly and once inside `count_shows()`. This duplicates computation.

**Recommended Fix:** Refactor to pass the pre-computed mask or base DataFrame to avoid redundant filtering.

---

## Issue 6: Inefficient String Normalization (Lines 220-227)

**Severity:** Low (Performance)  
**Location:** `rockbuzz_backstage_finance.py:220-227`

**Current Code:**
```python
def norm_col(s: str) -> str:
    s = s.strip()
    t = (s.lower()
           .replace("a","a").replace("a","a").replace("a","a").replace("a","a")
           .replace("e","e").replace("e","e").replace("i","i")
           .replace("o","o").replace("o","o").replace("o","o")
           .replace("u","u").replace("c","c"))
    return " ".join(t.split())
```

**Problem:** Multiple chained `replace()` calls are less efficient than using `str.translate()` with a translation table or the `unidecode` library.

**Recommended Fix:**
```python
import unicodedata

def norm_col(s: str) -> str:
    s = s.strip().lower()
    s = unicodedata.normalize('NFD', s)
    s = ''.join(c for c in s if unicodedata.category(c) != 'Mn')
    return ' '.join(s.split())
```

---

## Summary

| Issue | Severity | Type | Estimated Impact |
|-------|----------|------|------------------|
| 1. DataFrame filtering bug | Critical | Bug | Functionality broken |
| 2. iterrows() in lancamentos | Medium | Performance | 10-100x slower |
| 3. groupby apply() | Medium | Performance | 2-5x slower |
| 4. iterrows() in import | Medium | Performance | 10-100x slower |
| 5. Redundant function calls | Low | Performance | Minor overhead |
| 6. String normalization | Low | Performance | Minor overhead |

## Recommendation

**Priority 1:** Fix Issue 1 (critical bug) immediately as it affects functionality.

**Priority 2:** Address Issues 2 and 4 (iterrows) as they have the highest performance impact.

**Priority 3:** Optimize Issues 3, 5, and 6 for incremental improvements.

---

*Report generated: 2025-12-31*
