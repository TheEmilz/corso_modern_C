# Lezione 2 - Allineamento e Padding

## 🎯 Obiettivi

- Comprendere perché esiste il padding
- Calcolare sizeof di struct manualmente
- Ottimizzare struct per ridurre spreco memoria
- Comprendere cache line e performance

---

## 1. PERCHÉ IL PADDING?

**CPU moderna accede memoria in "word" (4/8 byte):**

```
SENZA ALLINEAMENTO:
struct Bad {
    char a;    // 1 byte
    int b;     // 4 byte (NON allineato!)
    char c;    // 1 byte
};

MEMORIA (ipotetica senza padding):
┌───┬───┬───┬───┬───┬───┐
│ a │  b (parte 1) │ c │
└───┴───┴───┴───┴───┴───┘
    ↑ b attraversa boundary!

PROBLEMA: Leggere 'b' richiede DUE accessi memoria!
1. Leggi word 0-3 → ottieni primi 3 byte di b
2. Leggi word 4-7 → ottieni ultimo byte di b + c
3. Combina i byte → LENTO!

CON ALLINEAMENTO (padding):
┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ a │ X │ X │ X │    b    │ c │ X │ X │ X │
└───┴───┴───┴───┴─────────┴───┴───┴───┴───┘
  ↑padding  ↑aligned!  ↑padding

Un solo accesso per leggere b! ⚡
```

---

## 2. REGOLE DI ALLINEAMENTO

```
REGOLA: Tipo T deve essere allineato a multiplo di sizeof(T)

char:    allineamento = 1 byte (ovunque)
short:   allineamento = 2 byte (indirizzo pari)
int:     allineamento = 4 byte (multiplo di 4)
long:    allineamento = 8 byte (multiplo di 8) [64-bit]
float:   allineamento = 4 byte
double:  allineamento = 8 byte
pointer: allineamento = 8 byte [64-bit]

STRUCT: allineamento = max(allineamenti dei membri)
```

---

## 3. CALCOLARE SIZEOF

### Esempio 1:
```c
struct Example {
    char a;      // 1 byte
    int b;       // 4 byte
    char c;      // 1 byte
    double d;    // 8 byte
};

CALCOLO:
Offset 0: char a (1 byte)
Offset 1-3: PADDING (int richiede allineamento a 4)
Offset 4: int b (4 byte)
Offset 8: char c (1 byte)
Offset 9-15: PADDING (double richiede allineamento a 8)
Offset 16: double d (8 byte)
Offset 24: PADDING finale (struct stesso allineato a 8)

sizeof(Example) = 24 byte
SPRECO: 7 byte di padding!

MEMORIA:
┌───┬───┬───┬───┬───────┬───┬───┬───┬───┬───┬───┬───┬─────────┐
│ a │ X │ X │ X │   b   │ c │ X │ X │ X │ X │ X │ X │    d    │
└───┴───┴───┴───┴───────┴───┴───┴───┴───┴───┴───┴───┴─────────┘
```

### Esempio 2: Ottimizzato
```c
struct Optimized {
    double d;    // 8 byte (più grande prima!)
    int b;       // 4 byte
    char a;      // 1 byte
    char c;      // 1 byte
    // 2 byte padding finale
};

MEMORIA:
┌─────────┬───────┬───┬───┬───┬───┐
│    d    │   b   │ a │ c │ X │ X │
└─────────┴───────┴───┴───┴───┴───┘

sizeof(Optimized) = 16 byte
SPRECO: solo 2 byte! (vs 7 prima)

REGOLA ORO: Ordina membri dal PIÙ GRANDE al PIÙ PICCOLO!
```

---

## 4. CACHE LINE

```
CACHE LINE tipica: 64 byte

struct Data {
    int value;
    char padding[60];  // Occupa esattamente 1 cache line
};

struct Data array[100];  // 100 elementi

ACCESSO array[0].value:
1. CPU carica INTERA cache line (64 byte)
2. Ottiene anche padding (sprecato ma inevitable)

FALSE SHARING (multi-threading):
Thread 1 modifica array[0].value
Thread 2 modifica array[1].value

Anche se modificano dati DIVERSI, se sono sulla
STESSA cache line → cache line invalidata!
PERFORMANCE DEGRADATION!

SOLUZIONE: Padding per separare su cache line diverse.
```

---

## 5. TOOLS

```c
// Vedere allineamento in C:
#include <stddef.h>
#include <stdio.h>

struct Example {
    char a;
    int b;
    char c;
    double d;
};

printf("sizeof(Example) = %zu\n", sizeof(struct Example));
printf("offset a = %zu\n", offsetof(struct Example, a));
printf("offset b = %zu\n", offsetof(struct Example, b));
printf("offset c = %zu\n", offsetof(struct Example, c));
printf("offset d = %zu\n", offsetof(struct Example, d));

// Output:
// sizeof(Example) = 24
// offset a = 0
// offset b = 4  (padding 1-3)
// offset c = 8
// offset d = 16 (padding 9-15)
```

---

## 6. PRAGMA PACK

```c
// Disabilita padding (USE WITH CAUTION!)
#pragma pack(push, 1)
struct Packed {
    char a;    // offset 0
    int b;     // offset 1 (NO padding!)
    char c;    // offset 5
    double d;  // offset 6 (NO padding!)
};
#pragma pack(pop)

sizeof(Packed) = 14 byte (vs 24 normale)

⚠️ TRADE-OFF:
✓ Risparmio memoria
✗ Accessi NON allineati → PIÙ LENTI!
✗ Alcuni CPU crashano su accessi non allineati!

USE CASE: Serializzazione rete/file
```

---

[← Stack vs Heap](01_stack_vs_heap.md) | [Torna al Modulo](README.md) | [Prossima: Endianness →](03_endianness.md)
