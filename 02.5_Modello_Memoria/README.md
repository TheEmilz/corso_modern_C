# Modulo 02.5 - Il Modello Mentale della Memoria

## 🎯 Obiettivo del Modulo

Questo modulo è fondamentale per sviluppare una **visualizzazione profonda** di come la memoria funziona in C. A differenza di linguaggi ad alto livello che nascondono i dettagli, C ti da controllo completo sulla memoria - ma con questo potere viene la responsabilità di comprendere esattamente cosa succede "dietro le quinte".

**Alla fine di questo modulo sarai in grado di:**
- Visualizzare mentalmente lo stack frame di ogni funzione
- Comprendere il layout esatto in memoria di ogni tipo di dato
- Ragionare su cache locality e allineamento dati
- Anticipare come il compilatore organizza la memoria
- Debuggare problemi di memoria con consapevolezza profonda

---

## 📚 Perché Questo Modulo è Cruciale

La maggior parte dei bug in C deriva da una **mancanza di modello mentale della memoria**:

```
❌ ERRORI COMUNI CHE DERIVANO DA INCOMPRENSIONE DELLA MEMORIA:

1. Buffer overflow        → Non visualizzi i limiti dell'array
2. Use-after-free         → Non comprendi il lifetime
3. Memory leak            → Non tracci ownership
4. Dangling pointer       → Non capisci quando la memoria viene liberata
5. Stack overflow         → Non visualizzi la crescita dello stack
6. Alignment errors       → Non conosci il padding
7. Endianness bugs        → Non capisci la rappresentazione byte
```

**Questo modulo costruisce il "sesto senso" della memoria** che distingue un programmatore C competente da un maestro.

---

## 🗺️ Struttura del Modulo

### [Lezione 1: Stack vs Heap - Visualizzazione Completa](01_stack_vs_heap.md)

**Domande a cui risponderemo:**
- Come funziona lo stack? (LIFO, stack frames, crescita)
- Come funziona l'heap? (allocatore, frammentazione)
- Cosa succede durante una chiamata a funzione?
- Cosa causa stack overflow?
- Come si manifesta la frammentazione dell'heap?

**Visualizzazioni:**
```
┌─ MEMORIA COMPLETA ────────────────────────────────┐
│                                                    │
│  CODE (.text)      ← Istruzioni macchina          │
│  ─────────────                                     │
│  RODATA (.rodata)  ← Stringhe letterali, const    │
│  ─────────────                                     │
│  DATA (.data)      ← Variabili globali inizializ. │
│  ─────────────                                     │
│  BSS (.bss)        ← Variabili globali non init.  │
│  ─────────────                                     │
│                                                    │
│  HEAP  ↓           ← malloc/free (cresce verso ↓) │
│  ═════════                                         │
│        ...         ← Spazio libero                 │
│  ═════════                                         │
│  STACK ↑           ← Variabili locali (verso ↑)   │
│  ─────────────                                     │
│  ENV/ARGS          ← Variabili ambiente           │
│                                                    │
└────────────────────────────────────────────────────┘
```

**Contenuti:**
- Anatomia di uno stack frame
- Call stack e return address
- Allocazione heap: malloc internals
- Frammentazione (interna ed esterna)
- Quando usare stack vs heap

---

### [Lezione 2: Allineamento e Padding](02_allineamento_padding.md)

**Domande a cui risponderemo:**
- Perché esiste il padding?
- Come calcolare sizeof di una struct?
- Cosa è l'allineamento dei dati?
- Come ottimizzare struct per ridurre padding?
- Cosa sono cache line e false sharing?

**Visualizzazioni:**
```
STRUCT SENZA OTTIMIZZAZIONE:
struct Example {
    char a;      // 1 byte
    // [3 byte PADDING]
    int b;       // 4 byte
    char c;      // 1 byte
    // [3 byte PADDING]
    double d;    // 8 byte
};
sizeof(Example) = 20 byte (SPRECO: 6 byte di padding!)

MEMORIA:
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ a │ X │ X │ X │    b    │ c │ X │ X │ X │        d        │
└───┴───┴───┴───┴───────┴───┴───┴───┴───┴─────────────────┘
  1   padding   4         1   padding   8

STRUCT OTTIMIZZATA:
struct ExampleOptimized {
    double d;    // 8 byte
    int b;       // 4 byte
    char a;      // 1 byte
    char c;      // 1 byte
    // [2 byte PADDING]
};
sizeof(ExampleOptimized) = 16 byte (SPRECO: solo 2 byte!)
```

**Contenuti:**
- Regole di allineamento per ogni tipo
- Padding in struct e union
- Pragma pack e __attribute__((packed))
- Cache line (tipicamente 64 byte)
- Performance impact dell'allineamento

---

### [Lezione 3: Endianness e Rappresentazione Binaria](03_endianness.md)

**Domande a cui risponderemo:**
- Cos'è Little-Endian vs Big-Endian?
- Come vedere i byte in memoria?
- Perché l'endianness importa?
- Come convertire tra endianness?
- Cosa è network byte order?

**Visualizzazioni:**
```
NUMERO: int x = 0x12345678 (hex)

LITTLE-ENDIAN (x86, ARM moderno):
┌────────┬────────┬────────┬────────┐
│  0x78  │  0x56  │  0x34  │  0x12  │  ← Byte meno significativo PRIMO
└────────┴────────┴────────┴────────┘
0x1000   0x1001   0x1002   0x1003

BIG-ENDIAN (Network, alcuni ARM):
┌────────┬────────┬────────┬────────┐
│  0x12  │  0x34  │  0x56  │  0x78  │  ← Byte più significativo PRIMO
└────────┴────────┴────────┴────────┘
0x1000   0x1001   0x1002   0x1003

LETTURA:
Little-Endian: Leggi da dx → sx (0x78563412 → 0x12345678)
Big-Endian: Leggi da sx → dx (0x12345678 → 0x12345678)
```

**Contenuti:**
- Rappresentazione binaria di int, float, double
- Visualizzare byte in memoria con debugger
- Union trick per vedere rappresentazione
- htonl/htons e ntohl/ntohs (conversioni rete)
- Portabilità tra architetture

---

### [Lezione 4: Segmenti di Memoria (Memory Segments)](04_segmenti_memoria.md)

**Domande a cui risponderemo:**
- Dove vanno le diverse variabili?
- Cos'è .text, .data, .bss, .rodata?
- Cosa sono le sezioni in un eseguibile?
- Come visualizzare i segmenti con tools?
- Differenza tra virtual memory e physical memory?

**Visualizzazioni:**
```
CODICE:
const char *msg = "Hello";  // Dove va msg? Dove va "Hello"?
static int counter = 0;     // Dove va?
int global = 42;            // Dove va?

void function() {
    int local = 10;         // Dove va?
    static int s = 0;       // Dove va?
    int *ptr = malloc(100); // Dove vanno ptr e il blocco allocato?
}

MEMORIA:
┌─────────────────────────────────────────┐
│ .text (CODE SEGMENT)                    │
│ ─────────────────────                   │
│ Codice macchina di function()           │ ← Read-Only, Executable
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ .rodata (READ-ONLY DATA)                │
│ ─────────────────────                   │
│ "Hello\0"  ← La stringa letterale       │ ← Read-Only
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ .data (INITIALIZED DATA)                │
│ ─────────────────────                   │
│ global = 42                             │ ← Read-Write
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ .bss (UNINITIALIZED DATA)               │
│ ─────────────────────                   │
│ counter = 0 (inizializzato a 0 auto)    │ ← Read-Write
│ s = 0       (static in function)        │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ HEAP                                    │
│ ─────────────────────                   │
│ Blocco 100 byte (malloc)  ← *ptr punta  │ ← Read-Write, Dynamic
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ STACK                                   │
│ ─────────────────────                   │
│ msg (puntatore) → punta a .rodata       │
│ local = 10                              │ ← Read-Write, Auto
│ ptr (puntatore) → punta a heap          │
└─────────────────────────────────────────┘
```

**Contenuti:**
- Linker e loader: come si costruisce l'eseguibile
- Tools: objdump, readelf, size, nm
- Memory mapping (mmap)
- Copy-on-write e virtual memory
- Protezioni memoria (NX bit, ASLR)

---

## 🎯 Approccio Pedagogico

Ogni lezione segue il framework avanzato:

### 1. VISUALIZZAZIONE CONTINUA
Ogni concetto include:
- Diagrammi ASCII della memoria
- Layout byte-by-byte
- Indirizzi espliciti
- Frecce e relazioni

### 2. PENSA COME IL COMPILATORE
Per ogni scelta:
- Perché il compilatore fa così?
- Quali ottimizzazioni applica?
- Cosa generà in assembly?

### 3. DEBUG SISTEMATICO
Per ogni concetto:
- Errori comuni
- Come riconoscerli
- Tools per investigare (gdb, valgrind, objdump)

### 4. ESEMPI INTERATTIVI
- 🤔 FERMATI E PENSA prima di vedere la soluzione
- Tracciamento passo-passo
- Confronti (cosa cambia se...)

---

## 🔧 Tool che Useremo

Durante questo modulo imparerai a usare:

```
GDB (Debugger):
  - Visualizzare memoria: x/10xb 0x1000
  - Stack frames: backtrace, frame
  - Registri: info registers

Valgrind:
  - Memory leak detection
  - Invalid memory access
  - Uninitialized values

objdump/readelf:
  - Vedere segmenti: readelf -S executable
  - Disassemblare: objdump -d executable
  - Simboli: nm executable

size:
  - Dimensione segmenti: size executable

hexdump:
  - Vedere byte di un file: hexdump -C file
```

---

## 💡 Perché Questo Modulo Ora?

**POSIZIONAMENTO:** Dopo Modulo 02 (Fondamenti) ma prima di Modulo 03 (Intermedio)

**RATIONALE:**
1. Hai già visto la sintassi base (Modulo 02)
2. Hai concetti di puntatori e array
3. PRIMA di allocazione dinamica profonda (Modulo 03)
4. Ti prepara per ownership e lifetime

Questo modulo **trasforma** la tua comprensione di:
- Puntatori: non sono "magia", sono indirizzi
- Array: non sono "astrazioni", sono memoria contigua
- Funzioni: non sono "black box", sono stack frame
- malloc: non è "keyword", è chiamata all'allocatore

---

## 🎓 Risultati Attesi

Dopo questo modulo, quando vedi codice C, **visualizzerai automaticamente**:

```c
int x = 42;
int *p = &x;
*p = 100;
```

**La tua mente:**
```
┌─ Stack @ 0x7fffffffdc00 ──┐
│ x: [00 00 00 2A]          │  ← 42 in little-endian
│    @ 0x7fffffffdc00       │
├───────────────────────────┤
│ p: [00 C0 DC FF FF 7F 00]│  ← Puntatore a x
│    @ 0x7fffffffdc04       │
└───────────────────────────┘

*p = 100:
1. Leggi p → 0x7fffffffdc00
2. Scrivi 100 @ 0x7fffffffdc00
3. x ora: [00 00 00 64]  ← 100 in hex
```

**Questa è maestria del C!** 🎯

---

## 📝 Come Studiare Questo Modulo

1. **Leggi con carta e penna**
   - Disegna i diagrammi tu stesso
   - Traccia l'esecuzione passo-passo

2. **Usa i tool**
   - Compila gli esempi
   - Usa gdb per vedere la memoria
   - Usa objdump per i segmenti

3. **Sperimenta**
   - Modifica gli esempi
   - Prevedi cosa succede
   - Verifica con il debugger

4. **Insegna a qualcuno**
   - Se puoi spiegare i concetti, li hai compresi
   - Disegna i diagrammi per un amico

---

## 🚀 Iniziamo!

Pronto a sviluppare il tuo "sesto senso della memoria"?

**Inizia dalla [Lezione 1: Stack vs Heap →](01_stack_vs_heap.md)**

---

[← Modulo 02: Fondamenti](../02_Fondamenti/README.md) | [Torna al Sommario](../README.md) | [Modulo 03: Intermedio →](../03_Intermedio/README.md)
