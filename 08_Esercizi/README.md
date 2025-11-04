# Modulo 08 - Esercizi e Sfide

## Esercizi per Principianti

### 1. Calcolatrice Base
Scrivi un programma che funziona come una calcolatrice con le quattro operazioni base.

**Requisiti:**
- Input: due numeri e un operatore
- Operazioni: +, -, *, /
- Gestione divisione per zero
- Loop per permettere multiple operazioni

**Esempio di output:**
```
Inserisci primo numero: 10
Inserisci operatore (+, -, *, /): *
Inserisci secondo numero: 5
Risultato: 50
Continuare? (s/n):
```

### 2. Trova il Massimo
Scrivi una funzione che trova il numero massimo in un array.

```c
int trova_massimo(int arr[], int size);
```

**Test case:**
```c
int arr[] = {3, 7, 2, 9, 1, 5};
// trova_massimo(arr, 6) dovrebbe ritornare 9
```

### 3. Inverti Stringa
Scrivi una funzione che inverte una stringa in-place.

```c
void inverti_stringa(char *str);
```

**Test case:**
```c
char str[] = "hello";
inverti_stringa(str);
// str dovrebbe essere "olleh"
```

### 4. Conta Vocali e Consonanti
Scrivi un programma che conta vocali e consonanti in una stringa.

**Esempio:**
```
Input: "Hello World"
Vocali: 3
Consonanti: 7
```

### 5. Numeri Primi
Scrivi una funzione che verifica se un numero è primo.

```c
bool is_primo(int n);
```

Poi scrivi un programma che stampa tutti i numeri primi da 1 a 100.

### 6. Fibonacci
Implementa la sequenza di Fibonacci in tre modi:
1. Ricorsivo
2. Iterativo
3. Con memoization

### 7. Palindromo
Scrivi una funzione che verifica se una stringa è un palindromo.

```c
bool is_palindromo(const char *str);
```

**Test cases:**
```c
is_palindromo("radar") // true
is_palindromo("hello") // false
is_palindromo("A man a plan a canal Panama") // true (ignora spazi e case)
```

### 8. Ordina Array
Implementa bubble sort per ordinare un array di interi.

```c
void bubble_sort(int arr[], int size);
```

### 9. Matrice Trasposta
Scrivi una funzione che calcola la trasposta di una matrice.

```c
void trasposta(int matrice[3][3], int risultato[3][3]);
```

### 10. Conta Parole
Scrivi un programma che conta il numero di parole in una stringa.

**Esempio:**
```
Input: "Hello world from C"
Output: 4 parole
```

## Esercizi Intermedi

### 11. Gestione Studenti
Crea un sistema per gestire studenti con:
- Struttura `Studente` (nome, matricola, voti[])
- Funzione per calcolare media voti
- Funzione per trovare studente con media più alta
- Salvataggio e caricamento da file

### 12. Binary Search Tree
Implementa un Binary Search Tree con operazioni:
- Inserimento
- Ricerca
- Cancellazione
- Attraversamento (in-order, pre-order, post-order)
- Trova minimo/massimo

### 13. Merge Sort
Implementa l'algoritmo merge sort.

```c
void merge_sort(int arr[], int left, int right);
```

### 14. Dynamic Array
Implementa un array dinamico che cresce automaticamente:

```c
typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} DynamicArray;

void darr_init(DynamicArray *arr);
void darr_push(DynamicArray *arr, int value);
int darr_get(const DynamicArray *arr, size_t index);
void darr_free(DynamicArray *arr);
```

### 15. String Builder
Implementa un string builder efficiente:

```c
typedef struct {
    char *buffer;
    size_t length;
    size_t capacity;
} StringBuilder;

void sb_init(StringBuilder *sb);
void sb_append(StringBuilder *sb, const char *str);
char* sb_to_string(const StringBuilder *sb);
void sb_free(StringBuilder *sb);
```

### 16. CSV Parser
Scrivi un parser per file CSV:
- Legge file CSV
- Gestisce campi tra virgolette
- Gestisce escape characters
- Ritorna array di record

### 17. Expression Evaluator
Estendi il parser del Lab 5 per supportare:
- Potenze (^)
- Funzioni matematiche (sin, cos, sqrt)
- Variabili
- Precedenza operatori corretta

### 18. Memory Allocator Personalizzato
Implementa un semplice memory allocator:

```c
void* my_malloc(size_t size);
void my_free(void *ptr);
void* my_realloc(void *ptr, size_t size);
```

### 19. LRU Cache
Implementa una LRU (Least Recently Used) cache:

```c
typedef struct LRUCache LRUCache;

LRUCache* lru_create(int capacity);
int lru_get(LRUCache *cache, int key);
void lru_put(LRUCache *cache, int key, int value);
void lru_destroy(LRUCache *cache);
```

### 20. Thread Pool
Implementa un thread pool per task paralleli:

```c
typedef struct ThreadPool ThreadPool;
typedef void (*task_function)(void *arg);

ThreadPool* pool_create(int num_threads);
void pool_submit(ThreadPool *pool, task_function func, void *arg);
void pool_wait(ThreadPool *pool);
void pool_destroy(ThreadPool *pool);
```

## Esercizi Avanzati

### 21. Garbage Collector
Implementa un semplice garbage collector mark-and-sweep:
- Track allocazioni
- Mark reachable objects
- Sweep unreachable objects
- API per allocazione e roots

### 22. JSON Parser
Scrivi un parser JSON completo:
- Supporta tutti i tipi JSON (object, array, string, number, boolean, null)
- Parse to internal representation
- Serialize back to JSON
- Error handling

### 23. HTTP Server
Implementa un HTTP server basilare:
- Gestisce richieste GET
- Serve file statici
- Supporta multiple connessioni
- Implementa basic routing

### 24. Virtual Machine
Crea una VM con bytecode interpreter:
- Set di istruzioni base (push, pop, add, sub, etc.)
- Stack-based architecture
- Chiamate di funzione
- Compiler da AST a bytecode

### 25. Database Engine
Implementa un semplice database engine:
- B-tree index
- Query parser (SELECT, INSERT, UPDATE, DELETE)
- Transactions (begin, commit, rollback)
- Persistenza su disco

### 26. Regex Engine
Implementa un motore di espressioni regolari:
- Pattern matching
- Operatori: *, +, ?, |, ()
- Character classes
- NFA o DFA based

### 27. Custom malloc con Buddy System
Implementa malloc usando il buddy system:
- Allocazione in potenze di 2
- Splitting e merging di blocchi
- Minimizza frammentazione
- Free list management

### 28. Lock-Free Data Structures
Implementa strutture dati lock-free:
- Lock-free queue
- Lock-free stack
- Usa atomic operations
- Compare-and-swap

### 29. Memory Profiler
Crea un memory profiler:
- Intercetta malloc/free
- Track allocazioni
- Detect memory leaks
- Genera report

### 30. JIT Compiler Basics
Implementa un semplice JIT compiler:
- Genera codice macchina a runtime
- Ottimizzazioni base
- Esegui codice generato
- Gestisci memoria per codice

## Progetti Finali

### Progetto 1: Sistema di File System
Implementa un file system simulato:

**Features:**
- Creazione/cancellazione file e directory
- Lettura/scrittura file
- Navigazione directory
- Permessi base
- Persistenza su file singolo
- Command-line interface

**Strutture dati suggerite:**
- Inode table
- Data blocks
- Directory entries
- Free block bitmap

### Progetto 2: Mini Shell
Crea una shell Unix-like:

**Features:**
- Esecuzione comandi
- Pipe tra comandi
- Redirezione I/O
- Background processes
- Built-in commands (cd, exit, etc.)
- Gestione segnali
- History dei comandi

### Progetto 3: Network Chat Application
Implementa un'applicazione di chat:

**Features:**
- Server multi-client
- Stanze chat multiple
- Messaggi privati
- Nickname users
- Command interface (/join, /leave, /msg)
- Persistenza messaggi
- Protocol personalizzato

### Progetto 4: Game Engine 2D
Crea un semplice game engine:

**Features:**
- Sistema di rendering
- Entity-Component System
- Collision detection
- Input handling
- Asset management
- Game loop
- Sample game (es. Snake o Breakout)

### Progetto 5: Compiler per Linguaggio Custom
Scrivi un compiler completo:

**Fasi:**
1. Lexer (tokenization)
2. Parser (AST generation)
3. Semantic analysis
4. IR generation
5. Optimization
6. Code generation (assembly o bytecode)
7. Linker/loader

**Linguaggio suggerito:**
```
# Esempio linguaggio semplice
fun fibonacci(n: int) -> int {
    if n <= 1 {
        return n;
    }
    return fibonacci(n-1) + fibonacci(n-2);
}

let result = fibonacci(10);
print(result);
```

## Sfide di Performance

### Challenge 1: Sorting Challenge
Implementa e confronta:
1. Bubble Sort
2. Insertion Sort
3. Quick Sort
4. Merge Sort
5. Heap Sort
6. Radix Sort

Benchmark su dataset di diverse dimensioni e caratteristiche.

### Challenge 2: String Search
Implementa algoritmi di string searching:
1. Naive search
2. KMP (Knuth-Morris-Pratt)
3. Boyer-Moore
4. Rabin-Karp

Confronta performance su diversi pattern e testi.

### Challenge 3: Matrix Operations
Ottimizza operazioni su matrici:
1. Moltiplicazione matrici naive
2. Moltiplicazione con tiling (cache-friendly)
3. Moltiplicazione con SIMD
4. Strassen algorithm

### Challenge 4: Parallel Sort
Implementa merge sort parallelo con pthreads:
- Dividi array tra threads
- Merge risultati
- Confronta con versione sequenziale
- Misura speedup

## Suggerimenti per gli Esercizi

### Per Principianti:
1. Inizia con esempi semplici
2. Testa con casi limite
3. Aggiungi error handling gradualmente
4. Commenta il codice mentre lo scrivi

### Per Intermedi:
1. Pianifica strutture dati prima di codificare
2. Scrivi test cases
3. Considera complessità temporale e spaziale
4. Usa valgrind per memory leaks

### Per Avanzati:
1. Design prima di implementare
2. Documenta API e invarianti
3. Scrivi test completi
4. Profile e ottimizza
5. Considera thread safety

## Risorse per Praticare

### Online Judges:
- **LeetCode** - Algoritmi e strutture dati
- **HackerRank** - Problemi C specifici
- **CodeWars** - Sfide di programmazione
- **Project Euler** - Problemi matematici/algoritmici

### Best Practices:
1. Leggi il problema completamente
2. Pensa alla soluzione prima di codificare
3. Inizia con soluzione semplice
4. Testa con esempi
5. Ottimizza se necessario
6. Rivedi il codice

### Debugging Tips:
1. Usa `printf` per debug temporaneo
2. Impara GDB per debugging avanzato
3. Usa valgrind per memory issues
4. Usa assert per verificare assunzioni
5. Compila con `-Wall -Wextra -Werror`

## Checklist per Ogni Esercizio

- [ ] Problema compreso chiaramente
- [ ] Test cases identificati
- [ ] Strutture dati progettate
- [ ] Codice implementato
- [ ] Compila senza warning
- [ ] Test passati
- [ ] Memory leaks verificati
- [ ] Edge cases gestiti
- [ ] Codice documentato
- [ ] Performance accettabile

---

[← Laboratori](../07_Laboratori/README.md) | [Torna al Sommario](../README.md)
