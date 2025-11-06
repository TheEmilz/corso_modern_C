# Fil-C: Il Compilatore per la Sicurezza della Memoria in C/C++

## Indice
- [Introduzione](#introduzione)
- [Architettura di Fil-C](#architettura-di-fil-c)
- [InvisiCaps: Il Modello delle Capabilities](#invisicaps-il-modello-delle-capabilities)
- [MonoCaps: Capabilities a 128-bit](#monocaps-capabilities-a-128-bit)
- [FUGC: Il Garbage Collector di Fil-C](#fugc-il-garbage-collector-di-fil-c)
- [Memory Safety: Teoria e Implementazione](#memory-safety-teoria-e-implementazione)
- [Estrazione e Gestione dei Puntatori](#estrazione-e-gestione-dei-puntatori)
- [Compatibilità e Performance](#compatibilità-e-performance)
- [Ricerca Accademica e Paper di Riferimento](#ricerca-accademica-e-paper-di-riferimento)
- [Link Utili e Risorse](#link-utili-e-risorse)

---

## Introduzione

Fil-C rappresenta un'innovazione rivoluzionaria nel campo della sicurezza della memoria per i linguaggi C e C++. Sviluppato come compilatore e runtime system open-source basato su LLVM, Fil-C affronta uno dei problemi più critici della programmazione di sistema: la gestione sicura della memoria.

### Il Problema della Memory Safety in C/C++

I linguaggi C e C++ sono notoriamente vulnerabili a errori di memoria che causano:
- **Buffer overflow**: accesso oltre i limiti di un array
- **Use-after-free**: utilizzo di memoria già deallocata
- **Dangling pointers**: puntatori a memoria non più valida
- **Double free**: deallocazione multipla della stessa memoria
- **Type confusion**: violazioni della sicurezza dei tipi

Questi bug non sono solo errori di programmazione, ma rappresentano la base di molti exploit di sicurezza critici. Secondo studi recenti, oltre il 70% delle vulnerabilità nei software di sistema derivano da problemi di memory safety.


### La Soluzione di Fil-C

Fil-C risolve questi problemi attraverso:

1. **Compatibilità Fanatica**: Supporta quasi tutto il codice C/C++ esistente con modifiche minime o nulle al sorgente
2. **Memory Safety Completa**: Cattura tutti gli errori di memoria a runtime, terminando il programma in modo controllato invece di causare comportamenti indefiniti
3. **Garbage Collection Automatico**: Elimina la gestione manuale della memoria tramite malloc/free
4. **Zero Escape Hatches**: Nessuna via di fuga dalle garanzie di sicurezza

---

## Architettura di Fil-C

### Struttura del Sistema

Fil-C è costruito come un sistema completo che integra:

```
┌─────────────────────────────────────────────┐
│           Codice Sorgente C/C++             │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│      Frontend Fil-C (basato su Clang)      │
│  • Parsing del codice sorgente              │
│  • Analisi semantica                        │
│  • Strumentazione automatica                │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│           LLVM IR Strumentato               │
│  • Ogni operazione su puntatori verificata  │
│  • Metadati di sicurezza inseriti           │
│  • Chiamate al runtime inserite             │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│       Backend e Code Generation             │
│  • Ottimizzazioni LLVM                      │
│  • Generazione codice macchina              │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│         Eseguibile Fil-C Sicuro             │
│  • Runtime Fil-C integrato                  │
│  • FUGC garbage collector                   │
│  • Sistema di capabilities attivo           │
└─────────────────────────────────────────────┘
```

### Principi di Design

**1. Strumentazione Completa**

Ogni accesso alla memoria nel programma è trasformato in una chiamata verificata:

```c
// Codice originale
int x = arr[i];

// Trasformato da Fil-C (concettualmente)
int x = filc_checked_array_access(arr_capability, i);
```

**2. Runtime Integrato**

Il runtime Fil-C fornisce:
- Sistema di gestione delle capabilities
- Garbage collector concorrente (FUGC)
- Allocatore di memoria sicuro
- Sistema di terminazione controllata per errori

**3. Librerie Standard Portate**

Fil-C include versioni sicure di:
- **musl libc**: Libreria C standard completa
- **LLVM libC++**: Libreria standard C++
- Librerie POSIX essenziali

---

## InvisiCaps: Il Modello delle Capabilities

### Concetto Fondamentale

Le **Invisible Capabilities (InvisiCaps)** sono il cuore dell'architettura di sicurezza di Fil-C. A differenza di altri sistemi di capabilities (come CHERI), le InvisiCaps sono "invisibili" al programma ma controllate dal runtime.

### Struttura di un InvisiCap

Ogni puntatore in Fil-C è associato a metadati invisibili che descrivono:

```
┌─────────────────────────────────────────────┐
│            Puntatore Visibile               │
│         (64-bit standard)                   │
│                                             │
│  Indirizzo: 0x7ffc1234abcd                 │
└─────────────────────────────────────────────┘
                    │
                    │ Associato a
                    ▼
┌─────────────────────────────────────────────┐
│      Metadati Capability (Invisibili)       │
├─────────────────────────────────────────────┤
│ • Base address: 0x7ffc12340000             │
│ • Size/Bounds: 1024 bytes                  │
│ • Type information: int*                    │
│ • Permissions: RW (Read/Write)             │
│ • Object lifetime: GC managed              │
│ • Allocation site: file.c:42               │
└─────────────────────────────────────────────┘
```

### Vantaggi delle InvisiCaps

**1. Compatibilità Binaria**

I puntatori rimangono a 64-bit, mantenendo la compatibilità con:
- Layout di memoria esistenti
- Interfacce di sistema operativo
- ABI (Application Binary Interface) standard
- Codice assembly esistente

**2. Verifica a Runtime**

Ogni operazione su puntatori è verificata:

```c
int *ptr = malloc(sizeof(int) * 10);
// Fil-C crea una capability: [base=ptr, size=40, type=int*]

ptr[5] = 42;   // ✓ OK: dentro i limiti
ptr[15] = 99;  // ✗ PANIC: out-of-bounds access!

free(ptr);     // Capability invalidata dal GC

ptr[5] = 10;   // ✗ PANIC: use-after-free!
```

**3. Type Safety**

Le capabilities includono informazioni sul tipo:

```c
struct Point {
    int x, y;
};

struct Point *p = malloc(sizeof(struct Point));
// Capability: [base=p, size=8, type=struct Point*]

int *bad_cast = (int*)p;
// Fil-C mantiene il tipo originale nella capability

bad_cast[10] = 0;  // ✗ PANIC: out-of-bounds!
// Il cast non può espandere i limiti
```


### Implementazione Tecnica

**Mapping Puntatore-Capability**

Fil-C usa una tabella hash ottimizzata per associare puntatori a capabilities:

```
Hash Table delle Capabilities
┌──────────┬──────────────────────────┐
│ Pointer  │ Capability Metadata      │
├──────────┼──────────────────────────┤
│ 0x1000   │ {base:0x1000, size:100}  │
│ 0x2000   │ {base:0x2000, size:50}   │
│ 0x3000   │ {base:0x3000, size:200}  │
└──────────┴──────────────────────────┘
```

**Operazioni Verificate**

```c
// Dereferenziazione
*ptr → filc_check_and_load(ptr_cap)

// Aritmetica dei puntatori
ptr + i → filc_offset_pointer(ptr_cap, i)

// Assegnamento
ptr = other → filc_copy_capability(other_cap)
```

---

## MonoCaps: Capabilities a 128-bit

### Estensione del Modello

Le **MonoCaps** sono capabilities estese a 128-bit che offrono garanzie ancora più forti rispetto alle InvisiCaps standard.

### Struttura di un MonoCap

```
┌────────────────────────────────────────────────────┐
│              MonoCap (128-bit)                     │
├────────────────────────────────────────────────────┤
│ Bit 0-63:   Indirizzo del puntatore (64-bit)      │
│ Bit 64-95:  Bounds/Size (32-bit)                  │
│ Bit 96-111: Type ID (16-bit)                      │
│ Bit 112-119: Permissions (8-bit)                  │
│ Bit 120-127: Flags e metadata (8-bit)             │
└────────────────────────────────────────────────────┘
```

### Confronto InvisiCaps vs MonoCaps

| Caratteristica | InvisiCaps | MonoCaps |
|----------------|------------|----------|
| Dimensione | 64-bit visibili + metadata | 128-bit totali |
| Compatibilità ABI | Completa | Limitata |
| Type checking | Runtime lookup | Embedded |
| Performance | Alta (dopo warm-up) | Più alta (info inline) |
| Uso di memoria | Basso | Doppio per puntatori |

---

## FUGC: Il Garbage Collector di Fil-C

### Introduzione a FUGC

**FUGC** (Fil's Unbelievable Garbage Collector) è un garbage collector concorrente, parallelo e non-mobile progettato specificamente per C/C++.

### Caratteristiche Principali

**1. Concurrent & Parallel**

- **Concurrent**: L'applicazione (mutator) continua l'esecuzione durante la garbage collection
- **Parallel**: Utilizza thread multipli per la collezione, sfruttando CPU multi-core
- **Non-moving**: Gli oggetti non vengono rilocati, mantenendo validi tutti i puntatori

```
Timeline della Garbage Collection
═════════════════════════════════════════

Mutator Thread:  ████████████████████████████
                 (continua sempre)

GC Thread 1:            ╔════╗      ╔════╗
                        ║ GC ║      ║ GC ║

GC Thread 2:            ╔════╗      ╔════╗
                        ║ GC ║      ║ GC ║

Pause Minime:           ↕ STW  ↕     ↕ STW ↕
                        (brevi)      (brevi)
```

### Algoritmo di FUGC

FUGC implementa una variante dell'**algoritmo di Dijkstra a grey-stack**:

**Fasi della Collezione:**

```
┌─────────────────────────────────────────────┐
│  1. INIZIALIZZAZIONE (Stop-The-World)       │
│  • Snapshot delle root                      │
│  • Marking iniziale                         │
│  • Durata: ~0.1-1ms                        │
└───────────────┬─────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────┐
│  2. CONCURRENT MARKING                      │
│  • GC threads marcano il grafo             │
│  • Mutator continua l'esecuzione           │
│  • Write barriers traccia modifiche        │
└───────────────┬─────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────┐
│  3. FINALIZZAZIONE (Stop-The-World)        │
│  • Riprocessa oggetti modificati           │
│  • Verifica completamento marking          │
│  • Durata: ~0.1-1ms                       │
└───────────────┬─────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────┐
│  4. CONCURRENT SWEEPING                     │
│  • Libera memoria non marcata              │
│  • Ritorna blocchi al free pool            │
│  • Mutator continua normalmente            │
└─────────────────────────────────────────────┘
```

### Tricolore Marking

FUGC usa lo schema tricolore standard:

- **Bianco**: Oggetti non visitati (candidati per la collezione)
- **Grigio**: Oggetti visitati ma non completamente processati
- **Nero**: Oggetti completamente processati (sicuramente vivi)

```c
// Pseudo-codice del marking
void mark_object(Object *obj) {
    if (is_white(obj)) {
        set_grey(obj);
        push_grey_stack(obj);
    }
}

void process_grey_stack() {
    while (!grey_stack_empty()) {
        Object *obj = pop_grey_stack();
        
        // Marca tutti i riferimenti
        for (Field *f : obj->fields) {
            if (is_pointer(f)) {
                Object *child = dereference(f);
                mark_object(child);
            }
        }
        
        set_black(obj);
    }
}
```

### Write Barriers

Per mantenere la correttezza durante l'esecuzione concorrente, FUGC usa **write barriers**:

```c
// Quando il mutator modifica un puntatore
void write_barrier(Object *obj, Field *field, Object *new_value) {
    // Snapshot-at-the-beginning barrier
    if (gc_in_progress && is_black(obj) && is_white(new_value)) {
        mark_object(new_value);
    }
    
    // Esegue la scrittura effettiva
    *field = new_value;
}
```

**Tipi di Write Barrier:**

1. **Snapshot-at-the-Beginning (SATB)**: Preserva lo stato iniziale del grafo
2. **Incremental Update**: Marca nuovi riferimenti creati durante la collezione

---

## Memory Safety: Teoria e Implementazione

### Definizione Formale di Memory Safety

Un programma è **memory-safe** se garantisce:

1. **Spatial Safety**: Ogni accesso alla memoria è dentro i limiti dell'oggetto allocato
2. **Temporal Safety**: Ogni accesso avviene mentre l'oggetto è ancora vivo
3. **Type Safety**: Ogni operazione rispetta il tipo dell'oggetto

### Violazioni Comuni in C/C++

#### 1. Spatial Safety Violations

**Buffer Overflow:**

```c
// UNSAFE C
void unsafe_example() {
    int buffer[10];
    buffer[15] = 42;  // Undefined Behavior!
}

// SAFE in Fil-C
void safe_example() {
    int buffer[10];
    buffer[15] = 42;  // PANIC: "index 15 out of bounds"
}
```

#### 2. Temporal Safety Violations

**Use-After-Free:**

```c
// UNSAFE C
int* create_dangling() {
    int *ptr = malloc(sizeof(int) * 10);
    free(ptr);
    return ptr;  // Dangling pointer!
}

// SAFE in Fil-C
int* safe_create() {
    int *ptr = malloc(sizeof(int) * 10);
    free(ptr);  // GC marca come invalidata
    return ptr;
}

void safe() {
    int *p = safe_create();
    *p = 42;  // PANIC: "use of freed memory"
}
```

#### 3. Type Safety Violations

**Type Confusion:**

```c
// UNSAFE C
struct TypeA {
    int x;
    void (*func)(void);
};

void type_confusion() {
    struct TypeA *a = malloc(sizeof(struct TypeA));
    struct TypeB *b = (struct TypeB *)a;
    // Il cast non cambia la capability in Fil-C!
}
```

---

## Estrazione e Gestione dei Puntatori

### Il Problema dell'Estrazione dei Puntatori

In C, i puntatori possono essere estratti, manipolati e ricostruiti in modi complessi:

```c
// Estrazione da struct
struct Data {
    int x;
    int y;
};

struct Data d = {10, 20};
int *py = &d.y;  // Estrae puntatore al campo y

// Aritmetica dei puntatori
int arr[10];
int *p1 = arr;
int *p2 = p1 + 5;  // Puntatore derivato

// Conversione a intero e back
uintptr_t addr = (uintptr_t)p2;
int *p3 = (int*)(addr + sizeof(int));
```

### Gestione delle Capabilities per Puntatori Estratti

#### 1. Puntatori a Campi di Struct

```c
struct Person {
    char name[50];
    int age;
    float salary;
};

struct Person p = {"Alice", 30, 50000.0};
// Capability: [base=&p, size=sizeof(Person), type=Person*]

int *age_ptr = &p.age;
// Fil-C crea una capability derivata:
// [base=&p, offset=50, size=sizeof(int), type=int*]
```

#### 2. Puntatori Derivati da Array

```c
int matrix[10][20];
// Capability: [base=matrix, size=10*20*sizeof(int)]

int *row5 = matrix[5];
// Derived capability: [base=matrix, offset=5*20*sizeof(int)]

int *element = &matrix[3][7];
// Derived capability con offset specifico
```

#### 3. Cast a Intero e Ricostruzione

```c
int *original = malloc(sizeof(int) * 10);
// Capability: cap_original = [base=original, size=40]

uintptr_t addr = (uintptr_t)original;
// addr è un intero, NO capability

int *reconstructed = (int*)addr;
// Fil-C cerca e recupera la capability!
```

### Tecniche di Tracking Avanzate

#### 1. Shadow Memory

Fil-C può usare shadow memory per tracking rapido:

```
Address Space          Shadow Space
┌──────────┐          ┌──────────┐
│ 0x1000   │────────▶ │ cap_info │
│ 0x1001   │          │          │
│ 0x1002   │          │          │
└──────────┘          └──────────┘
```

#### 2. Capability Derivation Rules

```c
// Regole per derivare capabilities
Capability derive(Capability parent, ptrdiff_t offset, size_t new_size) {
    // Verifica limiti
    if (offset < 0 || offset >= parent.size) {
        panic("Invalid offset");
    }
    
    return Capability {
        .base = parent.base,
        .offset = parent.offset + offset,
        .size = new_size,
        .type = derive_type(parent.type, offset)
    };
}
```

---

## Compatibilità e Performance

### Compatibilità

#### Codebases Supportate

Fil-C è stato testato con successo su:

**Interpreti e Runtime:**
- CPython (interprete Python)
- Lua interpreter
- Ruby interpreter

**Database:**
- SQLite
- Redis (parziale)

**Networking:**
- OpenSSH
- curl + libcurl

**Librerie:**
- zlib (compressione)
- libpng (immagini)
- ncurses (UI testuale)

**Utilità di sistema:**
- grep, sed, awk (GNU coreutils)
- bash (shell)

### Performance

#### Overhead Tipico

| Workload | Overhead | Note |
|----------|----------|------|
| Computazione intensiva | 5-20% | Pochi accessi memoria |
| Manipolazione stringhe | 30-80% | Molti bounds checks |
| Allocation-heavy | 40-120% | GC overhead |
| I/O intensivo | 10-30% | Bound checks su buffer |
| Media | ~50% | Varia molto per workload |

#### Confronto con Altri Sistemi

| Sistema | Overhead | Memory Safety | Compatibilità |
|---------|----------|---------------|---------------|
| Native C | 0% (baseline) | Nessuna | 100% |
| AddressSanitizer | 2x-5x | Parziale | Alta |
| Valgrind | 10x-50x | Alta | Media |
| Fil-C | 1.5x-3x | Completa | Alta |
| CHERI | 1.1x-1.5x | Completa | Richiede hardware |

#### Ottimizzazioni

**1. Bounds Check Elimination**

Il compilatore elimina checks ridondanti:

```c
for (int i = 0; i < 10; i++) {
    arr[i] = 0;  // Check solo prima del loop
}
```

**2. Capability Caching**

```c
// Prima accesso: lookup completo
int x = arr[i];

// Successivi accessi: usa cache
int y = arr[j];  // Stessa capability in cache
```

**3. Profiling e Diagnostics**

```bash
# Compilare con profiling
filcc -O2 -g -fprofile-generate program.c -o program

# Eseguire per raccogliere dati
./program

# Ricompilare con ottimizzazioni guidate
filcc -O2 -fprofile-use program.c -o program_optimized
```


---

## Ricerca Accademica e Paper di Riferimento

### Paper Fondamentali su Memory Safety

#### 1. **"Cyclone: A Safe Dialect of C"** (2002)
*Trevor Jim, Greg Morrisett, Dan Grossman, et al.*

- **Contributo**: Primo tentativo sistematico di creare un dialetto safe di C
- **Tecniche**: Region-based memory management, fat pointers, flow analysis
- **Limitazione**: Richiede modifiche significative al codice
- **Link**: [Cyclone Paper](https://www.cs.umd.edu/~mwh/papers/cyclone-safety.pdf)

#### 2. **"CCured: Type-Safe Retrofitting of Legacy Code"** (2002)
*George C. Necula, Scott McPeak, Westley Weimer*

- **Contributo**: Inferenza automatica di tipi safe per C esistente
- **Tecniche**: Static analysis + runtime checks selettivi
- **Relevanza per Fil-C**: Approccio ibrido statico/dinamico
- **Link**: [CCured at Berkeley](https://people.eecs.berkeley.edu/~necula/ccured.html)

#### 3. **"SoftBound: Highly Compatible and Complete Spatial Memory Safety for C"** (2009)
*Santosh Nagarakatte, Jianzhou Zhao, Milo Martin, Steve Zdancewic*

- **Contributo**: Spatial safety completa con overhead limitato
- **Tecniche**: Disjoint metadata storage (simile a Fil-C InvisiCaps)
- **Risultati**: ~67% overhead in media
- **Link**: [SoftBound Paper](https://repository.upenn.edu/cgi/viewcontent.cgi?article=1941&context=cis_papers)

#### 4. **"CETS: Compiler-Enforced Temporal Safety for C"** (2010)
*Santosh Nagarakatte, et al.*

- **Contributo**: Temporal safety (use-after-free prevention)
- **Tecniche**: Lock-and-key mechanism per object lifetime
- **Combinato con**: SoftBound per safety completa
- **Link**: [CETS Paper](https://www.cs.rutgers.edu/~santosh.nagarakatte/papers/cets-ismm10.pdf)

### Garbage Collection Research

#### 5. **"On-the-Fly Garbage Collection: An Exercise in Cooperation"** (1978)
*Edsger W. Dijkstra, Leslie Lamport, et al.*

- **Contributo**: Algoritmo tricolore concorrente (base per FUGC)
- **Tecniche**: Marking incrementale senza stop-the-world
- **Impatto**: Foundation per tutti i GC concorrenti moderni
- **Link**: [Dijkstra's Algorithm](https://lamport.azurewebsites.net/pubs/garbage.pdf)

#### 6. **"A Fully Concurrent Garbage Collector for Modern Processors"** (2000)
*David F. Bacon, et al.*

- **Contributo**: GC completamente concorrente senza pause
- **Tecniche**: Write barriers ottimizzate, parallel marking
- **Relevanza**: Simile all'approccio di FUGC
- **Link**: [Washington University Research](https://profiles.wustl.edu/en/publications/a-fully-concurrent-garbage-collector)

#### 7. **"Write Barrier Elision for Concurrent Garbage Collectors"** (2002)
*Martin Vechev, David F. Bacon, et al.*

- **Contributo**: Eliminazione write barriers ridondanti
- **Tecniche**: Static analysis per identificare writes sicuri
- **Impatto su Performance**: Riduzione overhead del 20-40%
- **Link**: [ETH Zurich Paper](https://files.sri.inf.ethz.ch/website/papers/Vechev04Barriers.pdf)

#### 8. **"Quantifying the Performance of Garbage Collection vs. Explicit Memory Management"** (2005)
*Matthew Hertz, Emery D. Berger*

- **Contributo**: Confronto rigoroso GC vs manual management
- **Risultati**: GC può essere competitivo o superiore con heap sufficiente
- **Rilevanza**: Giustifica l'approccio di Fil-C
- **Link**: [UMass Paper](https://people.cs.umass.edu/~emery/pubs/gcvsmalloc.pdf)

### Capability-Based Security

#### 9. **"CHERI: A Hybrid Capability-System Architecture"** (2015)
*Robert N. M. Watson, et al.*

- **Contributo**: Architettura hardware con capabilities
- **Differenza da Fil-C**: Hardware vs software, wide pointers vs InvisiCaps
- **Progetto**: University of Cambridge
- **Link**: [CHERI Project](https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/)

#### 10. **"Low-Fat Pointers: Compact Encoding and Efficient Gate-Level Implementation"** (2013)
*Gregory J. Duck, Roland H. C. Yap*

- **Contributo**: Encoding compatto per bounds metadata
- **Tecniche**: Sfrutta bit inutilizzati nei puntatori
- **Confronto**: Alternativa a fat pointers e separate metadata
- **Link**: [Low-Fat Pointers Paper](https://www.comp.nus.edu.sg/~gregory/papers/lowfat_ccs13.pdf)

### Fil-C Specific Research

#### 11. **"Fil-C: Memory Safety with Fanatical C/C++ Compatibility"** (2024)
*Filip Pizlo*

- **Venue**: SPLASH 2024 (REBASE track)
- **Contributo**: Presentazione completa di Fil-C
- **Novità**: InvisiCaps, FUGC, zero-escape guarantee
- **Link**: [SPLASH 2024 Talk](https://2024.splashcon.org/details/splash-2024-rebase/3/Fil-C-memory-safety-with-fanatical-C-C-compatibility)

#### 12. **"The Fil-C Manifesto"** (2024)
*Filip Pizlo*

- **Tipo**: Technical manifesto e design document
- **Contenuto**: Philosophy, architecture, implementation details
- **Repository**: GitHub documentation
- **Link**: [Fil-C Manifesto](https://github.com/pizlonator/fil-c/blob/deluge/Manifesto.md)

### Related Topics e Survey Papers

#### 13. **"SoK: Eternal War in Memory"** (2013)
*László Szekeres, et al.*

- **Tipo**: Systematization of Knowledge
- **Ambito**: Comprehensive survey of memory safety attacks and defenses
- **Utilità**: Contestualizza Fil-C nel panorama delle soluzioni
- **Link**: [IEEE S&P Paper](https://www.cs.berkeley.edu/~dawnsong/papers/Oakland13-SoK-CR.pdf)

#### 14. **"Memory Safety for Low-Level Software/Hardware Interactions"** (2009)
*John Criswell, Nicolas Geoffray, Vikram Adve*

- **Contributo**: Safety per interazioni hardware/software
- **Tecniche**: Simili a quelle necessarie per Fil-C MMIO
- **Link**: [USENIX Paper](https://www.usenix.org/legacy/event/usenix09/tech/full_papers/criswell/criswell.pdf)

#### 15. **"Historical Evolution and Future Trends in Garbage Collection"** (2020)
*Survey paper on GC techniques*

- **Tipo**: Survey completo delle tecniche GC
- **Contenuto**: Da mark-sweep a GC concorrenti moderni
- **Utilità**: Background teorico per comprendere FUGC
- **Link**: [Survey Paper](https://www.ijirmps.org/papers/2020/2/232039.pdf)

### Letture Consigliate per Approfondimento

**Per iniziare:**
1. The Fil-C Manifesto (start here!)
2. "SoK: Eternal War in Memory" (context)
3. "Cyclone: A Safe Dialect of C" (historical)

**Per memory safety:**
4. SoftBound + CETS papers (spatial + temporal)
5. CCured (type system approach)
6. Low-Fat Pointers (encoding alternatives)

**Per garbage collection:**
7. Dijkstra's on-the-fly GC (foundations)
8. "Fully Concurrent GC" paper
9. GC vs manual management comparison

**Per capabilities:**
10. CHERI architecture
11. Fil-C InvisiCaps documentation
12. MonoCaps technical details

---

## Link Utili e Risorse

### Documentazione Ufficiale Fil-C

#### Sito Web Principale
- **Homepage**: [https://fil-c.org/](https://fil-c.org/)
  - Introduzione al progetto
  - Getting started guide
  - News e updates

#### Documentazione Tecnica
- **Documentation Hub**: [https://fil-c.org/documentation](https://fil-c.org/documentation)
  - Reference completo
  - API documentation
  - Esempi di utilizzo

- **InvisiCaps**: [https://fil-c.org/invisicaps](https://fil-c.org/invisicaps)
  - Modello delle capabilities
  - Teoria e implementazione

- **InvisiCaps By Example**: [https://fil-c.org/invisicaps_by_example](https://fil-c.org/invisicaps_by_example)
  - Tutorial pratico
  - Esempi annotati
  - Pattern comuni

- **FUGC Documentation**: [https://fil-c.org/fugc](https://fil-c.org/fugc)
  - Architettura del GC
  - Algoritmi e ottimizzazioni
  - Tuning e diagnostics

### Repository e Codice Sorgente

#### GitHub Repository
- **Main Repository**: [https://github.com/pizlonator/fil-c](https://github.com/pizlonator/fil-c)
  - Codice sorgente completo
  - Build instructions
  - Issue tracker

- **Manifesto**: [https://github.com/pizlonator/fil-c/blob/deluge/Manifesto.md](https://github.com/pizlonator/fil-c/blob/deluge/Manifesto.md)
  - Design philosophy
  - Technical decisions
  - Roadmap

- **Examples**: [https://github.com/pizlonator/fil-c/tree/deluge/examples](https://github.com/pizlonator/fil-c/tree/deluge/examples)
  - Programmi di esempio
  - Casi d'uso reali
  - Test suite

### Community e Discussioni

#### Forum e Chat
- **GitHub Discussions**: [https://github.com/pizlonator/fil-c/discussions](https://github.com/pizlonator/fil-c/discussions)
  - Q&A
  - Feature requests
  - Discussioni tecniche

#### Discussioni Storiche
- **Hacker News Thread**: [https://news.ycombinator.com/item?id=39449500](https://news.ycombinator.com/item?id=39449500)
  - Discussione iniziale sul manifesto
  - Feedback della community
  - Comparazioni con alternative

- **Lobsters Discussion**: [https://lobste.rs/s/1utowa/fil_c_manifesto_garbage_memory_safety_out](https://lobste.rs/s/1utowa/fil_c_manifesto_garbage_memory_safety_out)
  - Analisi tecnica
  - Critiche costruttive

### Video e Presentazioni

#### Conference Talks
- **SPLASH 2024 Presentation**: [https://2024.splashcon.org/details/splash-2024-rebase/3/Fil-C-memory-safety-with-fanatical-C-C-compatibility](https://2024.splashcon.org/details/splash-2024-rebase/3/Fil-C-memory-safety-with-fanatical-C-C-compatibility)
  - Presentazione ufficiale
  - Slides e video

- **YouTube Introduction**: [https://www.youtube.com/watch?v=JTxIWXWTAXM](https://www.youtube.com/watch?v=JTxIWXWTAXM)
  - Introduzione video
  - Demo pratiche

### Articoli e Analisi

#### Technical Analysis
- **GizVault - What is Fil-C**: [https://www.gizvault.com/archives/what-is-fil-c-and-how-to-use-it](https://www.gizvault.com/archives/what-is-fil-c-and-how-to-use-it)
  - Guida pratica
  - Tutorial setup

- **GizVault - FUGC Deep Dive**: [https://www.gizvault.com/archives/understand-the-gc-in-fil-c](https://www.gizvault.com/archives/understand-the-gc-in-fil-c)
  - Analisi dettagliata del GC
  - Internals

- **Altus Intel Coverage**: [https://www.altusintel.com/public-yycr12/](https://www.altusintel.com/public-yycr12/)
  - News e analisi
  - Confronti industriali

### Progetti Correlati e Alternative

#### CHERI (Hardware Capabilities)
- **CHERI Project**: [https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/](https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/)
- **CHERI ISA**: [https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-951.pdf](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-951.pdf)

#### AddressSanitizer
- **ASan Documentation**: [https://github.com/google/sanitizers/wiki/AddressSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer)
- **Clang ASan**: [https://clang.llvm.org/docs/AddressSanitizer.html](https://clang.llvm.org/docs/AddressSanitizer.html)

#### Rust (Language-Level Safety)
- **Rust Book**: [https://doc.rust-lang.org/book/](https://doc.rust-lang.org/book/)
- **Rust Memory Safety**: [https://doc.rust-lang.org/nomicon/](https://doc.rust-lang.org/nomicon/)

### Academic Resources

#### Memory Safety Papers
- **SoftBound**: [https://repository.upenn.edu/cis_papers/](https://repository.upenn.edu/cis_papers/)
- **CETS**: [https://www.cs.rutgers.edu/~santosh.nagarakatte/papers/](https://www.cs.rutgers.edu/~santosh.nagarakatte/papers/)

#### GC Research
- **GC Papers**: [http://www.iecc.com/gclist/](http://www.iecc.com/gclist/)
- **GC Bibliography**: [https://www.ics.uci.edu/~franz/research/garbagecollection/](https://www.ics.uci.edu/~franz/research/garbagecollection/)

### Tools e Utilities

#### LLVM Resources
- **LLVM Project**: [https://llvm.org/](https://llvm.org/)
- **LLVM IR**: [https://llvm.org/docs/LangRef.html](https://llvm.org/docs/LangRef.html)
- **Clang**: [https://clang.llvm.org/](https://clang.llvm.org/)

#### Debugging e Profiling
- **Valgrind**: [https://valgrind.org/](https://valgrind.org/)
- **GDB**: [https://www.gnu.org/software/gdb/](https://www.gnu.org/software/gdb/)
- **perf**: [https://perf.wiki.kernel.org/](https://perf.wiki.kernel.org/)

### Tutorial e Guide

#### Getting Started
1. **Clone the repository**:
   ```bash
   git clone https://github.com/pizlonator/fil-c.git
   cd fil-c
   ```

2. **Build Fil-C**:
   ```bash
   ./build.sh
   ```

3. **Compile a program**:
   ```bash
   filcc -o myprogram myprogram.c
   ./myprogram
   ```

#### Example Programs

**Hello World con Fil-C:**
```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    printf("Hello from Fil-C!\n");
    
    // Allocazione sicura
    int *arr = malloc(sizeof(int) * 10);
    
    // Accesso verificato
    for (int i = 0; i < 10; i++) {
        arr[i] = i * i;
    }
    
    // GC si occupa della deallocazione
    return 0;
}
```

**Compilazione:**
```bash
filcc -O2 -g hello.c -o hello
./hello
```

### Riferimenti Rapidi

#### Comandi Principali
```bash
# Compilare con Fil-C
filcc [opzioni] file.c -o output

# Opzioni comuni
-O2                    # Ottimizzazioni
-g                     # Debug symbols
-v                     # Verbose
-filc-verbose=N        # Verbosity level (0-3)
-filc-gc-stats         # Abilita statistiche GC

# Environment variables
export FILC_GC_VERBOSE=3     # GC diagnostics
export FILC_HEAP_SIZE=512M   # Heap size
```

#### Link Rapidi per Riferimento

| Risorsa | URL |
|---------|-----|
| Homepage | https://fil-c.org/ |
| GitHub | https://github.com/pizlonator/fil-c |
| Docs | https://fil-c.org/documentation |
| InvisiCaps | https://fil-c.org/invisicaps |
| FUGC | https://fil-c.org/fugc |
| SPLASH Talk | https://2024.splashcon.org/details/splash-2024-rebase/3/... |

---

## Conclusioni

Fil-C rappresenta un avanzamento significativo nella ricerca della memory safety per C e C++. Combinando:

- **InvisiCaps**: Sistema di capabilities invisibili per compatibilità perfetta
- **FUGC**: Garbage collector concorrente per temporal safety
- **Strumentazione completa**: Verifica di ogni accesso alla memoria
- **Zero escapes**: Nessuna via d'uscita dalle garanzie di sicurezza

Fil-C dimostra che è possibile ottenere memory safety completa mantenendo alta compatibilità con codebases esistenti. Sebbene il costo in performance sia non trascurabile (1.5x-3x overhead), rappresenta un compromesso accettabile considerando l'eliminazione completa di una classe di vulnerabilità che causa la maggioranza degli exploit moderni.

La ricerca continua a evolversi, con futuri miglioramenti previsti per:
- Generational garbage collection per ridurre pause
- Ottimizzazioni guidate da profile
- Support hardware-accelerated (es. integrazione con CHERI)
- Riduzione ulteriore dell'overhead

Fil-C non è solo uno strumento pratico, ma anche una dimostrazione di concetti teorici avanzati applicati al mondo reale, rendendo il C sicuro senza sacrificarne la potenza e flessibilità.

### Implicazioni per il Futuro

**Per i Programmatori:**
- Possibilità di scrivere codice C/C++ memory-safe senza cambiare linguaggio
- Migrazione graduale di codebases esistenti
- Eliminazione di intere classi di bug critici

**Per la Sicurezza:**
- Riduzione drastica delle vulnerabilità di memory safety
- Protezione automatica contro exploit comuni
- Base più solida per software critico

**Per la Ricerca:**
- Dimostra la fattibilità di safety completa per C/C++
- Fornisce insights per futuri miglioramenti
- Ponte tra ricerca accademica e applicazioni pratiche

---

[← Torna al Modulo Avanzato](README.md) | [Torna al Corso](../README.md)
