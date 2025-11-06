# Glossario Completo dei Comandi di Compilazione C

## Indice
1. [Processo di Compilazione](#processo-di-compilazione)
2. [Comando GCC Base](#comando-gcc-base)
3. [Opzioni di Compilazione](#opzioni-di-compilazione)
4. [Preprocessore](#preprocessore)
5. [Debugging](#debugging)
6. [Ottimizzazione](#ottimizzazione)
7. [Linking e Librerie](#linking-e-librerie)
8. [Scenari Comuni](#scenari-comuni)
9. [Risoluzione Problemi](#risoluzione-problemi)

---

## Processo di Compilazione

La compilazione di un programma C avviene in **4 fasi distinte**:

### 1. Preprocessing (Preprocessore)
Il preprocessore elabora le direttive che iniziano con `#`:
- Espande le macro (`#define`)
- Include i file header (`#include`)
- Gestisce le direttive condizionali (`#ifdef`, `#ifndef`)

```bash
# Eseguire solo il preprocessing
gcc -E programma.c -o programma.i
```

**Spiegazione comando:**
- `gcc`: Invoca il compilatore GNU C
- `-E`: Ferma la compilazione dopo la fase di preprocessing
- `programma.c`: Il file sorgente da processare
- `-o programma.i`: Nome del file di output (per convenzione, estensione `.i`)

**Cosa succede:** Il preprocessore sostituisce tutte le inclusioni e macro, producendo un file `.i` che contiene solo codice C puro senza direttive del preprocessore.

### 2. Compilation (Compilazione vera e propria)
Il compilatore traduce il codice C in linguaggio Assembly:

```bash
# Eseguire preprocessing e compilazione
gcc -S programma.c -o programma.s
```

**Spiegazione comando:**
- `-S`: Ferma la compilazione dopo la generazione del codice Assembly
- `programma.s`: File Assembly generato (estensione `.s`)

**Cosa succede:** Il compilatore analizza la sintassi C, verifica la correttezza del codice e lo traduce in istruzioni Assembly specifiche per l'architettura target (es. x86-64).

### 3. Assembly (Assemblaggio)
L'assemblatore converte il codice Assembly in codice macchina (object code):

```bash
# Eseguire preprocessing, compilazione e assemblaggio
gcc -c programma.c -o programma.o
```

**Spiegazione comando:**
- `-c`: Ferma dopo l'assemblaggio, produce un file oggetto
- `programma.o`: File oggetto binario (estensione `.o`)

**Cosa succede:** L'assemblatore traduce le istruzioni Assembly in codice binario (0 e 1) che il processore può eseguire. Il file oggetto contiene:
- Codice macchina delle funzioni
- Dati inizializzati
- Tabella dei simboli (nomi di funzioni e variabili)
- Informazioni di relocation

### 4. Linking (Collegamento)
Il linker combina tutti i file oggetto e le librerie necessarie in un eseguibile:

```bash
# Compilazione completa (tutte le fasi)
gcc programma.c -o programma
```

**Spiegazione comando:**
- `programma.c`: File sorgente
- `-o programma`: Nome dell'eseguibile finale

**Cosa succede:** Il linker:
1. Risolve i riferimenti ai simboli (funzioni e variabili esterne)
2. Collega le librerie standard (es. `printf` da libc)
3. Assegna gli indirizzi finali in memoria
4. Crea l'eseguibile finale

---

## Comando GCC Base

### Sintassi Generale
```bash
gcc [opzioni] file_sorgente.c -o nome_eseguibile
```

### Esempio Base Annotato
```bash
gcc hello.c -o hello
```

**Analisi dettagliata:**
1. `gcc`: Invoca il GNU Compiler Collection
2. `hello.c`: Nome del file sorgente da compilare
3. `-o`: Flag che specifica il nome del file di output
4. `hello`: Nome dell'eseguibile da creare

**Senza `-o`:**
```bash
gcc hello.c
```
Crea un eseguibile di nome `a.out` (default Unix/Linux) o `a.exe` (Windows)

### Compilare ed Eseguire
```bash
# Compila
gcc programma.c -o programma

# Esegui su Linux/macOS
./programma

# Esegui su Windows
programma.exe
```

**Perché `./`?**
- Il punto `.` rappresenta la directory corrente
- `/` è il separatore di directory
- `./programma` dice al sistema: "esegui il file `programma` nella directory corrente"
- Senza `./`, il sistema cerca solo nelle directory del PATH

---

## Opzioni di Compilazione

### Warning (Avvisi)

#### `-Wall` (Warning All)
Abilita un set completo di warning comuni:

```bash
gcc -Wall programma.c -o programma
```

**Warning abilitati da `-Wall`:**
- Variabili non inizializzate
- Funzioni senza valore di ritorno
- Conversioni implicite pericolose
- Codice irraggiungibile
- Confronti sempre veri/falsi
- Parametri non usati

**Esempio:**
```c
#include <stdio.h>

int main(void)
{
    int x;  // Non inizializzato
    printf("%d\n", x);  // -Wall segnalerà un warning
    return 0;
}
```

Compilando con `-Wall`:
```bash
$ gcc -Wall test.c -o test
test.c: In function 'main':
test.c:5:20: warning: 'x' is used uninitialized [-Wuninitialized]
    5 |     printf("%d\n", x);
      |                    ^
```

#### `-Wextra` (Warning Extra)
Abilita warning aggiuntivi non inclusi in `-Wall`:

```bash
gcc -Wall -Wextra programma.c -o programma
```

**Warning aggiuntivi:**
- Confronto tra signed e unsigned
- Parametri non usati nelle funzioni
- Inizializzatori mancanti nelle struct
- Puntatori che potrebbero essere NULL

#### `-Werror` (Warning as Error)
Tratta tutti i warning come errori (blocca la compilazione):

```bash
gcc -Wall -Wextra -Werror programma.c -o programma
```

**Utile per:**
- Progetti professionali che richiedono codice pulito
- Continuous Integration (CI)
- Evitare di ignorare warning importanti

#### `-pedantic`
Impone stretta aderenza allo standard C:

```bash
gcc -Wall -Wextra -pedantic programma.c -o programma
```

**Cosa controlla:**
- Estensioni non standard del compilatore
- Codice che viola lo standard C
- Uso di feature deprecate

### Standard C

#### `-std=c89` o `-std=c90`
Compila secondo lo standard ANSI C (C89/C90):

```bash
gcc -std=c89 programma.c -o programma
```

**Limiti:**
- No commenti `//`
- No variabili dichiarate nel mezzo del codice
- No tipo `bool` (serve definirlo manualmente)
- No tipo `long long`

#### `-std=c99`
Compila secondo lo standard C99:

```bash
gcc -std=c99 programma.c -o programma
```

**Nuove feature:**
- Commenti stile C++ (`//`)
- Variabili dichiarabili ovunque
- Tipo `bool` (con `<stdbool.h>`)
- Tipo `long long`
- Array a lunghezza variabile (VLA)
- Keyword `inline`
- Funzione `snprintf`

#### `-std=c11`
Compila secondo lo standard C11:

```bash
gcc -std=c11 programma.c -o programma
```

**Nuove feature:**
- Multi-threading (`<threads.h>`)
- Operazioni atomiche (`<stdatomic.h>`)
- `_Generic` keyword
- `_Static_assert`
- Unicode support migliorato

#### `-std=c17` o `-std=c18`
```bash
gcc -std=c17 programma.c -o programma
```

**Note:** C17 è principalmente una correzione di bug di C11, senza nuove feature.

#### `-std=c23`
```bash
gcc -std=c23 programma.c -o programma
```

**Nuove feature (richiede GCC 13+):**
- `nullptr` constant
- Attributi migliorati
- `typeof` operator
- Miglioramenti ai preprocessore

### Compilazione Multi-File

#### Metodo 1: Compilazione Diretta
```bash
gcc main.c utils.c math_ops.c -o programma
```

**Spiegazione:**
- Compila tutti i file `.c` insieme
- Genera un unico eseguibile
- **Svantaggio:** Ricompila tutto anche se cambi un solo file

#### Metodo 2: Compilazione Separata (Consigliata)
```bash
# Passo 1: Compila ogni file in file oggetto
gcc -c main.c -o main.o
gcc -c utils.c -o utils.o
gcc -c math_ops.c -o math_ops.o

# Passo 2: Linka tutti i file oggetto
gcc main.o utils.o math_ops.o -o programma
```

**Vantaggi:**
- Ricompili solo i file modificati
- Più veloce per progetti grandi
- Usato dai sistemi di build (Make, CMake)

#### Esempio con Header Files
Struttura del progetto:
```
progetto/
  ├── main.c
  ├── utils.c
  ├── utils.h
  ├── math_ops.c
  └── math_ops.h
```

**utils.h:**
```c
#ifndef UTILS_H
#define UTILS_H

void stampa_messaggio(const char *msg);

#endif
```

**utils.c:**
```c
#include <stdio.h>
#include "utils.h"

void stampa_messaggio(const char *msg)
{
    printf("%s\n", msg);
}
```

**main.c:**
```c
#include "utils.h"

int main(void)
{
    stampa_messaggio("Hello from utils!");
    return 0;
}
```

**Compilazione:**
```bash
gcc -c utils.c -o utils.o
gcc -c main.c -o main.o
gcc main.o utils.o -o programma
```

---

## Preprocessore

### Visualizzare Output del Preprocessore
```bash
gcc -E programma.c
```

**Output:** Mostra tutto il codice dopo l'espansione delle macro e inclusione dei file.

**Esempio utile per debug:**
```c
#define SQUARE(x) x * x

int main(void) {
    int result = SQUARE(3 + 2);  // Bug!
    return 0;
}
```

```bash
$ gcc -E program.c
# ... output del preprocessore ...
int main(void) {
    int result = 3 + 2 * 3 + 2;  // Espanso a: 11, non 25!
    return 0;
}
```

### Definire Macro da Linea di Comando
```bash
gcc -DDEBUG programma.c -o programma
```

**Equivalente a scrivere nel codice:**
```c
#define DEBUG
```

**Con valore:**
```bash
gcc -DMAX_SIZE=100 programma.c -o programma
```

**Uso nel codice:**
```c
#ifdef DEBUG
    printf("Debug mode attivo\n");
#endif

int buffer[MAX_SIZE];  // MAX_SIZE = 100
```

### Path di Inclusione
```bash
gcc -I/percorso/include programma.c -o programma
```

**Spiegazione:**
- `-I`: Aggiunge directory per cercare file header
- Utile per librerie installate in posizioni non standard

**Esempio:**
```bash
gcc -I/usr/local/include -I../lib/include main.c -o main
```

---

## Debugging

### Informazioni di Debug (`-g`)
```bash
gcc -g programma.c -o programma
```

**Cosa fa `-g`:**
- Inserisce simboli di debugging nell'eseguibile
- Mappa il codice macchina al codice sorgente
- Permette di usare debugger come GDB
- Include nomi di variabili e funzioni
- Aggiunge informazioni sui numeri di linea

**Eseguibile senza `-g`:** ~10 KB
**Eseguibile con `-g`:** ~50 KB (varia in base al progetto)

### Usare GDB (GNU Debugger)

#### Compilazione per Debug
```bash
gcc -g -Wall programma.c -o programma
```

#### Avviare GDB
```bash
gdb ./programma
```

#### Comandi GDB Fondamentali

**Impostare breakpoint:**
```gdb
(gdb) break main
Breakpoint 1 at 0x1149: file programma.c, line 5.

(gdb) break programma.c:10
Breakpoint 2 at 0x1165: file programma.c, line 10.

(gdb) break mia_funzione
Breakpoint 3 at 0x1180: file programma.c, line 25.
```

**Eseguire il programma:**
```gdb
(gdb) run
Starting program: /home/user/programma
```

**Eseguire con argomenti:**
```gdb
(gdb) run arg1 arg2
```

**Navigazione:**
```gdb
(gdb) next          # Esegue la prossima linea (non entra nelle funzioni)
(gdb) step          # Esegue la prossima linea (entra nelle funzioni)
(gdb) continue      # Continua fino al prossimo breakpoint
(gdb) finish        # Esegue fino alla fine della funzione corrente
```

**Ispezionare variabili:**
```gdb
(gdb) print x       # Stampa il valore di x
$1 = 42

(gdb) print &x      # Stampa l'indirizzo di x
$2 = (int *) 0x7fffffffdc4c

(gdb) print *ptr    # Dereferenzia un puntatore
$3 = 100

(gdb) print arr[3]  # Accedi a un elemento array
$4 = 30
```

**Visualizzare il codice:**
```gdb
(gdb) list          # Mostra il codice sorgente
(gdb) list 10       # Mostra codice attorno alla linea 10
(gdb) list mia_funzione  # Mostra il codice di una funzione
```

**Stack trace:**
```gdb
(gdb) backtrace     # Mostra lo stack delle chiamate
#0  funzione_c () at programma.c:25
#1  funzione_b () at programma.c:15
#2  funzione_a () at programma.c:10
#3  main () at programma.c:5
```

**Uscire:**
```gdb
(gdb) quit
```

#### Esempio Completo di Debug Session

**Codice (bug.c):**
```c
#include <stdio.h>

int somma(int a, int b)
{
    int risultato = a + b;
    return risultato;
}

int main(void)
{
    int x = 5;
    int y = 10;
    int z = somma(x, y);
    printf("Risultato: %d\n", z);
    return 0;
}
```

**Sessione GDB:**
```bash
$ gcc -g bug.c -o bug
$ gdb ./bug

(gdb) break main
Breakpoint 1 at 0x1149: file bug.c, line 10.

(gdb) run
Starting program: /home/user/bug

Breakpoint 1, main () at bug.c:10
10          int x = 5;

(gdb) next
11          int y = 10;

(gdb) print x
$1 = 5

(gdb) next
12          int z = somma(x, y);

(gdb) step
somma (a=5, b=10) at bug.c:4
4           int risultato = a + b;

(gdb) next
5           return risultato;

(gdb) print risultato
$2 = 15

(gdb) continue
Continuing.
Risultato: 15
[Inferior 1 (process 12345) exited normally]

(gdb) quit
```

### Valgrind - Memory Checker

Valgrind è uno strumento per rilevare:
- Memory leaks (memoria non liberata)
- Accessi a memoria non valida
- Uso di memoria non inizializzata

#### Installazione
```bash
# Ubuntu/Debian
sudo apt install valgrind

# macOS
brew install valgrind
```

#### Uso Base
```bash
valgrind ./programma
```

#### Memory Leak Check
```bash
valgrind --leak-check=full ./programma
```

#### Esempio con Memory Leak

**leak.c:**
```c
#include <stdlib.h>

int main(void)
{
    int *ptr = malloc(10 * sizeof(int));
    // Dimenticato free(ptr)!
    return 0;
}
```

**Compilazione e check:**
```bash
$ gcc -g leak.c -o leak
$ valgrind --leak-check=full ./leak

==12345== HEAP SUMMARY:
==12345==     in use at exit: 40 bytes in 1 blocks
==12345==   total heap usage: 1 allocs, 0 frees, 40 bytes allocated
==12345==
==12345== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1
==12345==    at 0x4C2DB8F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==12345==    by 0x10915B: main (leak.c:5)
==12345==
==12345== LEAK SUMMARY:
==12345==    definitely lost: 40 bytes in 1 blocks
```

---

## Ottimizzazione

GCC offre vari livelli di ottimizzazione per migliorare le performance del codice.

### Livelli di Ottimizzazione

#### `-O0` (Default)
Nessuna ottimizzazione:
```bash
gcc programma.c -o programma
# Equivalente a:
gcc -O0 programma.c -o programma
```

**Caratteristiche:**
- Compilazione più veloce
- Debug più facile
- Codice non ottimizzato
- **Usa per:** sviluppo e debugging

#### `-O1` (Ottimizzazione Base)
```bash
gcc -O1 programma.c -o programma
```

**Ottimizzazioni:**
- Eliminazione di codice morto
- Semplificazione di espressioni
- Inlining di funzioni piccole
- Ottimizzazioni locali

**Bilancio:** Buon compromesso tra velocità di compilazione e performance

#### `-O2` (Ottimizzazione Standard)
```bash
gcc -O2 programma.c -o programma
```

**Ottimizzazioni aggiuntive:**
- Loop unrolling parziale
- Function inlining aggressivo
- Ottimizzazioni di scheduling
- Ottimizzazioni di register allocation

**Uso:** Standard per build di release

#### `-O3` (Ottimizzazione Massima)
```bash
gcc -O3 programma.c -o programma
```

**Ottimizzazioni aggressive:**
- Loop vectorization
- Loop unrolling completo
- Ottimizzazioni interprocedurali
- Predicazione delle branch

**Attenzione:**
- Compilazione molto più lenta
- Binario più grande
- Possibili problemi con codice non standard

#### `-Os` (Ottimizza per Dimensione)
```bash
gcc -Os programma.c -o programma
```

**Obiettivo:** Ridurre la dimensione del binario
**Uso:** Sistemi embedded con memoria limitata

#### `-Ofast` (Oltre O3)
```bash
gcc -Ofast programma.c -o programma
```

**Attenzione:** Può violare lo standard C (es. ottimizzazioni floating-point aggressive)

### Confronto Performance

**Esempio (fibonacci.c):**
```c
#include <stdio.h>

unsigned long long fib(int n)
{
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);
}

int main(void)
{
    printf("fib(45) = %llu\n", fib(45));
    return 0;
}
```

**Test di performance:**
```bash
# Senza ottimizzazione
$ gcc fibonacci.c -o fib_O0
$ time ./fib_O0
fib(45) = 1134903170
real    0m8.234s

# Con O2
$ gcc -O2 fibonacci.c -o fib_O2
$ time ./fib_O2
fib(45) = 1134903170
real    0m5.123s

# Con O3
$ gcc -O3 fibonacci.c -o fib_O3
$ time ./fib_O3
fib(45) = 1134903170
real    0m4.891s
```

### Ottimizzazioni Specifiche

#### `-march=native`
Ottimizza per l'architettura del processore corrente:
```bash
gcc -O3 -march=native programma.c -o programma
```

**Attenzione:** L'eseguibile potrebbe non funzionare su CPU diverse

#### `-flto` (Link Time Optimization)
Ottimizzazione durante il linking:
```bash
gcc -O3 -flto main.c utils.c -o programma
```

**Vantaggi:** Ottimizzazioni interprocedurali tra file diversi

---

## Linking e Librerie

### Linking di Librerie Standard

#### Libreria Matematica (`-lm`)
```bash
gcc programma.c -o programma -lm
```

**Quando serve:**
- Funzioni in `<math.h>`: `sqrt()`, `sin()`, `cos()`, `pow()`, ecc.

**Esempio:**
```c
#include <stdio.h>
#include <math.h>

int main(void)
{
    double x = 16.0;
    double radice = sqrt(x);
    printf("Radice di %.2f = %.2f\n", x, radice);
    return 0;
}
```

```bash
gcc math_example.c -o math_example -lm
```

**Senza `-lm`:**
```bash
$ gcc math_example.c -o math_example
/usr/bin/ld: /tmp/ccXXXXXX.o: undefined reference to `sqrt'
collect2: error: ld returned 1 exit status
```

#### Libreria Thread POSIX (`-lpthread`)
```bash
gcc programma.c -o programma -lpthread
```

**Quando serve:**
- Multi-threading con `<pthread.h>`

#### Libreria Real-time (`-lrt`)
```bash
gcc programma.c -o programma -lrt
```

**Quando serve:**
- Timer ad alta precisione
- Funzioni in `<time.h>` avanzate

### Librerie Personalizzate

#### Creare una Libreria Statica (`.a`)

**Passo 1: Compilare i file oggetto**
```bash
gcc -c libreria.c -o libreria.o
gcc -c utils.c -o utils.o
```

**Passo 2: Creare l'archivio**
```bash
ar rcs libmialibreria.a libreria.o utils.o
```

**Spiegazione comando `ar`:**
- `ar`: Archiver tool
- `r`: Inserisci/sostituisci file nell'archivio
- `c`: Crea l'archivio se non esiste
- `s`: Crea indice dei simboli
- `libmialibreria.a`: Nome della libreria (convenzione: `lib` + nome + `.a`)

**Passo 3: Usare la libreria**
```bash
gcc main.c -L. -lmialibreria -o programma
```

**Spiegazione:**
- `-L.`: Cerca librerie nella directory corrente (`.`)
- `-lmialibreria`: Linka con `libmialibreria.a`

#### Creare una Libreria Dinamica (`.so` su Linux, `.dll` su Windows)

**Passo 1: Compilare con Position Independent Code**
```bash
gcc -c -fPIC libreria.c -o libreria.o
gcc -c -fPIC utils.c -o utils.o
```

**Spiegazione `-fPIC`:**
- Position Independent Code
- Necessario per librerie condivise
- Permette al codice di essere caricato a qualsiasi indirizzo

**Passo 2: Creare la libreria condivisa**
```bash
gcc -shared libreria.o utils.o -o libmialibreria.so
```

**Spiegazione:**
- `-shared`: Crea una libreria condivisa

**Passo 3: Usare la libreria**
```bash
gcc main.c -L. -lmialibreria -o programma
```

**Passo 4: Eseguire (Linux)**
```bash
LD_LIBRARY_PATH=. ./programma
```

**Spiegazione:**
- `LD_LIBRARY_PATH`: Directory dove cercare librerie condivise
- Oppure installa la libreria in `/usr/lib` o `/usr/local/lib`

### Visualizzare Dipendenze

#### Linux: `ldd`
```bash
ldd ./programma
```

**Output esempio:**
```
linux-vdso.so.1 =>  (0x00007fff2a7fe000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f9e1b200000)
/lib64/ld-linux-x86-64.so.2 (0x00007f9e1b800000)
```

#### macOS: `otool`
```bash
otool -L ./programma
```

#### Windows: `depends.exe` (Dependency Walker)

---

## Scenari Comuni

### Scenario 1: Progetto Semplice a File Singolo

**hello.c:**
```c
#include <stdio.h>

int main(void)
{
    printf("Hello, World!\n");
    return 0;
}
```

**Compilazione base:**
```bash
gcc hello.c -o hello
./hello
```

**Compilazione con controlli:**
```bash
gcc -Wall -Wextra -std=c11 hello.c -o hello
./hello
```

### Scenario 2: Progetto Multi-File con Header

**Struttura:**
```
progetto/
  ├── main.c
  ├── calc.c
  ├── calc.h
  └── Makefile
```

**calc.h:**
```c
#ifndef CALC_H
#define CALC_H

int somma(int a, int b);
int sottrazione(int a, int b);

#endif
```

**calc.c:**
```c
#include "calc.h"

int somma(int a, int b)
{
    return a + b;
}

int sottrazione(int a, int b)
{
    return a - b;
}
```

**main.c:**
```c
#include <stdio.h>
#include "calc.h"

int main(void)
{
    int x = 10, y = 5;
    printf("%d + %d = %d\n", x, y, somma(x, y));
    printf("%d - %d = %d\n", x, y, sottrazione(x, y));
    return 0;
}
```

**Compilazione manuale:**
```bash
# Metodo 1: Tutto insieme
gcc -Wall -Wextra main.c calc.c -o programma

# Metodo 2: Separata (consigliata)
gcc -Wall -Wextra -c calc.c -o calc.o
gcc -Wall -Wextra -c main.c -o main.o
gcc main.o calc.o -o programma
```

**Makefile:**
```makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c11
TARGET = programma
OBJS = main.o calc.o

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(OBJS) -o $(TARGET)

main.o: main.c calc.h
	$(CC) $(CFLAGS) -c main.c

calc.o: calc.c calc.h
	$(CC) $(CFLAGS) -c calc.c

clean:
	rm -f $(OBJS) $(TARGET)
```

**Uso del Makefile:**
```bash
make        # Compila
./programma # Esegui
make clean  # Pulisci
```

### Scenario 3: Progetto con Libreria Matematica

**physics.c:**
```c
#include <stdio.h>
#include <math.h>

#define G 9.81  // Accelerazione gravitazionale (m/s²)

double calcola_velocita(double altezza)
{
    // v = sqrt(2 * g * h)
    return sqrt(2 * G * altezza);
}

int main(void)
{
    double h = 10.0;  // metri
    double v = calcola_velocita(h);
    printf("Velocità dopo caduta di %.2fm: %.2f m/s\n", h, v);
    return 0;
}
```

**Compilazione:**
```bash
gcc -Wall physics.c -o physics -lm
./physics
```

**Output:**
```
Velocità dopo caduta di 10.00m: 14.01 m/s
```

### Scenario 4: Debug di un Programma con Errore

**bug.c:**
```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *arr = malloc(10 * sizeof(int));
    
    for (int i = 0; i <= 10; i++) {  // BUG: <= dovrebbe essere <
        arr[i] = i * 2;
    }
    
    for (int i = 0; i < 10; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    
    free(arr);
    return 0;
}
```

**Compilazione con debug:**
```bash
gcc -g -Wall bug.c -o bug
```

**Esecuzione normale:**
```bash
./bug
# Potrebbe crashare o comportarsi stranamente
```

**Debug con Valgrind:**
```bash
valgrind --leak-check=full ./bug
```

**Output Valgrind:**
```
==12345== Invalid write of size 4
==12345==    at 0x109189: main (bug.c:8)
==12345==  Address 0x522d068 is 0 bytes after a block of size 40 alloc'd
```

**Debug con GDB:**
```bash
gdb ./bug
(gdb) break 8
(gdb) run
(gdb) print i
(gdb) print arr[i]
```

### Scenario 5: Ottimizzazione per Produzione

**game_loop.c:**
```c
#include <stdio.h>

double complesso_calcolo(int n)
{
    double result = 0.0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            result += (i * j) / (i + j + 1.0);
        }
    }
    return result;
}

int main(void)
{
    double r = complesso_calcolo(5000);
    printf("Risultato: %f\n", r);
    return 0;
}
```

**Build per sviluppo:**
```bash
gcc -g -Wall -Wextra game_loop.c -o game_debug -lm
```

**Build per produzione:**
```bash
gcc -O3 -march=native -DNDEBUG game_loop.c -o game_release -lm
```

**Confronto performance:**
```bash
time ./game_debug
time ./game_release
```

---

## Risoluzione Problemi

### Errore: `undefined reference to 'funzione'`

**Causa:** Il linker non trova la definizione della funzione.

**Soluzioni:**
1. Hai dimenticato di compilare/linkare un file
2. Hai dimenticato di linkare una libreria

**Esempio:**
```bash
# Errore
$ gcc main.c -o prog
/usr/bin/ld: /tmp/cc123.o: undefined reference to `somma'

# Soluzione: includi il file mancante
$ gcc main.c calc.c -o prog
```

### Errore: `implicit declaration of function`

**Causa:** Il compilatore non conosce la funzione (manca l'header o il prototipo).

**Soluzione:** Includi l'header corretto.

**Esempio:**
```c
// Errore: manca #include <string.h>
int main(void) {
    char str[10];
    strcpy(str, "test");  // warning: implicit declaration
    return 0;
}
```

**Soluzione:**
```c
#include <string.h>  // Aggiunto!

int main(void) {
    char str[10];
    strcpy(str, "test");
    return 0;
}
```

### Errore: `permission denied`

**Causa:** Il file non ha permessi di esecuzione.

**Soluzione:**
```bash
chmod +x programma
./programma
```

### Errore: `cannot find -lnome`

**Causa:** Il linker non trova la libreria `libnome.a` o `libnome.so`.

**Soluzioni:**
1. Installa la libreria
2. Specifica il path con `-L`

**Esempio:**
```bash
# Errore
$ gcc main.c -o prog -lmylib
/usr/bin/ld: cannot find -lmylib

# Soluzione
$ gcc main.c -o prog -L/percorso/libreria -lmylib
```

### Errore: `No such file or directory` (per header)

**Causa:** Il compilatore non trova il file header.

**Soluzione:** Specifica il path con `-I`.

**Esempio:**
```bash
# Errore
$ gcc main.c -o prog
main.c:1:10: fatal error: myheader.h: No such file or directory

# Soluzione
$ gcc -I/percorso/include main.c -o prog
```

### Warning: `comparison between signed and unsigned`

**Causa:** Confronto tra tipi signed e unsigned.

**Esempio:**
```c
int i;
size_t len = strlen(str);
if (i < len) { ... }  // warning
```

**Soluzione:**
```c
size_t i;
size_t len = strlen(str);
if (i < len) { ... }  // OK
```

### Segmentation Fault

**Cause comuni:**
1. Dereferenziazione di puntatore NULL
2. Accesso a memoria non allocata
3. Buffer overflow
4. Stack overflow (ricorsione infinita)

**Debug:**
```bash
# Con GDB
gdb ./programma
(gdb) run
# Dopo il crash
(gdb) backtrace
(gdb) print variabile

# Con Valgrind
valgrind ./programma
```

---

## Comandi Utili Aggiuntivi

### Visualizzare Simboli in un File Oggetto
```bash
nm programma.o
```

**Output:**
```
0000000000000000 T main
                 U printf
0000000000000000 T somma
```

**Legenda:**
- `T`: Simbolo definito nella sezione text (codice)
- `U`: Simbolo non definito (da linkare)
- `D`: Simbolo nella sezione data (variabili inizializzate)
- `B`: Simbolo nella sezione BSS (variabili non inizializzate)

### Disassemblare un Eseguibile
```bash
objdump -d programma
```

**Mostra:** Il codice Assembly del programma.

### Visualizzare Sezioni di un Eseguibile
```bash
objdump -h programma
```

**Mostra:** Le sezioni dell'eseguibile (.text, .data, .bss, ecc.)

### Dimensione delle Sezioni
```bash
size programma
```

**Output:**
```
   text    data     bss     dec     hex filename
   1234     568     128    1930     78a programma
```

### Strippare Simboli di Debug (Ridurre Dimensione)
```bash
strip programma
```

**Effetto:** Rimuove simboli di debug, riduce la dimensione del file.

**Nota:** Non più possibile il debug con GDB dopo strip!

### File Preprocessato Salvato
```bash
gcc -E -o output.i input.c
```

### File Assembly Generato
```bash
gcc -S -o output.s input.c
```

### Preprocessing, Compilation, Assembly in un colpo
```bash
gcc -save-temps programma.c -o programma
```

**Genera:**
- `programma.i` (preprocessato)
- `programma.s` (assembly)
- `programma.o` (oggetto)
- `programma` (eseguibile)

---

## Riepilogo Comandi Essenziali

| Comando | Descrizione |
|---------|-------------|
| `gcc file.c -o prog` | Compilazione base |
| `gcc -Wall -Wextra file.c -o prog` | Con tutti i warning |
| `gcc -g file.c -o prog` | Con info di debug |
| `gcc -O2 file.c -o prog` | Con ottimizzazione |
| `gcc -std=c11 file.c -o prog` | Standard C11 |
| `gcc file.c -o prog -lm` | Linka libreria math |
| `gcc -c file.c -o file.o` | Compila solo file oggetto |
| `gcc file1.o file2.o -o prog` | Linka file oggetto |
| `gdb ./prog` | Debug con GDB |
| `valgrind ./prog` | Check memory con Valgrind |
| `make` | Compila con Makefile |
| `make clean` | Pulisci file compilati |

---

## Link Utili

- **GCC Manual:** https://gcc.gnu.org/onlinedocs/
- **GDB Manual:** https://sourceware.org/gdb/documentation/
- **Valgrind Manual:** https://valgrind.org/docs/manual/manual.html
- **Make Tutorial:** https://makefiletutorial.com/

---

[← Torna al Sommario](README.md)
