# Ownership e Lifetime in C

## 🎯 Obiettivi

Comprendere i concetti di:
- **Ownership**: Chi è responsabile di liberare la memoria
- **Lifetime**: Quando una variabile/memoria è valida
- **Borrowing**: Passaggio temporaneo di accesso senza trasferire ownership
- Come prevenire dangling pointers e memory leaks

---

## 1. IL CONCETTO DI OWNERSHIP

### 1.1 Cos'è l'Ownership?

**Ownership** = Responsabilità di gestire (deallocare) una risorsa.

```
REGOLA D'ORO: CHI ALLOCA, DEALLOCA

Ogni blocco di memoria ha UN solo proprietario (owner).
Il proprietario è responsabile di chiamare free().
```

### Esempio Base

```c
// ✅ OWNERSHIP CHIARO
void function() {
    int *p = malloc(100 * sizeof(int));  // function() è OWNER
    if (p == NULL) return;
    
    // ... uso p ...
    
    free(p);  // function() libera (è responsabile)
}
```

---

## 2. OWNERSHIP PATTERNS

### Pattern 1: CREATORE È OWNER

```c
/**
 * @ownership CALLER
 * Il chiamante è proprietario della stringa ritornata e DEVE fare free()
 */
char *create_string(const char *src) {
    char *str = malloc(strlen(src) + 1);
    if (str == NULL) return NULL;
    strcpy(str, src);
    return str;  // Trasferisce ownership al chiamante
}

// USO:
char *s = create_string("hello");
if (s != NULL) {
    printf("%s\n", s);
    free(s);  // ← CHIAMANTE libera (è owner)
}
```

### Pattern 2: CONSUMATORE È OWNER

```c
/**
 * @ownership CALLEE
 * La funzione prende ownership e libera la memoria
 */
void consume_string(char *str) {
    printf("%s\n", str);
    free(str);  // ← CALLEE libera (prende ownership)
    // Chiamante NON deve più usare str!
}

// USO:
char *s = malloc(100);
strcpy(s, "hello");
consume_string(s);  // Trasferisce ownership
// s è ora invalido! Non usarlo
```

### Pattern 3: OWNERSHIP CONDIVISO (EVITARE!)

```c
// ❌ PATTERN PERICOLOSO: Ownership ambiguo
char *global_ptr = NULL;

void set_data(char *data) {
    global_ptr = data;  // Chi libera? set_data o chiamante?
}

// ✅ SOLUZIONE: Documentare ownership
/**
 * @ownership SHARED
 * global_ptr condivide ownership. Chiamante deve chiamare free_global()
 */
void set_data_clear(char *data) {
    global_ptr = data;
}

void free_global() {
    free(global_ptr);
    global_ptr = NULL;
}
```

---

## 3. LIFETIME

### 3.1 Cos'è il Lifetime?

**Lifetime** = Periodo durante cui una variabile/memoria è valida.

### Lifetime delle Variabili

```c
// SCOPE GLOBALE: lifetime = intera esecuzione programma
int global_var = 10;

void function() {
    // SCOPE LOCALE: lifetime = durata funzione
    int local_var = 20;
    
    // STATIC: lifetime = intera esecuzione, ma scope locale
    static int static_var = 30;
    
    // HEAP: lifetime = da malloc a free
    int *heap_var = malloc(sizeof(int));
    *heap_var = 40;
    
    // FINE FUNZIONE:
    // - local_var distrutto
    // - static_var persiste
    // - heap_var puntatore distrutto, ma memoria persiste!
    
    free(heap_var);  // Libera memoria heap
}
```

### Diagramma Lifetime

```
TIMELINE:
─────────────────────────────────────────────────────────→
         ┌─ main() ──────────────────────────────┐
         │                                       │
Inizio   │   ┌─ function() ─────────────┐       │   Fine
Programma│   │                           │       │   Programma
         │   │  local_var  ──────────────┤       │
         │   │  heap_var (ptr) ──────────┤       │
         │   │  heap memory ─────────────────┐   │
         │   └───────────────────────────┘   │   │
         │                                   │   │
global_var ────────────────────────────────────────────
static_var ────────────────────────────────────────────
         │                                   │   │
         └───────────────────────────────────┴───┘
                                             ↑
                                        free() qui
```

---

## 4. DANGLING POINTERS

### 4.1 Cos'è un Dangling Pointer?

Puntatore che punta a memoria non più valida.

### Causa 1: Puntare a Variabile Locale

```c
// ❌ DISASTRO: Ritorna puntatore a locale
int *bad_function() {
    int x = 42;
    return &x;  // BUG! x distrutto al return
}

/* LIFETIME:
 * ┌─ bad_function ──┐
 * │  x = 42         │
 * └─────────────────┘
 *         ↑
 * Dopo return: x NON ESISTE PIÙ!
 * Ma puntatore ritornato punta ancora lì!
 */

// USO (ERRORE):
int *p = bad_function();
printf("%d\n", *p);  // UB! p è dangling

// ✅ SOLUZIONE: Usa heap
int *good_function() {
    int *x = malloc(sizeof(int));
    if (x == NULL) return NULL;
    *x = 42;
    return x;  // OK: x persiste dopo return
}

// Chiamante DEVE free()
int *p = good_function();
if (p != NULL) {
    printf("%d\n", *p);
    free(p);
}
```

### Causa 2: Use-After-Free

```c
// ❌ DANGLING dopo free
int *p = malloc(sizeof(int));
*p = 42;
free(p);
printf("%d\n", *p);  // UB! p è dangling

/* LIFETIME MEMORIA:
 * malloc() ─────┬─────────────┐
 *               │  *p = 42;   │
 *               │  VALIDA     │
 * free()  ──────┴─────────────┘
 *                      ↓
 *               *p → INVALIDA!
 */

// ✅ SOLUZIONE: Nullifica
free(p);
p = NULL;  // Ora *p causerà crash immediato (meglio di UB)
```

---

## 5. BORROWING

### 5.1 Cos'è il Borrowing?

**Borrowing** = Passaggio temporaneo di accesso SENZA trasferire ownership.

```c
// BORROW: Funzione usa stringa ma NON prende ownership
void print_string(const char *str) {
    printf("%s\n", str);
    // NON fa free(str)!
}

// USO:
char *s = create_string("hello");
print_string(s);  // ← BORROW: accesso temporaneo
// s è ancora valido qui
free(s);  // Chiamante libera (è ancora owner)
```

### Regole del Borrowing

```
1. Funzione che fa BORROW riceve puntatore
2. NON libera la memoria
3. Può solo LEGGERE (const) o MODIFICARE
4. Lifetime: solo durante la chiamata
5. Ownership rimane al chiamante
```

### Borrow Mutabile vs Immutabile

```c
// BORROW IMMUTABILE: Solo lettura
void read_data(const int *data, size_t size) {
    for (size_t i = 0; i < size; i++) {
        printf("%d ", data[i]);  // OK: lettura
    }
    // data[0] = 10;  ← ERRORE: const!
}

// BORROW MUTABILE: Può modificare
void modify_data(int *data, size_t size) {
    for (size_t i = 0; i < size; i++) {
        data[i] *= 2;  // OK: modifica
    }
}

// USO:
int *arr = malloc(5 * sizeof(int));
// ... inizializza arr ...
read_data(arr, 5);    // Borrow immutabile
modify_data(arr, 5);  // Borrow mutabile
free(arr);  // Ancora owner
```

---

## 6. OWNERSHIP IN STRUCT

### Pattern: Struct Possiede Dati

```c
typedef struct {
    char *name;   // Struct POSSIEDE name
    int *data;    // Struct POSSIEDE data
    size_t size;
} Container;

// CREAZIONE: Alloca struct E suoi dati
Container *create_container(const char *name, size_t size) {
    Container *c = malloc(sizeof(Container));
    if (c == NULL) return NULL;
    
    c->name = malloc(strlen(name) + 1);
    if (c->name == NULL) {
        free(c);
        return NULL;
    }
    strcpy(c->name, name);
    
    c->data = malloc(size * sizeof(int));
    if (c->data == NULL) {
        free(c->name);
        free(c);
        return NULL;
    }
    c->size = size;
    
    return c;
}

// DISTRUZIONE: Libera struct E suoi dati
void destroy_container(Container *c) {
    if (c != NULL) {
        free(c->name);  // Libera name
        free(c->data);  // Libera data
        free(c);        // Libera struct
    }
}

// USO:
Container *c = create_container("test", 10);
if (c != NULL) {
    // ... uso c ...
    destroy_container(c);  // Libera TUTTO
}
```

### Diagramma Ownership

```
┌─ Chiamante ─────────────────────────┐
│                                     │
│  c → Container                      │  ← Owner di Container
│       ├─ name → "test"              │  ← Container owner di name
│       ├─ data → [0][1][2]...[9]     │  ← Container owner di data
│       └─ size = 10                  │
│                                     │
│  destroy_container(c):              │
│    1. free(c->name)                 │
│    2. free(c->data)                 │
│    3. free(c)                       │
└─────────────────────────────────────┘
```

---

## 7. BEST PRACTICES

### Rule 1: Documenta Ownership

```c
/**
 * @ownership CALLER - Chiamante deve free() il risultato
 */
char *allocate_string(size_t len);

/**
 * @ownership CALLEE - Funzione libera il parametro
 */
void consume_string(char *str);

/**
 * @ownership NONE - Funzione fa solo borrow (non libera)
 */
void print_string(const char *str);
```

### Rule 2: Un Owner, Una Responsabilità

```c
// ❌ AMBIGUO:
void unclear_function(char *data) {
    // Chi libera data?
}

// ✅ CHIARO:
/**
 * @ownership NONE - data fa borrow
 */
void process_data(const char *data) {
    // Legge solo, non libera
}
```

### Rule 3: Nullifica Dopo Free

```c
// ✅ PATTERN SAFE:
free(ptr);
ptr = NULL;  // Previene dangling pointer

if (ptr != NULL) {
    *ptr = 10;  // Questo non si eseguirà
}
```

### Rule 4: RAII-like Pattern

```c
// PATTERN: Acquisizione e rilascio nello stesso scope
void function() {
    char *data = malloc(100);  // ← Acquisizione
    if (data == NULL) return;
    
    // ... uso data ...
    
    free(data);  // ← Rilascio nello stesso scope
}
```

---

## 8. ANTI-PATTERNS COMUNI

### Anti-Pattern 1: Double Free

```c
// ❌ DISASTRO:
free(ptr);
free(ptr);  // UB! Memoria già liberata

// ✅ SOLUZIONE:
free(ptr);
ptr = NULL;
if (ptr != NULL) {  // Protezione
    free(ptr);
}
```

### Anti-Pattern 2: Ownership Condiviso Implicito

```c
// ❌ AMBIGUO:
char *shared = create_string("data");
function_a(shared);  // Libera?
function_b(shared);  // Libera?
// Chi libera shared?

// ✅ ESPLICITO:
char *data = create_string("data");
process_data(data);  // Borrow (NON libera)
analyze_data(data);  // Borrow (NON libera)
free(data);  // Owner libera
```

### Anti-Pattern 3: Lifetime Mismatches

```c
// ❌ PERICOLOSO:
char *get_static() {
    static char buf[100];
    return buf;  // OK per lifetime, MA...
}

char *s1 = get_static();
strcpy(s1, "first");
char *s2 = get_static();
strcpy(s2, "second");
printf("%s\n", s1);  // "second"! s1 e s2 puntano stesso buffer!

// ✅ SOLUZIONE: Usa heap
char *get_new() {
    char *buf = malloc(100);
    return buf;  // Ogni chiamata = nuovo buffer
}
```

---

## 9. 🔧 ESERCIZI

### Esercizio 1: Identifica Owner

```c
char *create() { return malloc(100); }
void process(char *s) { printf("%s", s); }
void destroy(char *s) { free(s); }

int main() {
    char *s = create();
    process(s);
    destroy(s);
}
```

<details>
<summary>Soluzione</summary>

```
create(): Crea e trasferisce ownership → CALLER
process(): Fa borrow, NON owner
destroy(): Consuma (libera), prende ownership
main(): Owner iniziale, poi trasferisce a destroy()

CORRETTA gestione ownership!
```
</details>

### Esercizio 2: Trova Dangling Pointer

```c
int *func1() {
    int x = 10;
    return &x;
}

int *func2() {
    int *p = malloc(sizeof(int));
    *p = 20;
    free(p);
    return p;
}
```

<details>
<summary>Soluzione</summary>

```
func1(): Ritorna &x dove x è locale → DANGLING!
        x distrutto al return.

func2(): Ritorna p dopo free(p) → DANGLING!
        Memoria liberata ma puntatore ritornato.

ENTRAMBE CAUSANO UB!

FIX func1(): malloc invece di locale
FIX func2(): rimuovi free() (trasferisci ownership)
```
</details>

---

## 10. 🎓 PUNTI CHIAVE

**OWNERSHIP:**
- Ogni risorsa ha UN owner
- Owner è responsabile di liberare
- Documenta ownership nelle funzioni
- Trasferimento ownership deve essere esplicito

**LIFETIME:**
- Stack: lifetime = scope funzione
- Heap: lifetime = da malloc a free
- Static/Global: lifetime = programma intero
- Non ritornare puntatori a locali!

**BORROWING:**
- Accesso temporaneo senza ownership
- Usa const per borrow immutabile
- Lifetime borrow = durata chiamata
- Non liberare memoria in borrow!

**REGOLA D'ORO:**
> CHI ALLOCA, DEALLOCA
> (a meno che ownership sia trasferito esplicitamente)

---

[← Torna al Modulo](README.md) | [Modulo 04 Avanzato →](../04_Avanzato/README.md)
