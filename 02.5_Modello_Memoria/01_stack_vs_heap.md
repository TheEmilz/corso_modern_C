# Lezione 1 - Stack vs Heap: Visualizzazione Completa

## 🎯 Obiettivi

Alla fine di questa lezione sarai in grado di:
- Visualizzare mentalmente la struttura della memoria
- Comprendere come funziona lo stack (LIFO, stack frames)
- Comprendere come funziona l'heap (allocatore, frammentazione)
- Tracciare le chiamate a funzioni nello stack
- Riconoscere quando usare stack vs heap

---

## 1. IL GRANDE QUADRO: Memoria di un Processo

Quando un programma C viene eseguito, il sistema operativo gli assegna uno **spazio di indirizzi virtuali**:

```
MEMORIA VIRTUALE (Tipico processo a 64-bit):

Indirizzi Alti (0x7FFF...)
┌────────────────────────────────────────┐
│  STACK                                 │  ← Variabili locali, parametri
│  │                                     │     Cresce verso il BASSO ↓
│  │                                     │
│  ↓                                     │
├────────────────────────────────────────┤
│                                        │
│         SPAZIO LIBERO                  │  ← "Hole" tra stack e heap
│                                        │
├────────────────────────────────────────┤
│  ↑                                     │
│  │                                     │
│  │                                     │
│  HEAP                                  │  ← malloc/calloc/realloc
├────────────────────────────────────────┤     Cresce verso l'ALTO ↑
│  BSS (Uninitialized Data)             │  ← static int x; (= 0 automatico)
├────────────────────────────────────────┤
│  DATA (Initialized Data)               │  ← static int x = 42;
├────────────────────────────────────────┤     int global = 10;
│  RODATA (Read-Only Data)               │  ← const char *s = "Hello";
├────────────────────────────────────────┤     (la stringa "Hello")
│  TEXT (Code Segment)                   │  ← Codice macchina (istruzioni)
└────────────────────────────────────────┘
Indirizzi Bassi (0x0000...)

CARATTERISTICHE:
✓ Stack e Heap crescono in direzioni OPPOSTE
✓ Se si incontrano → Out of Memory!
✓ Segmenti hanno PERMESSI diversi (r/w/x)
```

---

## 2. LO STACK: Last In, First Out

### 2.1 Cos'è lo Stack?

Lo **stack** è una regione di memoria gestita **automaticamente** dal sistema per:
- Variabili locali
- Parametri di funzione
- Indirizzi di ritorno (return addresses)
- Frame pointer

```
PRINCIPIO LIFO (Last In, First Out):
Come una pila di piatti: l'ultimo aggiunto è il primo rimosso.

OPERAZIONI:
- PUSH: Aggiungi elemento in cima (sp--)
- POP: Rimuovi elemento dalla cima (sp++)

GESTIONE:
✓ AUTOMATICA: allocazione/deallocazione gestita dal compilatore
✓ VELOCE: solo incremento/decremento di un puntatore (stack pointer)
✓ LIMITATA: dimensione tipica 1-8 MB
```

### 2.2 Anatomia di uno Stack Frame

Ogni **chiamata a funzione** crea un nuovo **stack frame**:

```
CODICE:
int somma(int a, int b) {
    int risultato = a + b;
    return risultato;
}

int main(void) {
    int x = 5;
    int y = 10;
    int z = somma(x, y);
    return 0;
}

STACK DURANTE L'ESECUZIONE:

┌─ INIZIO: main() ──────────────────────────┐
│                                            │
│ ┌────────────────────────────────────┐    │
│ │ STACK FRAME di main()              │    │
│ │ ──────────────────────              │    │
│ │ y = 10     @ 0x7fff...04           │    │
│ │ x = 5      @ 0x7fff...00  ← SP     │    │
│ │ (return address to OS)              │    │
│ └────────────────────────────────────┘    │
│                                            │
└────────────────────────────────────────────┘

┌─ DURANTE: somma(5, 10) ───────────────────┐
│                                            │
│ ┌────────────────────────────────────┐    │
│ │ STACK FRAME di somma()             │    │
│ │ ──────────────────────              │    │
│ │ risultato = 15  @ 0x7fff...f8      │    │
│ │ b = 10          @ 0x7fff...f4      │    │
│ │ a = 5           @ 0x7fff...f0      │    │
│ │ return addr     @ 0x7fff...ec ← SP │    │
│ └────────────────────────────────────┘    │
│ ┌────────────────────────────────────┐    │
│ │ STACK FRAME di main()              │    │
│ │ ──────────────────────              │    │
│ │ z = ???    (non ancora assegnato)  │    │
│ │ y = 10     @ 0x7fff...04           │    │
│ │ x = 5      @ 0x7fff...00           │    │
│ │ (return address to OS)              │    │
│ └────────────────────────────────────┘    │
│                                            │
└────────────────────────────────────────────┘

┌─ DOPO: return di somma() ─────────────────┐
│                                            │
│ ┌────────────────────────────────────┐    │
│ │ STACK FRAME di main()              │    │
│ │ ──────────────────────              │    │
│ │ z = 15     @ 0x7fff...08  ← SP     │    │
│ │ y = 10     @ 0x7fff...04           │    │
│ │ x = 5      @ 0x7fff...00           │    │
│ │ (return address to OS)              │    │
│ └────────────────────────────────────┘    │
│                                            │
│ Frame di somma() DISTRUTTO automaticamente!│
└────────────────────────────────────────────┘

COMPONENTI STACK FRAME:
1. Variabili locali (risultato, x, y, z)
2. Parametri (a, b)
3. Return address (dove tornare dopo la chiamata)
4. Frame pointer salvato (per ricostruire frame chiamante)
```

### 2.3 Cosa Succede Durante una Chiamata

```
CHIAMATA: z = somma(x, y);

ASSEMBLY (concettuale):
┌────────────────────────────────────────────┐
│ 1. PUSH parametri sullo stack              │
│    push y (10)                             │
│    push x (5)                              │
│                                            │
│ 2. CALL somma                              │
│    - Push return address sullo stack       │
│    - Jump a somma()                        │
│                                            │
│ 3. SETUP frame di somma                    │
│    push rbp  (salva frame pointer)         │
│    mov rbp, rsp  (nuovo frame pointer)     │
│    sub rsp, 16   (alloca spazio locali)    │
│                                            │
│ 4. ESEGUI corpo di somma()                 │
│    mov eax, [rbp+8]   ; a                  │
│    add eax, [rbp+12]  ; a + b              │
│    mov [rbp-4], eax   ; risultato = a+b    │
│                                            │
│ 5. CLEANUP e RETURN                        │
│    mov eax, [rbp-4]   ; return risultato   │
│    mov rsp, rbp       ; ripristina SP      │
│    pop rbp            ; ripristina FP      │
│    ret                ; pop return addr     │
│                                            │
│ 6. BACK in main                            │
│    mov [rbp-12], eax  ; z = risultato      │
│    add rsp, 8         ; rimuovi parametri  │
└────────────────────────────────────────────┘

COMPLESSITÀ TEMPORALE: O(1) per push/pop!
```

### 2.4 Stack Overflow

```
PROBLEMA: Ricorsione troppo profonda

void ricorsione_infinita(int n) {
    int arr[1000];  // 4000 byte per chiamata
    printf("%d\n", n);
    ricorsione_infinita(n + 1);  // NESSUN CASO BASE!
}

STACK:
┌──────────────────┐
│ ricorsione(1000) │
├──────────────────┤
│ ricorsione(999)  │
├──────────────────┤
│ ...              │  ← Stack cresce...
├──────────────────┤
│ ricorsione(2)    │
├──────────────────┤
│ ricorsione(1)    │
├──────────────────┤
│ main()           │
└──────────────────┘

DOPO ~2000 chiamate (con arr[1000]):
4000 byte/chiamata × 2000 = 8 MB
→ STACK OVERFLOW! Segmentation Fault

SINTOMI:
✗ Segmentation fault
✗ "Stack overflow detected"
✗ Crash improvviso

SOLUZIONI:
✓ Limita profondità ricorsione
✓ Usa iterazione
✓ Alloca grandi array sull'heap
```

---

## 3. L'HEAP: Allocazione Dinamica

### 3.1 Cos'è l'Heap?

L'**heap** è una regione di memoria per **allocazione dinamica**:

```
CARATTERISTICHE:
✓ DIMENSIONE FLESSIBILE: allochi quanto serve
✓ LIFETIME CONTROLLATO: tu decidi quando liberare
✓ CONDIVISO: accessibile da tutte le funzioni
✗ GESTIONE MANUALE: malloc/free a tua cura
✗ PIÙ LENTO: richiede ricerca spazio libero
✗ FRAMMENTAZIONE: può sprecare memoria

OPERAZIONI:
- malloc(size): Alloca size byte
- calloc(n, size): Alloca e inizializza a 0
- realloc(ptr, new_size): Ridimensiona blocco
- free(ptr): Libera memoria
```

### 3.2 Come Funziona malloc()

```
HEAP INTERNALS (semplificato):

┌─ HEAP ─────────────────────────────────────┐
│                                            │
│ ┌──────────────────┐  ┌─────────────────┐ │
│ │ BLOCCO LIBERO    │  │ BLOCCO USATO    │ │
│ │ Size: 1024 byte  │  │ Size: 256 byte  │ │
│ │ Next: → ptr      │  │ Data: [...]     │ │
│ └──────────────────┘  └─────────────────┘ │
│          ↓                                 │
│ ┌──────────────────┐  ┌─────────────────┐ │
│ │ BLOCCO LIBERO    │  │ BLOCCO USATO    │ │
│ │ Size: 2048 byte  │  │ Size: 512 byte  │ │
│ │ Next: NULL       │  │ Data: [...]     │ │
│ └──────────────────┘  └─────────────────┘ │
│                                            │
└────────────────────────────────────────────┘

ALLOCATORE (malloc) mantiene:
- FREE LIST: Lista blocchi liberi
- METADATA: Ogni blocco ha header (size, free/used)

CHIAMATA malloc(100):
1. Cerca nella free list blocco >= 100 byte
2. Se trovato:
   - Rimuovi da free list
   - Split se blocco troppo grande
   - Ritorna puntatore
3. Se NON trovato:
   - Richiedi più memoria al OS (sbrk/mmap)
   - Aggiungi alla heap
   - Ritorna puntatore

METADATA OVERHEAD: Tipicamente 8-16 byte per blocco
malloc(10) → usa ~26 byte (10 dati + 16 metadata)
```

### 3.3 Frammentazione

**FRAMMENTAZIONE ESTERNA:** Spazio totale sufficiente, ma disperso

```
SCENARIO:
malloc(100) → A
malloc(100) → B
malloc(100) → C
free(B)

HEAP:
┌────────┬────────┬────────┐
│   A    │ LIBERO │   C    │
│ 100 B  │ 100 B  │ 100 B  │
└────────┴────────┴────────┘

malloc(200) → FALLISCE!
Anche se c'è spazio libero totale, 
NON è contiguo: serve blocco di 200 byte continui.

SOLUZIONI:
- Compattazione (costoso, raramente fatto)
- Migliori strategie allocazione (best-fit, worst-fit)
- Pool allocator per oggetti stessa dimensione
```

**FRAMMENTAZIONE INTERNA:** Spreco dentro il blocco

```
malloc(10) su allocatore che usa blocchi da 16 byte:

┌────────────────┐
│ Usati: 10 B    │
│ Spreco: 6 B    │ ← FRAMMENTAZIONE INTERNA
│ (+ metadata)   │
└────────────────┘

NON recuperabile: lo spreco è "intrappolato" nel blocco.
```

### 3.4 Memory Leak

```
// ❌ DISASTRO: Memory Leak
void function() {
    int *ptr = malloc(1000 * sizeof(int));
    // ... uso ptr ...
    // NESSUN free(ptr)!
}  // ptr esce dallo scope

/* MEMORIA DOPO function():
 * 
 * STACK: ptr è sparito
 * HEAP: 4000 byte ancora allocati!
 * 
 * NON HAI PIÙ IL PUNTATORE → MEMORIA PERSA!
 * 
 * Se chiami function() 1000 volte:
 * 4 MB di memoria PERSI (leak)!
 */

// ✅ SOLUZIONE:
void function() {
    int *ptr = malloc(1000 * sizeof(int));
    if (ptr == NULL) {
        // Gestisci errore
        return;
    }
    // ... uso ptr ...
    free(ptr);  // ← FONDAMENTALE!
    ptr = NULL; // Best practice
}
```

---

## 4. CONFRONTO: Stack vs Heap

```
┌─────────────────────┬────────────────┬──────────────────┐
│  CARATTERISTICA     │     STACK      │      HEAP        │
├─────────────────────┼────────────────┼──────────────────┤
│ Allocazione         │ Automatica     │ Manuale (malloc) │
│ Deallocazione       │ Automatica     │ Manuale (free)   │
│ Velocità            │ O(1) ⚡        │ O(n) variabile   │
│ Dimensione          │ Piccola (MB)   │ Grande (GB)      │
│ Lifetime            │ Scope funzione │ Controllato      │
│ Frammentazione      │ No ✓           │ Sì ✗             │
│ Overhead            │ Minimo         │ Metadata         │
│ Accesso             │ Locale         │ Globale          │
│ Thread-safe         │ Sì (per thread)│ Richiede sync    │
│ Rischi              │ Stack overflow │ Leak, corruption │
└─────────────────────┴────────────────┴──────────────────┘

🎯 USA STACK QUANDO:
  ✅ Dimensione piccola e conosciuta a compile-time
  ✅ Lifetime legato alla funzione
  ✅ Performance critiche

🎯 USA HEAP QUANDO:
  ✅ Dimensione grande o sconosciuta a compile-time
  ✅ Lifetime deve sopravvivere alla funzione
  ✅ Dati condivisi tra funzioni
  ✅ Strutture dati dinamiche (liste, alberi)
```

---

## 5. ESEMPI PRATICI

### Esempio 1: Variabile Locale (Stack)

```c
void example() {
    int x = 42;  // STACK: allocato all'ingresso in example()
    // x vive qui
}  // x deallocato automaticamente

/* MEMORIA:
 * DURANTE example():
 * ┌─ Stack ────────┐
 * │ x = 42         │
 * └────────────────┘
 * 
 * DOPO example():
 * ┌─ Stack ────────┐
 * │ (spazio riusato)│
 * └────────────────┘
 */
```

### Esempio 2: Array Grande (Heap vs Stack)

```c
// ❌ PROBLEMA: Array grande sullo stack
void bad() {
    int huge[1000000];  // 4 MB!
    // Rischio STACK OVERFLOW
}

// ✅ SOLUZIONE: Usa heap
void good() {
    int *huge = malloc(1000000 * sizeof(int));
    if (huge == NULL) {
        fprintf(stderr, "Out of memory!\n");
        return;
    }
    // ... usa huge ...
    free(huge);  // FONDAMENTALE
}
```

### Esempio 3: Ritornare Dati da Funzione

```c
// ❌ DISASTRO: Ritorna puntatore a stack!
int* bad_function() {
    int x = 42;
    return &x;  // BUG! x sarà distrutto!
}

// Chiamante:
int *ptr = bad_function();
printf("%d\n", *ptr);  // UNDEFINED BEHAVIOR!

/* MEMORIA:
 * Durante bad_function():
 * ┌─ Stack ────────┐
 * │ x = 42         │ ← ptr punta qui
 * └────────────────┘
 * 
 * Dopo return:
 * ┌─ Stack ────────┐
 * │ ??? (garbage)  │ ← ptr punta a memoria invalida!
 * └────────────────┘
 */

// ✅ SOLUZIONE 1: Usa heap
int* good_function() {
    int *x = malloc(sizeof(int));
    if (x == NULL) return NULL;
    *x = 42;
    return x;  // OK: x sull'heap persiste
}

// ✅ SOLUZIONE 2: Passa buffer
void good_function2(int *result) {
    *result = 42;  // Modifica parametro
}
```

---

## 6. 🔧 ESERCIZI

### 🟢 LIVELLO 1

**Esercizio 1:** Dove sono allocate queste variabili?
```c
int global = 10;        // ?
const char *msg = "Hi"; // msg e "Hi"?

void func() {
    int local = 5;          // ?
    static int counter = 0; // ?
    int *ptr = malloc(100); // ptr e blocco?
}
```

<details>
<summary>Soluzione</summary>

```
global → .data (dati inizializzati)
msg → Stack (puntatore), "Hi" → .rodata (stringa)
local → Stack
counter → .bss (static, inizializzato a 0)
ptr → Stack (puntatore), blocco 100 byte → Heap
```
</details>

### 🟡 LIVELLO 2

**Esercizio 2:** Correggi questo codice:
```c
char* create_string() {
    char str[100] = "Hello";
    return str;  // Bug?
}
```

<details>
<summary>Soluzione</summary>

```c
// ❌ PROBLEMA: str è su stack, distrutto al return

// ✅ SOLUZIONE:
char* create_string() {
    char *str = malloc(100);
    if (str == NULL) return NULL;
    strcpy(str, "Hello");
    return str;
}
// Chiamante DEVE fare free(str)!
```
</details>

---

## 7. 🎓 PUNTI CHIAVE

**STACK:**
- Automatico, veloce, limitato
- LIFO: ultimo entrato, primo uscito
- Stack frame per ogni chiamata
- Rischio: overflow con ricorsione profonda

**HEAP:**
- Manuale, flessibile, lento
- Gestione malloc/free
- Rischi: leak, frammentazione, corruption
- Necessario per dati che sopravvivono alla funzione

**REGOLA D'ORO:**
> Default → Stack (se possibile)
> Grande/Dinamico/Persistente → Heap

---

[Torna al Modulo](README.md) | [Prossima Lezione: Allineamento →](02_allineamento_padding.md)
