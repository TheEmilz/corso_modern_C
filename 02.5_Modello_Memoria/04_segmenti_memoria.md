# Lezione 4 - Segmenti di Memoria

## 🎯 Obiettivi

- Comprendere i diversi segmenti di memoria (.text, .data, .bss, .rodata)
- Sapere dove vengono allocate le variabili
- Usare tools per ispezionare segmenti
- Comprendere protezioni memoria

---

## 1. ANATOMIA DI UN ESEGUIBILE

```
FILE ESEGUIBILE (ELF su Linux):

┌──────────────────────────────────┐
│ HEADER ELF                       │  ← Metadati
├──────────────────────────────────┤
│ .text (CODE)                     │  ← Codice macchina
│ - Funzioni compilate             │
│ - Read-Only, Executable          │
├──────────────────────────────────┤
│ .rodata (READ-ONLY DATA)         │  ← Costanti
│ - Stringhe letterali             │
│ - const variabili                │
│ - Read-Only, Non-Executable      │
├──────────────────────────────────┤
│ .data (INITIALIZED DATA)         │  ← Globali inizializzate
│ - Variabili globali = valore     │
│ - static inizializzate           │
│ - Read-Write, Non-Executable     │
├──────────────────────────────────┤
│ .bss (UNINITIALIZED DATA)        │  ← Globali non inizializzate
│ - Variabili globali senza init   │
│ - static senza init              │
│ - Inizializzate a 0 automatico   │
│ - Read-Write, Non-Executable     │
├──────────────────────────────────┤
│ Symbol Table                     │  ← Nomi funzioni/variabili
├──────────────────────────────────┤
│ Debug Info                       │  ← Per debugger
└──────────────────────────────────┘
```

---

## 2. DOVE VA OGNI VARIABILE?

```c
#include <stdio.h>
#include <stdlib.h>

// GLOBALI
int global_init = 42;           // → .data
int global_uninit;              // → .bss
const int global_const = 100;   // → .rodata
static int static_global = 5;   // → .data

const char *message = "Hello";  // message → .data (o stack in funzione)
                                // "Hello" → .rodata

void function() {
    // LOCALI
    int local = 10;             // → STACK
    static int static_local = 0;// → .bss (prima chiamata)
                                //   poi mantiene valore
    
    // HEAP
    int *heap_ptr = malloc(sizeof(int));  // heap_ptr → STACK
                                          // blocco → HEAP
    *heap_ptr = 20;
    
    free(heap_ptr);
}

int main() {
    function();
    return 0;
}
```

**VISUALIZZAZIONE:**

```
┌─ .text ────────────────────┐
│ Codice di function()       │
│ Codice di main()           │
└────────────────────────────┘

┌─ .rodata ──────────────────┐
│ "Hello\0"                  │
│ global_const = 100         │
└────────────────────────────┘

┌─ .data ────────────────────┐
│ global_init = 42           │
│ static_global = 5          │
│ message = 0x... (→rodata)  │
└────────────────────────────┘

┌─ .bss ─────────────────────┐
│ global_uninit = 0          │
│ static_local = 0           │
└────────────────────────────┘

┌─ HEAP ─────────────────────┐
│ malloc(sizeof(int)) = 20   │
└────────────────────────────┘

┌─ STACK ────────────────────┐
│ local = 10                 │
│ heap_ptr = 0x... (→heap)   │
└────────────────────────────┘
```

---

## 3. TOOLS PER ISPEZIONARE

### size - Dimensioni Segmenti

```bash
$ gcc -o program program.c
$ size program

   text    data     bss     dec     hex filename
   1234     512     128    1874     752 program

# text: codice + rodata
# data: variabili globali inizializzate
# bss: variabili globali non inizializzate
# dec/hex: totale in decimale/esadecimale
```

### nm - Simboli

```bash
$ nm program

0000000000001129 T main
0000000000001149 T function
0000000000004010 D global_init
0000000000004020 B global_uninit
0000000000002000 R global_const

# T = .text (funzione)
# D = .data (inizializzata)
# B = .bss (non inizializzata)
# R = .rodata (read-only)
# Indirizzo in prima colonna
```

### readelf - Dettagli Segmenti

```bash
$ readelf -S program

Section Headers:
  [Nr] Name              Type            Address          Off    Size
  [ 1] .text             PROGBITS        0000000000001000  001000 000200
  [ 2] .rodata           PROGBITS        0000000000002000  002000 000020
  [ 3] .data             PROGBITS        0000000000004000  003000 000010
  [ 4] .bss              NOBITS          0000000000004010  003010 000008

# Address: indirizzo in memoria virtuale
# Off: offset nel file
# Size: dimensione in byte
```

### objdump - Disassemblare

```bash
$ objdump -d program

0000000000001129 <main>:
    1129:       55                      push   %rbp
    112a:       48 89 e5                mov    %rsp,%rbp
    112d:       b8 00 00 00 00          mov    $0x0,%eax
    ...

# Mostra codice assembly della funzione main
```

---

## 4. PROTEZIONI MEMORIA

### NX Bit (Non-Executable)

```
PROBLEMA: Buffer overflow attack
Attaccante inietta codice nello stack/heap e lo esegue.

SOLUZIONE: NX (No-eXecute) bit
- STACK: Read-Write, NON-Executable
- HEAP: Read-Write, NON-Executable
- .text: Read-Only, Executable

Se provi a eseguire codice dallo stack → CRASH!

Verifica:
$ readelf -l program | grep GNU_STACK
  GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000
                 ...        RW     ← Read-Write ma NON Executable!
```

### ASLR (Address Space Layout Randomization)

```
SENZA ASLR (vecchio):
Ogni esecuzione → STESSI indirizzi
Attaccante SA dove sono stack/heap/librerie

CON ASLR (moderno):
Ogni esecuzione → indirizzi RANDOMIZZATI

Esempio:
$ ./program
Stack @ 0x7ffd1234...

$ ./program  (seconda esecuzione)
Stack @ 0x7ffe5678...  ← DIVERSO!

Verifica:
$ cat /proc/sys/kernel/randomize_va_space
2   ← 2 = ASLR completo
    0 = disabilitato
    1 = parziale
```

---

## 5. VIRTUAL MEMORY

```
OGNI PROCESSO vede:
- Spazio indirizzi VIRTUALE (es: 0x0 - 0x7FFFFFFFFFFF su 64-bit)
- ISOLATO dagli altri processi

MAPPING virtuale → fisico gestito da MMU (Memory Management Unit)

VANTAGGI:
✓ Isolamento: processo A non può accedere memoria di B
✓ Protezione: kernel può marcare pagine read-only/non-exec
✓ Swap: pagine non usate → disco
✓ Shared libraries: stessa libreria condivisa tra processi

COPY-ON-WRITE:
fork() crea processo figlio
Inizialmente: figlio CONDIVIDE pagine col padre (read-only)
Quando uno scrive: pagina COPIATA → privata

MEMORY MAPPING (mmap):
Mappa file direttamente in memoria
Accesso file = accesso memoria (no read/write esplicite)
```

---

## 6. ESEMPIO PRATICO

```c
#include <stdio.h>
#include <stdlib.h>

int global = 10;
static int static_var = 20;
const int const_var = 30;

int main() {
    int local = 40;
    int *heap = malloc(sizeof(int));
    *heap = 50;
    
    printf("Indirizzi:\n");
    printf("global:     %p (data)\n", (void*)&global);
    printf("static_var: %p (data)\n", (void*)&static_var);
    printf("const_var:  %p (rodata)\n", (void*)&const_var);
    printf("main:       %p (text)\n", (void*)main);
    printf("local:      %p (stack)\n", (void*)&local);
    printf("heap:       %p (heap)\n", (void*)heap);
    
    free(heap);
    return 0;
}

/* OUTPUT ESEMPIO (indirizzi variano):
 * global:     0x404020 (data)
 * static_var: 0x404024 (data)
 * const_var:  0x402000 (rodata)
 * main:       0x401126 (text)
 * local:      0x7ffd1234abcd (stack)
 * heap:       0x1a2b3c4d5e6f (heap)
 * 
 * NOTA: stack ha indirizzi ALTI, heap BASSI
 */
```

---

## 7. 🔧 ESERCIZIO

Compila e analizza:

```c
int x = 1;
int y;
const char *msg = "Test";

int main() {
    static int z = 2;
    int w = 3;
    return 0;
}
```

Usa `nm`, `size`, `readelf` per trovare dove va ogni variabile.

<details>
<summary>Soluzione</summary>

```
x → .data (globale inizializzata)
y → .bss (globale non inizializzata)
msg → .data (puntatore), "Test" → .rodata (stringa)
z → .data (static inizializzata)
w → stack (locale)

Verifica con:
$ gcc -o test test.c
$ nm test | grep ' x'
$ nm test | grep ' y'
$ nm test | grep ' z'
```
</details>

---

[← Endianness](03_endianness.md) | [Torna al Modulo](README.md) | [Modulo 03 →](../03_Intermedio/README.md)
