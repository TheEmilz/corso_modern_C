# Modulo 05 - Assembly e Programmazione Low-Level

## Relazione tra C e Assembly

### Dal C all'Assembly

Quando compili un programma C, il compilatore lo traduce in linguaggio assembly specifico per l'architettura target (x86, ARM, etc.).

```c
/* simple.c */
int add(int a, int b) {
    return a + b;
}
```

Per vedere l'assembly generato:
```bash
# Genera assembly (x86-64)
gcc -S -O0 simple.c -o simple.s

# Oppure con ottimizzazioni
gcc -S -O2 simple.c -o simple_opt.s

# Disassembla un binario
objdump -d program
```

Assembly x86-64 generato (AT&T syntax):
```asm
add:
    push   %rbp
    mov    %rsp,%rbp
    mov    %edi,-0x4(%rbp)    # primo parametro (a)
    mov    %esi,-0x8(%rbp)    # secondo parametro (b)
    mov    -0x4(%rbp),%edx
    mov    -0x8(%rbp),%eax
    add    %edx,%eax          # somma
    pop    %rbp
    ret
```

### Architettura del Processore

#### Registri x86-64

**Registri General Purpose (64-bit):**
- `RAX`, `RBX`, `RCX`, `RDX` - Uso generale
- `RSI`, `RDI` - Source/Destination Index
- `RSP` - Stack Pointer
- `RBP` - Base Pointer
- `R8`-`R15` - Registri aggiuntivi

**Versioni più piccole:**
- 32-bit: `EAX`, `EBX`, etc.
- 16-bit: `AX`, `BX`, etc.
- 8-bit: `AL`, `AH`, `BL`, `BH`, etc.

#### Calling Convention (System V AMD64 ABI - Linux/Mac)

**Passaggio parametri:**
1. Primo parametro: `RDI`
2. Secondo parametro: `RSI`
3. Terzo parametro: `RDX`
4. Quarto parametro: `RCX`
5. Quinto parametro: `R8`
6. Sesto parametro: `R9`
7. Altri: Stack

**Valore di ritorno:** `RAX`

```c
#include <stdio.h>

long example_function(long a, long b, long c, long d, long e, long f, long g)
{
    return a + b + c + d + e + f + g;
}

int main(void)
{
    long result = example_function(1, 2, 3, 4, 5, 6, 7);
    printf("Result: %ld\n", result);
    return 0;
}
```

Assembly generato mostra:
- Parametri 1-6 in registri
- Parametro 7 sullo stack

### Stack e Stack Frame

```c
#include <stdio.h>

void level3(void) {
    printf("Level 3\n");
}

void level2(void) {
    level3();
}

void level1(void) {
    level2();
}

int main(void) {
    level1();
    return 0;
}
```

**Stack durante l'esecuzione:**
```
[High Address]
+------------------+
|   main's frame   |
+------------------+
|  level1's frame  |
+------------------+
|  level2's frame  |
+------------------+
|  level3's frame  | <- RSP (Stack Pointer)
+------------------+
[Low Address]
```

Ogni frame contiene:
- Return address
- Variabili locali
- Saved registers
- Parametri (se necessario)

## Inline Assembly

### Sintassi GCC (AT&T)

```c
#include <stdio.h>

int main(void)
{
    int a = 10, b = 20, result;
    
    /* Basic inline assembly */
    __asm__ (
        "movl %1, %%eax\n\t"    // Move b to EAX
        "addl %2, %%eax\n\t"    // Add a to EAX
        "movl %%eax, %0\n\t"    // Move EAX to result
        : "=r" (result)          // Output operands
        : "r" (b), "r" (a)       // Input operands
        : "%eax"                 // Clobbered registers
    );
    
    printf("Result: %d\n", result);
    return 0;
}
```

### Extended ASM - Operazioni Atomiche

```c
#include <stdio.h>
#include <stdint.h>

/* Atomic increment usando LOCK prefix */
static inline void atomic_increment(int *ptr)
{
    __asm__ __volatile__ (
        "lock incl %0"
        : "+m" (*ptr)
        :
        : "cc"
    );
}

/* Atomic compare-and-swap */
static inline int atomic_cas(int *ptr, int old_val, int new_val)
{
    int result;
    __asm__ __volatile__ (
        "lock cmpxchgl %2, %1"
        : "=a" (result), "+m" (*ptr)
        : "r" (new_val), "0" (old_val)
        : "cc"
    );
    return result;
}

/* Read Time-Stamp Counter */
static inline uint64_t rdtsc(void)
{
    uint32_t lo, hi;
    __asm__ __volatile__ (
        "rdtsc"
        : "=a" (lo), "=d" (hi)
    );
    return ((uint64_t)hi << 32) | lo;
}

int main(void)
{
    int counter = 0;
    
    printf("Counter: %d\n", counter);
    atomic_increment(&counter);
    printf("After atomic increment: %d\n", counter);
    
    int old_val = atomic_cas(&counter, 1, 100);
    printf("CAS returned: %d, counter is now: %d\n", old_val, counter);
    
    uint64_t cycles1 = rdtsc();
    for (volatile int i = 0; i < 1000000; i++);
    uint64_t cycles2 = rdtsc();
    printf("CPU cycles: %lu\n", cycles2 - cycles1);
    
    return 0;
}
```

### SIMD Instructions

```c
#include <stdio.h>
#include <stdint.h>

/* Somma di 4 float usando SSE */
void add_four_floats_sse(float *result, const float *a, const float *b)
{
    __asm__ (
        "movups (%1), %%xmm0\n\t"   // Carica 4 float da a
        "movups (%2), %%xmm1\n\t"   // Carica 4 float da b
        "addps %%xmm1, %%xmm0\n\t"  // Somma vettoriale
        "movups %%xmm0, (%0)\n\t"   // Salva risultato
        :
        : "r" (result), "r" (a), "r" (b)
        : "%xmm0", "%xmm1"
    );
}

int main(void)
{
    float a[4] = {1.0f, 2.0f, 3.0f, 4.0f};
    float b[4] = {5.0f, 6.0f, 7.0f, 8.0f};
    float result[4];
    
    add_four_floats_sse(result, a, b);
    
    printf("SIMD Addition:\n");
    for (int i = 0; i < 4; i++) {
        printf("%.1f + %.1f = %.1f\n", a[i], b[i], result[i]);
    }
    
    return 0;
}
```

## Chiamate di Sistema (System Calls)

### System Call in Linux

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/syscall.h>

/* System call usando wrapper libc */
void example_libc(void)
{
    write(STDOUT_FILENO, "Hello from libc\n", 16);
}

/* System call diretta usando assembly */
void example_syscall_asm(void)
{
    const char *msg = "Hello from syscall\n";
    long len = 19;
    
    __asm__ __volatile__ (
        "mov $1, %%rax\n\t"      // syscall number (sys_write = 1)
        "mov $1, %%rdi\n\t"      // file descriptor (stdout = 1)
        "mov %0, %%rsi\n\t"      // buffer
        "mov %1, %%rdx\n\t"      // count
        "syscall\n\t"
        :
        : "r" (msg), "r" (len)
        : "%rax", "%rdi", "%rsi", "%rdx"
    );
}

/* System call usando syscall() */
void example_syscall_wrapper(void)
{
    const char *msg = "Hello from syscall wrapper\n";
    syscall(SYS_write, STDOUT_FILENO, msg, 27);
}

/* Esempio: getpid con syscall diretta */
long my_getpid(void)
{
    long pid;
    __asm__ __volatile__ (
        "mov $39, %%rax\n\t"     // sys_getpid = 39
        "syscall\n\t"
        "mov %%rax, %0\n\t"
        : "=r" (pid)
        :
        : "%rax"
    );
    return pid;
}

int main(void)
{
    example_libc();
    example_syscall_asm();
    example_syscall_wrapper();
    
    printf("PID (libc): %d\n", getpid());
    printf("PID (syscall): %ld\n", my_getpid());
    
    return 0;
}
```

### Tabella System Call Comuni (Linux x86-64)

| Numero | Nome | Descrizione |
|--------|------|-------------|
| 0 | sys_read | Leggi da file descriptor |
| 1 | sys_write | Scrivi su file descriptor |
| 2 | sys_open | Apri file |
| 3 | sys_close | Chiudi file descriptor |
| 9 | sys_mmap | Map memoria |
| 12 | sys_brk | Cambia data segment |
| 39 | sys_getpid | Get process ID |
| 57 | sys_fork | Crea processo figlio |
| 60 | sys_exit | Termina processo |

## Memory Layout di un Processo

```c
#include <stdio.h>
#include <stdlib.h>

/* Variabili globali inizializzate - .data segment */
int global_initialized = 42;

/* Variabili globali non inizializzate - .bss segment */
int global_uninitialized;

/* Variabili globali costanti - .rodata segment */
const char *const_string = "Hello, World!";

void show_memory_layout(void)
{
    /* Variabili locali - stack */
    int stack_var = 100;
    
    /* Memoria dinamica - heap */
    int *heap_var = malloc(sizeof(int));
    *heap_var = 200;
    
    /* Mostra indirizzi */
    printf("=== Memory Layout ===\n\n");
    
    printf("Text Segment (Code):\n");
    printf("  Function: %p\n\n", (void*)show_memory_layout);
    
    printf("Data Segment (.data):\n");
    printf("  Global initialized: %p\n\n", (void*)&global_initialized);
    
    printf("BSS Segment (.bss):\n");
    printf("  Global uninitialized: %p\n\n", (void*)&global_uninitialized);
    
    printf("Read-only Data (.rodata):\n");
    printf("  Const string: %p\n\n", (void*)const_string);
    
    printf("Heap:\n");
    printf("  Dynamic variable: %p\n\n", (void*)heap_var);
    
    printf("Stack:\n");
    printf("  Local variable: %p\n\n", (void*)&stack_var);
    
    free(heap_var);
}

int main(void)
{
    show_memory_layout();
    return 0;
}
```

**Layout tipico (Linux x86-64):**
```
[High Address 0x7FFFFFFFFFFF]
+---------------------------+
|       Stack               | <- Grows downward
|       ↓                   |
+---------------------------+
|       (unused)            |
+---------------------------+
|       ↑                   |
|       Heap                | <- Grows upward
+---------------------------+
|       .bss                | <- Uninitialized data
+---------------------------+
|       .data               | <- Initialized data
+---------------------------+
|       .rodata             | <- Read-only data
+---------------------------+
|       .text               | <- Code
+---------------------------+
[Low Address 0x400000]
```

## Cache e Prestazioni

### Cache Lines e False Sharing

```c
#include <stdio.h>
#include <pthread.h>
#include <time.h>

#define NUM_THREADS 4
#define ITERATIONS 10000000

/* BAD: False sharing - variabili vicine condivise */
struct {
    long counter[NUM_THREADS];  // Probabilmente nella stessa cache line!
} shared_bad;

/* GOOD: Padding per evitare false sharing */
struct {
    long counter;
    char padding[64 - sizeof(long)];  // Assicura cache line separata
} shared_good[NUM_THREADS];

void* worker_bad(void* arg)
{
    int id = *(int*)arg;
    for (int i = 0; i < ITERATIONS; i++) {
        shared_bad.counter[id]++;
    }
    return NULL;
}

void* worker_good(void* arg)
{
    int id = *(int*)arg;
    for (int i = 0; i < ITERATIONS; i++) {
        shared_good[id].counter++;
    }
    return NULL;
}

double benchmark(void* (*worker)(void*))
{
    pthread_t threads[NUM_THREADS];
    int ids[NUM_THREADS];
    
    clock_t start = clock();
    
    for (int i = 0; i < NUM_THREADS; i++) {
        ids[i] = i;
        pthread_create(&threads[i], NULL, worker, &ids[i]);
    }
    
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }
    
    clock_t end = clock();
    return ((double)(end - start)) / CLOCKS_PER_SEC;
}

int main(void)
{
    printf("Cache line size: %d bytes\n", 64);  // Tipico per x86-64
    printf("sizeof(long): %lu bytes\n\n", sizeof(long));
    
    double time_bad = benchmark(worker_bad);
    printf("With false sharing: %.6f seconds\n", time_bad);
    
    double time_good = benchmark(worker_good);
    printf("With padding: %.6f seconds\n", time_good);
    
    printf("\nSpeedup: %.2fx\n", time_bad / time_good);
    
    return 0;
}
```

### Prefetching

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define SIZE 10000000

/* Software prefetching */
void sum_with_prefetch(int *arr, long size)
{
    long sum = 0;
    for (long i = 0; i < size; i++) {
        /* Prefetch next cache line */
        __builtin_prefetch(&arr[i + 64], 0, 1);
        sum += arr[i];
    }
}

void sum_without_prefetch(int *arr, long size)
{
    long sum = 0;
    for (long i = 0; i < size; i++) {
        sum += arr[i];
    }
}

int main(void)
{
    int *arr = malloc(SIZE * sizeof(int));
    
    for (int i = 0; i < SIZE; i++) {
        arr[i] = i;
    }
    
    clock_t start, end;
    
    start = clock();
    sum_without_prefetch(arr, SIZE);
    end = clock();
    double time_without = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    start = clock();
    sum_with_prefetch(arr, SIZE);
    end = clock();
    double time_with = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("Without prefetch: %.6f seconds\n", time_without);
    printf("With prefetch: %.6f seconds\n", time_with);
    printf("Improvement: %.2f%%\n", (time_without - time_with) / time_without * 100);
    
    free(arr);
    return 0;
}
```

## Compilatore e Ottimizzazioni

### Livelli di Ottimizzazione GCC

```bash
# Nessuna ottimizzazione (debug)
gcc -O0 program.c -o program

# Ottimizzazioni base
gcc -O1 program.c -o program

# Ottimizzazioni più aggressive
gcc -O2 program.c -o program

# Massima ottimizzazione (può aumentare dimensione codice)
gcc -O3 program.c -o program

# Ottimizza per dimensione
gcc -Os program.c -o program

# Ottimizzazione estrema (può violare standard)
gcc -Ofast program.c -o program
```

### Volatile Keyword

```c
#include <stdio.h>

int main(void)
{
    /* Senza volatile - il compilatore può ottimizzare via il loop */
    int x = 0;
    while (x == 0) {
        // Compilatore: "x non cambia mai, loop infinito inutile!"
        // Potrebbe ottimizzare completamente via questo codice
    }
    
    /* Con volatile - il compilatore deve sempre leggere dalla memoria */
    volatile int y = 0;
    while (y == 0) {
        // Compilatore: "y potrebbe cambiare esternamente, devo controllare"
        // Mantiene il loop e legge sempre dalla memoria
    }
    
    return 0;
}
```

Uso tipico di `volatile`:
- Memory-mapped I/O registers
- Variabili modificate da ISR (Interrupt Service Routines)
- Variabili modificate da altri thread (meglio usare atomic)

### Restrict Keyword

```c
#include <stdio.h>
#include <string.h>

/* Senza restrict - compilatore deve assumere aliasing */
void copy_slow(int *dest, int *src, int n)
{
    for (int i = 0; i < n; i++) {
        dest[i] = src[i];
    }
}

/* Con restrict - compilatore sa che dest e src non si sovrappongono */
void copy_fast(int *restrict dest, int *restrict src, int n)
{
    for (int i = 0; i < n; i++) {
        dest[i] = src[i];
    }
}

int main(void)
{
    int arr1[100];
    int arr2[100];
    
    /* Inizializza */
    for (int i = 0; i < 100; i++) {
        arr1[i] = i;
    }
    
    /* Copy usando restrict permette migliori ottimizzazioni */
    copy_fast(arr2, arr1, 100);
    
    printf("First 5 elements: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr2[i]);
    }
    printf("\n");
    
    return 0;
}
```

## Debugging Low-Level

### GDB per Assembly

```bash
# Compila con simboli di debug
gcc -g -O0 program.c -o program

# Avvia GDB
gdb ./program

# Comandi GDB utili per assembly:
# layout asm          - Mostra assembly
# layout regs         - Mostra registri
# stepi (si)          - Step instruction
# nexti (ni)          - Next instruction
# info registers      - Mostra tutti i registri
# x/10x $rsp          - Esamina 10 word sullo stack
# disassemble func    - Disassembla funzione
# break *0x400123     - Breakpoint su indirizzo
```

### Valgrind per Memory Issues

```bash
# Controlla memory leaks
valgrind --leak-check=full ./program

# Controlla cache usage
valgrind --tool=cachegrind ./program

# Controlla heap profiling
valgrind --tool=massif ./program
```

## Esercizi

1. Scrivi una funzione in assembly inline che conta i bit a 1
2. Implementa una funzione che usa SIMD per moltiplicare due vettori
3. Analizza il layout di memoria di un programma complesso
4. Ottimizza un algoritmo evitando false sharing
5. Usa GDB per analizzare lo stack durante una chiamata ricorsiva

---

[← Avanzato](../04_Avanzato/README.md) | [Torna al Sommario](../README.md) | [Prossimo: Comparazioni →](../06_Comparazioni/README.md)
