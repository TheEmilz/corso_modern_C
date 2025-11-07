# Guida Completa all'Undefined Behavior in C

## 📋 Indice

1. [Buffer Overflow](#1-buffer-overflow)
2. [Use-After-Free](#2-use-after-free)
3. [NULL Pointer Dereference](#3-null-pointer-dereference)
4. [Signed Integer Overflow](#4-signed-integer-overflow)
5. [Uninitialized Variables](#5-uninitialized-variables)
6. [Sequence Point Violations](#6-sequence-point-violations)
7. [Division by Zero](#7-division-by-zero)
8. [Invalid Pointer Arithmetic](#8-invalid-pointer-arithmetic)
9. [Strict Aliasing Violations](#9-strict-aliasing-violations)
10. [Data Races](#10-data-races)

---

## 1. BUFFER OVERFLOW

### Descrizione
Scrivere/leggere oltre i limiti di un array o buffer.

### Esempio
```c
int arr[5];
arr[10] = 42;  // UB! Indice fuori bounds

char buf[10];
strcpy(buf, "Questa stringa è troppo lunga!");  // UB! Overflow
```

### Cosa Può Succedere
```
MEMORIA:
┌─────────────┬──────────────────┐
│ arr[0..4]   │ altra variabile  │
└─────────────┴──────────────────┘
             ↑
      arr[10] scrive QUI! ← Corrompe altra variabile

CONSEGUENZE:
✗ Segmentation fault (se scrivi in memoria protetta)
✗ Corruzione dati
✗ Exploit di sicurezza (buffer overflow attack!)
✗ Comportamento imprevedibile
```

### Come Detectare
```bash
# AddressSanitizer
gcc -fsanitize=address -g program.c
./a.out

# Output:
# ==12345==ERROR: AddressSanitizer: heap-buffer-overflow
# WRITE of size 4 at 0x... thread T0
```

### Come Prevenire
```c
// ❌ PERICOLOSO
void copy(char *dest, const char *src) {
    strcpy(dest, src);  // NO controllo dimensione!
}

// ✅ SICURO
void copy_safe(char *dest, const char *src, size_t dest_size) {
    if (dest == NULL || src == NULL || dest_size == 0) return;
    
    strncpy(dest, src, dest_size - 1);
    dest[dest_size - 1] = '\0';  // Garantisci null-termination
}

// ✅ ANCORA MEGLIO: Usa snprintf
snprintf(dest, dest_size, "%s", src);

// ✅ CONTROLLI BOUNDS
#define ARRAY_SIZE 10
int arr[ARRAY_SIZE];
int index = get_index();

if (index >= 0 && index < ARRAY_SIZE) {
    arr[index] = value;  // Sicuro
} else {
    fprintf(stderr, "Indice %d fuori range!\n", index);
}
```

---

## 2. USE-AFTER-FREE

### Descrizione
Usare un puntatore dopo aver liberato la memoria puntata.

### Esempio
```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);
printf("%d\n", *p);  // UB! p è dangling pointer
```

### Cosa Può Succedere
```
MEMORIA DOPO free(p):
┌──────────────────────┐
│ p → [??? garbage]    │  ← Memoria restituita all'allocatore
└──────────────────────┘

p CONTIENE ANCORA L'INDIRIZZO, ma:
- Memoria può essere riassegnata
- Contenuto è indefinito
- Lettura = UB
- Scrittura = corruzione heap!

SCENARIO PEGGIORE:
1. free(p)
2. malloc() riusa quella memoria per altro
3. *p = 10  ← CORROMPI dati di qualcun altro!
```

### Come Detectare
```bash
# AddressSanitizer
gcc -fsanitize=address -g program.c

# Output:
# ==12345==ERROR: AddressSanitizer: heap-use-after-free
# READ of size 4 at 0x... thread T0
```

### Come Prevenire
```c
// ✅ PATTERN: Nullifica dopo free
int *p = malloc(sizeof(int));
*p = 42;
free(p);
p = NULL;  // Ora dereferenziare p → crash immediato (meglio di corruzione!)

// ✅ OWNERSHIP CHIARO
/* Regola: Chi alloca, dealloca
 * Se funzione ritorna puntatore → documenta ownership!
 */

/**
 * @ownership CALLER - Il chiamante DEVE free() il risultato
 */
char *create_string(const char *src) {
    char *s = malloc(strlen(src) + 1);
    if (s != NULL) {
        strcpy(s, src);
    }
    return s;
}

// Uso:
char *s = create_string("hello");
// ... uso s ...
free(s);  // Caller libera
s = NULL;
```

---

## 3. NULL POINTER DEREFERENCE

### Descrizione
Dereferenziare un puntatore NULL.

### Esempio
```c
int *p = NULL;
*p = 42;  // UB! Segmentation fault
```

### Cosa Può Succedere
```
NULL = 0x0 (indirizzo invalido)

TENTATIVO ACCESSO @ 0x0:
- Sistema Operativo: "ACCESSO NEGATO!"
- Risultato: Segmentation Fault (SIGSEGV)

CRASH IMMEDIATO (questo è un "bene":  fail fast!)
```

### Come Detectare
```bash
# AddressSanitizer
gcc -fsanitize=address -g program.c

# Crash immediato con stacktrace
```

### Come Prevenire
```c
// ✅ CONTROLLA SEMPRE malloc
int *p = malloc(sizeof(int));
if (p == NULL) {
    fprintf(stderr, "Out of memory!\n");
    return -1;
}
*p = 42;  // Sicuro

// ✅ CONTROLLA parametri funzioni
void process(int *data) {
    if (data == NULL) {
        fprintf(stderr, "Error: NULL pointer\n");
        return;
    }
    *data = 10;  // Sicuro
}

// ✅ ASSERZIONI (debug mode)
#include <assert.h>

void critical_function(int *p) {
    assert(p != NULL && "Pointer must not be NULL");
    *p = 100;
}
```

---

## 4. SIGNED INTEGER OVERFLOW

### Descrizione
Overflow di interi con segno (signed).

### Esempio
```c
int x = INT_MAX;  // 2147483647
x++;  // UB! x diventa ???
```

### Cosa Può Succedere
```
INT_MAX = 0x7FFFFFFF (2147483647)
INT_MAX + 1 = ???

MATEMATICAMENTE: 2147483648
MA int signed è a 32-bit con complemento a 2:
0x7FFFFFFF + 1 = 0x80000000 = -2147483648 (wrapped!)

LO STANDARD C DICE: UNDEFINED BEHAVIOR!
Il compilatore può:
- Assumere che overflow non avvenga mai
- Ottimizzare basandosi su questo assunto
- Comportamento imprevedibile

ESEMPIO REALE (ottimizzazione):
int x = INT_MAX;
if (x + 1 > x) {  // Compilatore assume SEMPRE true!
    // Questo blocco può essere RIMOSSO!
}
```

### Come Detectare
```bash
# UndefinedBehaviorSanitizer
gcc -fsanitize=undefined -g program.c

# Output:
# program.c:5:7: runtime error: signed integer overflow:
# 2147483647 + 1 cannot be represented in type 'int'
```

### Come Prevenire
```c
// ✅ CONTROLLA PRIMA
#include <limits.h>

int safe_add(int a, int b, int *result) {
    if (a > 0 && b > INT_MAX - a) {
        return -1;  // Overflow
    }
    if (a < 0 && b < INT_MIN - a) {
        return -1;  // Underflow
    }
    *result = a + b;
    return 0;  // Success
}

// ✅ USA unsigned (wraparound definito)
unsigned int x = UINT_MAX;
x++;  // OK! Wraparound a 0 (comportamento definito)

// ✅ CONTROLLA LIMITI
int increment_safe(int x) {
    if (x == INT_MAX) {
        fprintf(stderr, "Cannot increment INT_MAX\n");
        return INT_MAX;
    }
    return x + 1;
}
```

---

## 5. UNINITIALIZED VARIABLES

### Descrizione
Leggere variabili non inizializzate.

### Esempio
```c
int x;  // Non inizializzata
printf("%d\n", x);  // UB! x contiene garbage
```

### Cosa Può Succedere
```
MEMORIA STACK:
┌──────────────┐
│ x: [??][??]  │  ← Contenuto casuale (leftover da chiamate precedenti)
└──────────────┘

POSSIBILI VALORI:
- 0 (fortuna!)
- 42
- -1234567
- Qualsiasi valore!

PERICOLO: Test possono passare localmente (x=0 per caso)
          ma fallire in produzione (x=garbage)
```

### Come Detectare
```bash
# Valgrind
valgrind --track-origins=yes ./program

# Output:
# Conditional jump or move depends on uninitialised value(s)
# at 0x...: main (program.c:10)

# MemorySanitizer (Clang)
clang -fsanitize=memory -g program.c
```

### Come Prevenire
```c
// ✅ INIZIALIZZA SEMPRE
int x = 0;  // Esplicito
printf("%d\n", x);  // Sicuro

// ✅ ZERO INIT (C99+)
int arr[100] = {0};  // Tutti a 0

// ✅ MALLOC + MEMSET o CALLOC
int *p = malloc(10 * sizeof(int));
if (p != NULL) {
    memset(p, 0, 10 * sizeof(int));  // Zero init
}
// Oppure:
int *p2 = calloc(10, sizeof(int));  // Già a 0

// ✅ WARNING COMPILATORE
gcc -Wall -Wextra -Wuninitialized program.c
```

---

## 6. SEQUENCE POINT VIOLATIONS

### Descrizione
Modificare stessa variabile più volte tra due sequence point.

### Esempio
```c
int x = 5;
x = x++ + ++x;  // UB! x modificata 3 volte senza sequence point
```

### Cosa Può Succedere
```
SEQUENCE POINTS: Punti dove side effects sono garantiti completi
- ; (fine statement)
- && || ? : (short-circuit)
- , (comma operator, non in funzioni!)
- Fine espressione di controllo (if, while, ...)

ESEMPIO:
x = x++ + ++x;

POSSIBILI VALUTAZIONI:
1. x++ → usa 5, poi x=6
   ++x → x=7, usa 7
   5 + 7 = 12, x = 12? 7? 6?

2. ++x → x=6, usa 6
   x++ → usa 6, poi x=7
   6 + 6 = 12, x = 12? 7?

3. Altre combinazioni!

LO STANDARD: UNDEFINED! Compilatore sceglie.
```

### Come Detectare
```bash
# Compiler warnings
gcc -Wall -Wextra -Wsequence-point program.c

# Warning:
# program.c:5:7: warning: operation on 'x' may be undefined
```

### Come Prevenire
```c
// ❌ EVITA
x = x++ + ++x;
arr[i++] = i;
f(i++, i++);

// ✅ SEPARA IN STATEMENT DISTINTI
int temp1 = x++;
int temp2 = ++x;
x = temp1 + temp2;

// ✅ UNA MODIFICA PER STATEMENT
i++;
arr[i] = value;

// ✅ CHIARO E LEGGIBILE
x = x + 1;  // Invece di x++ in espressioni complesse
```

---

## 7. DIVISION BY ZERO

### Descrizione
Divisione per zero.

### Esempio
```c
int x = 10 / 0;  // UB!
int y = 10 % 0;  // UB! (modulo zero)
```

### Cosa Può Succedere
```
CPU TRAP: Divisione per 0 causa eccezione hardware
- SIGFPE (Floating Point Exception)  ← Nome fuorviante!
- Crash immediato

FLOATING POINT:
double x = 10.0 / 0.0;  // NON UB! Risultato: +Infinity
```

### Come Detectare
```bash
# UndefinedBehaviorSanitizer
gcc -fsanitize=undefined -g program.c

# Output:
# program.c:5:15: runtime error: division by zero
```

### Come Prevenire
```c
// ✅ CONTROLLA DENOMINATORE
int safe_divide(int a, int b, int *result) {
    if (b == 0) {
        fprintf(stderr, "Error: division by zero\n");
        return -1;
    }
    *result = a / b;
    return 0;
}

// ✅ ASSERZIONI
int divide(int a, int b) {
    assert(b != 0 && "Divisor must not be zero");
    return a / b;
}
```

---

## 8. INVALID POINTER ARITHMETIC

### Descrizione
Aritmetica puntatori fuori dai limiti dell'array.

### Esempio
```c
int arr[5];
int *p = arr + 10;  // UB! Oltre array
int *q = arr - 1;   // UB! Prima dell'array
```

### Cosa Può Succedere
```
ARRAY: int arr[5]
┌───┬───┬───┬───┬───┐
│ 0 │ 1 │ 2 │ 3 │ 4 │
└───┴───┴───┴───┴───┘
 ↑                   ↑
arr               arr+5 ← ONE-PAST-END (legale!)

arr + 10 → OLTRE array! UB!
arr - 1 → PRIMA array! UB!

ECCEZIONE: arr+5 (one-past-end) è LEGALE
           MA non puoi dereferenziare!

int *end = arr + 5;  // OK
*end = 42;           // UB! Dereferenzia one-past-end
```

### Come Detectare
```bash
# AddressSanitizer
gcc -fsanitize=address -g program.c
```

### Come Prevenire
```c
// ✅ CONTROLLA BOUNDS
int arr[5];
int *p = arr;

while (p < arr + 5) {  // OK: confronto con one-past-end
    *p = 0;
    p++;
}

// ✅ USA INDICI (più chiaro)
for (int i = 0; i < 5; i++) {
    arr[i] = 0;
}
```

---

## 9. STRICT ALIASING VIOLATIONS

### Descrizione
Accesso memoria attraverso puntatore di tipo incompatibile.

### Esempio
```c
int x = 42;
float *fp = (float*)&x;
*fp = 3.14;  // UB! Violazione strict aliasing
```

### Cosa Può Succedere
```
STRICT ALIASING RULE:
Due puntatori di tipo diverso (eccetto char*)
NON possono puntare alla stessa memoria.

Il compilatore ASSUME questa regola e ottimizza:

int x = 1;
float *fp = (float*)&x;
*fp = 0.5;
printf("%d\n", x);  // Compilatore può ancora usare valore 1!
                    // (assume x non modificato da *fp)

RISULTATO: Comportamento imprevedibile con ottimizzazioni
```

### Come Detectare
```bash
# UndefinedBehaviorSanitizer
gcc -fsanitize=undefined -g program.c
```

### Come Prevenire
```c
// ❌ PERICOLOSO
int x = 42;
float *fp = (float*)&x;
*fp = 3.14;

// ✅ USA UNION (type-punning safe)
union IntFloat {
    int i;
    float f;
};

union IntFloat u;
u.i = 42;
printf("As float: %f\n", u.f);  // OK!

// ✅ MEMCPY (sempre safe)
int x = 42;
float f;
memcpy(&f, &x, sizeof(float));  // OK!

// ✅ CHAR* SEMPRE SAFE
int x = 42;
unsigned char *bytes = (unsigned char*)&x;
for (size_t i = 0; i < sizeof(int); i++) {
    printf("%02X ", bytes[i]);  // OK!
}
```

---

## 10. DATA RACES

### Descrizione
Accesso concorrente a stessa variabile senza sincronizzazione.

### Esempio
```c
int counter = 0;

// Thread 1:
counter++;  // Leggi, incrementa, scrivi

// Thread 2:
counter++;  // Leggi, incrementa, scrivi
// UB! Data race!
```

### Cosa Può Succedere
```
SCENARIO:
counter = 0

Thread 1: Leggi counter (0)
Thread 2: Leggi counter (0)
Thread 1: Incrementa (1), Scrivi counter = 1
Thread 2: Incrementa (1), Scrivi counter = 1

RISULTATO: counter = 1 (invece di 2!)

ANCHE PEGGIO: Scritture concorrenti possono corrompere!
```

### Come Detectare
```bash
# ThreadSanitizer
gcc -fsanitize=thread -g -pthread program.c

# Output:
# WARNING: ThreadSanitizer: data race
```

### Come Prevenire
```c
// ✅ MUTEX
#include <pthread.h>

int counter = 0;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

void increment() {
    pthread_mutex_lock(&lock);
    counter++;
    pthread_mutex_unlock(&lock);
}

// ✅ ATOMIC (C11)
#include <stdatomic.h>

atomic_int counter = 0;

void increment() {
    atomic_fetch_add(&counter, 1);  // Atomico, no race!
}

// ✅ THREAD-LOCAL (se non serve condividere)
_Thread_local int counter = 0;  // Ogni thread ha sua copia
```

---

## 📝 CHECKLIST ANTI-UB

Prima di commit:

- [ ] Compilato con `-Wall -Wextra -Werror`
- [ ] Testato con `-fsanitize=address`
- [ ] Testato con `-fsanitize=undefined`
- [ ] Tutti i puntatori controllati per NULL
- [ ] Tutti gli indici controllati per bounds
- [ ] Tutte le variabili inizializzate
- [ ] Nessuna espressione complessa con ++/--
- [ ] Malloc sempre con controllo NULL
- [ ] Ogni malloc ha corrispondente free
- [ ] Puntatori nullificati dopo free
- [ ] Divisioni con controllo zero
- [ ] Multi-threading con sincronizzazione

---

[Torna al Modulo](README.md) | [Modulo 04 Avanzato →](../04_Avanzato/README.md)
