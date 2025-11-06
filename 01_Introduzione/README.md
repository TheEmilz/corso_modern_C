# Modulo 01 - Introduzione al Linguaggio C

## Storia del Linguaggio C

### Le Origini (1969-1973)

Il linguaggio C è stato sviluppato da **Dennis Ritchie** ai Bell Laboratories tra il 1969 e il 1973. La sua creazione è strettamente legata allo sviluppo del sistema operativo UNIX.

#### Timeline Storica:
- **1969**: Ken Thompson crea il linguaggio B (predecessore di C)
- **1971**: Dennis Ritchie inizia a sviluppare C come evoluzione di B
- **1972**: Prima implementazione stabile di C
- **1973**: UNIX viene riscritto in C (un evento rivoluzionario!)

### Perché C è Stato Rivoluzionario?

Prima di C, i sistemi operativi erano scritti in Assembly. C ha permesso:
- **Portabilità**: codice più facile da trasferire tra diverse architetture
- **Astrazione**: livello intermedio tra linguaggi ad alto livello e Assembly
- **Efficienza**: prestazioni vicine all'Assembly con maggiore leggibilità

## Evoluzione degli Standard

### K&R C (1978)
Brian Kernighan e Dennis Ritchie pubblicano "The C Programming Language", definendo il primo standard de facto.

```c
/* Stile K&R - dichiarazioni implicite */
main()
{
    printf("Hello, World!\n");
}
```

### ANSI C / C89 (1989)
Primo standard ufficiale ANSI (American National Standards Institute).

**Caratteristiche principali:**
- Prototipi di funzione
- Tipo `void`
- Standard library definitiva

```c
#include <stdio.h>

int main(void)
{
    printf("Hello, World!\n");
    return 0;
}
```

### C99 (1999)
Standard ISO che introduce molte nuove feature:
- Tipo `long long`
- Tipo `_Bool`
- Array di lunghezza variabile (VLA)
- Commenti in stile C++ (`//`)
- `inline` keyword
- Funzioni matematiche estese

```c
#include <stdio.h>
#include <stdbool.h>

int main(void)
{
    bool flag = true;  // Tipo booleano
    // Commento stile C++
    long long big_number = 9223372036854775807LL;
    printf("Big number: %lld\n", big_number);
    return 0;
}
```

### C11 (2011)
Introduce supporto per multi-threading e sicurezza:
- Thread support (`threads.h`)
- Atomic operations
- Bounds checking functions
- `_Generic` keyword
- `static_assert`

```c
#include <stdio.h>
#include <threads.h>

int main(void)
{
    static_assert(sizeof(int) >= 4, "int deve essere almeno 4 bytes");
    printf("Assertion passed!\n");
    return 0;
}
```

### C17 (2018)
Principalmente correzioni di bug e chiarificazioni, nessuna nuova feature.

### C23 (2023)
L'ultimo standard include:
- `nullptr` constant
- Miglioramenti a `_BitInt(N)`
- `typeof` operator
- Attributi migliorati

## Perché Imparare C nel 2025?

### 1. **Fondamento di Altri Linguaggi**
Molti linguaggi moderni sono influenzati da C:
- C++, C#, Java (sintassi)
- Python (implementato in C - CPython)
- Ruby, Perl, PHP (influenzati da C)

### 2. **Comprensione del Computer**
C ti insegna:
- Come funziona la memoria
- Come interagire con l'hardware
- Cosa succede "sotto il cofano"

### 3. **Performance Critiche**
C è ancora usato per:
- Sistemi operativi (Linux, Windows)
- Database (PostgreSQL, MySQL)
- Linguaggi di programmazione (interpreti, VM)
- Sistemi embedded
- Driver hardware
- Game engines

### 4. **Codice Legacy**
Miliardi di righe di codice C esistente richiedono manutenzione e sviluppo.

## Setup Ambiente di Sviluppo

### Linux (Ubuntu/Debian)
```bash
# Installa GCC e strumenti di sviluppo
sudo apt update
sudo apt install build-essential gdb

# Verifica installazione
gcc --version
gdb --version
```

### macOS
```bash
# Installa Xcode Command Line Tools
xcode-select --install

# Oppure usa Homebrew per GCC
brew install gcc

# Verifica
gcc --version
```

### Windows
**Opzione 1: WSL (Windows Subsystem for Linux)**
```bash
# Installa WSL, poi segui istruzioni Linux
wsl --install
```

**Opzione 2: MinGW-w64**
- Scarica da: https://www.mingw-w64.org/
- Aggiungi al PATH di sistema

**Opzione 3: Visual Studio**
- Installa Visual Studio Community
- Seleziona "Desktop development with C++"

### Editor e IDE Consigliati

1. **Visual Studio Code** (gratuito, leggero)
   - Estensioni: C/C++, Code Runner
   
2. **CLion** (JetBrains, a pagamento ma con trial)
   - IDE completo con debugging avanzato

3. **Vim/Neovim** (per puristi)
   - Con plugin LSP per autocompletamento

4. **Code::Blocks** (gratuito, IDE completo)

### Il Tuo Primo Programma

Crea un file `hello.c`:

```c
#include <stdio.h>

int main(void)
{
    printf("Hello, World! Benvenuto nel corso di C!\n");
    return 0;
}
```

#### Spiegazione Dettagliata Linea per Linea

**Linea 1: `#include <stdio.h>`**
- `#include`: Direttiva del preprocessore che dice di includere un file
- `<stdio.h>`: File header della libreria standard che contiene:
  - Dichiarazioni delle funzioni di Input/Output
  - `printf()` - per stampare a schermo
  - `scanf()` - per leggere input
  - `fopen()`, `fclose()` - per gestire file
- Le parentesi angolate `< >` indicano che è un header di sistema
- Senza questa linea, il compilatore non saprebbe cosa fare quando incontra `printf()`

**Linea 3: `int main(void)`**
- `int`: Tipo di ritorno della funzione - un numero intero
- `main`: Nome speciale - è il punto di ingresso del programma
- `(void)`: Parametri della funzione - `void` significa "nessun parametro"
- Ogni programma C **deve** avere una funzione `main()`
- Quando esegui il programma, il sistema operativo chiama `main()`

**Linea 4: `{`**
- Parentesi graffa di apertura
- Segna l'inizio del corpo della funzione
- Tutto il codice della funzione va tra `{` e `}`

**Linea 5: `printf("Hello, World! Benvenuto nel corso di C!\n");`**
- `printf`: Funzione che stampa testo sullo schermo (console)
- `"..."`: Stringa letterale - testo tra virgolette
- `\n`: Carattere speciale di "newline" (vai a capo)
- `;`: Punto e virgola - obbligatorio alla fine di ogni istruzione C
- Il testo viene inviato allo "standard output" (di solito il terminale)

**Linea 6: `return 0;`**
- `return`: Restituisce un valore e termina la funzione
- `0`: Valore restituito - per convenzione significa "successo"
- Il valore viene passato al sistema operativo
- Valori diversi da 0 indicano errori (es. `return 1` per errore generico)
- Puoi vedere il valore di ritorno nel terminale con `echo $?` (Linux/Mac)

**Linea 7: `}`**
- Parentesi graffa di chiusura
- Segna la fine del corpo della funzione

#### Compilazione ed Esecuzione - Passo Passo

**Passo 1: Crea il file sorgente**
```bash
# Usa un editor di testo (nano, vim, VSCode, ecc.)
nano hello.c
# Scrivi il codice sopra e salva
```

**Passo 2: Compila il programma**
```bash
gcc hello.c -o hello
```

Cosa succede internamente:
1. **Preprocessing**: Il preprocessore sostituisce `#include <stdio.h>` con il contenuto del file header
2. **Compilation**: Il compilatore traduce il codice C in Assembly
3. **Assembly**: L'assembler converte Assembly in codice macchina (file oggetto)
4. **Linking**: Il linker collega il tuo codice con le librerie (come `libc` per `printf`)
5. **Output**: Viene creato il file eseguibile `hello`

**Passo 3: Esegui il programma**
```bash
./hello
```

Output atteso:
```
Hello, World! Benvenuto nel corso di C!
```

Cosa succede:
1. Il sistema operativo carica l'eseguibile in memoria
2. Viene chiamata la funzione `main()`
3. `printf()` scrive sulla console
4. Il programma termina e restituisce 0 al sistema operativo

**Passo 4: Verifica il codice di uscita**
```bash
echo $?
```

Output:
```
0
```

Questo conferma che il programma è terminato con successo!

### Esempio Avanzato con Spiegazione Dettagliata

Creiamo un programma più complesso per esplorare altri concetti:

```c
/* 
 * Programma: calcolatrice semplice
 * Autore: Studente C
 * Data: 2025
 */

#include <stdio.h>   // Per printf e scanf
#include <stdlib.h>  // Per EXIT_SUCCESS e EXIT_FAILURE

/* Costante globale - valore che non cambia mai */
#define VERSIONE "1.0"

/* Funzione che somma due numeri */
int somma(int a, int b)
{
    int risultato = a + b;
    return risultato;
}

int main(void)
{
    /* Dichiarazione variabili */
    int numero1;
    int numero2;
    int risultato;
    
    /* Stampa intestazione */
    printf("=== Calcolatrice Semplice v%s ===\n", VERSIONE);
    printf("\n");
    
    /* Input: leggi primo numero */
    printf("Inserisci il primo numero: ");
    scanf("%d", &numero1);
    
    /* Input: leggi secondo numero */
    printf("Inserisci il secondo numero: ");
    scanf("%d", &numero2);
    
    /* Elaborazione: calcola la somma */
    risultato = somma(numero1, numero2);
    
    /* Output: mostra il risultato */
    printf("\n");
    printf("Risultato: %d + %d = %d\n", numero1, numero2, risultato);
    
    /* Termina con successo */
    return EXIT_SUCCESS;
}
```

#### Spiegazione Dettagliata del Programma Avanzato

**Commenti:**
```c
/* Questo è un commento multi-linea */
// Questo è un commento su singola linea (C99+)
```
- I commenti vengono ignorati dal compilatore
- Servono per documentare il codice
- `/* ... */` può estendersi su più righe
- `//` commenta fino alla fine della linea

**Header multipli:**
```c
#include <stdio.h>   // Input/Output standard
#include <stdlib.h>  // Funzioni di utilità generale
```
- `<stdlib.h>` include:
  - `EXIT_SUCCESS` (valore 0)
  - `EXIT_FAILURE` (valore 1)
  - Funzioni come `malloc()`, `exit()`, `atoi()`

**Macro costante:**
```c
#define VERSIONE "1.0"
```
- `#define`: Crea una macro (sostituzione testuale)
- `VERSIONE`: Nome della macro (per convenzione in MAIUSCOLO)
- `"1.0"`: Valore che verrà sostituito ovunque appaia `VERSIONE`
- Durante il preprocessing, ogni occorrenza di `VERSIONE` diventa `"1.0"`

**Funzione personalizzata:**
```c
int somma(int a, int b)
{
    int risultato = a + b;
    return risultato;
}
```
- `int somma`: Dichiara una funzione che restituisce un `int`
- `(int a, int b)`: Parametri - due numeri interi
- I parametri `a` e `b` sono copie locali (passaggio per valore)
- `risultato`: Variabile locale alla funzione
- `return risultato`: Restituisce il valore al chiamante

**Dichiarazione variabili:**
```c
int numero1;
int numero2;
int risultato;
```
- Ogni variabile deve essere dichiarata prima dell'uso
- `int`: Tipo di dato - numero intero a 32 bit
- Le variabili non inizializzate contengono valori casuali (garbage)
- Buona pratica: inizializza sempre, es. `int numero1 = 0;`

**Printf con placeholder:**
```c
printf("=== Calcolatrice Semplice v%s ===\n", VERSIONE);
```
- `%s`: Placeholder per una stringa
- `VERSIONE`: Valore che sostituisce `%s`
- Durante l'esecuzione: `%s` → `"1.0"`

**Scanf per input:**
```c
scanf("%d", &numero1);
```
- `scanf`: Legge input dalla tastiera
- `%d`: Format specifier per un intero decimale
- `&numero1`: L'operatore `&` passa l'**indirizzo** della variabile
- **Perché `&`?** `scanf` deve modificare la variabile, quindi serve sapere dove si trova in memoria
- L'utente digita un numero e preme Invio

**Chiamata di funzione:**
```c
risultato = somma(numero1, numero2);
```
- Chiama la funzione `somma` con i valori di `numero1` e `numero2`
- Il valore restituito viene assegnato a `risultato`
- Flusso:
  1. Copia i valori di `numero1` e `numero2` in `a` e `b`
  2. Esegue il codice della funzione
  3. Restituisce il risultato
  4. Assegna il valore a `risultato`

**Printf con placeholder multipli:**
```c
printf("Risultato: %d + %d = %d\n", numero1, numero2, risultato);
```
- Primo `%d` → `numero1`
- Secondo `%d` → `numero2`
- Terzo `%d` → `risultato`
- L'ordine dei placeholder corrisponde all'ordine degli argomenti

**Return con costante:**
```c
return EXIT_SUCCESS;
```
- `EXIT_SUCCESS`: Macro definita in `<stdlib.h>`, vale 0
- Più leggibile di `return 0`
- Alternativa: `EXIT_FAILURE` (errore)

#### Compilazione ed Esecuzione del Programma Avanzato

```bash
# Compila con warning e standard C11
gcc -Wall -Wextra -std=c11 calcolatrice.c -o calcolatrice

# Esegui
./calcolatrice
```

**Sessione di esempio:**
```
=== Calcolatrice Semplice v1.0 ===

Inserisci il primo numero: 15
Inserisci il secondo numero: 27

Risultato: 15 + 27 = 42
```

**Verifica codice di uscita:**
```bash
echo $?
# Output: 0 (successo)
```

### Opzioni di Compilazione Utili

#### Compilazione con Warning Completi
```bash
gcc -Wall -Wextra -g hello.c -o hello
```

**Spiegazione opzioni:**
- `-Wall`: Abilita tutti i warning comuni
  - Variabili non usate
  - Funzioni senza return
  - Conversioni pericolose
- `-Wextra`: Warning aggiuntivi
  - Confronti signed/unsigned
  - Parametri non usati
- `-g`: Include informazioni di debug
  - Permette di usare GDB
  - Include nomi di variabili e numeri di linea
  - L'eseguibile sarà più grande

#### Compilazione con Ottimizzazione
```bash
gcc -O2 hello.c -o hello
```

**Spiegazione:**
- `-O2`: Livello di ottimizzazione 2
  - Codice più veloce
  - Binario potenzialmente più piccolo
  - Compilazione più lenta
  - Utile per versioni di produzione

**Livelli di ottimizzazione:**
- `-O0`: Nessuna ottimizzazione (default)
- `-O1`: Ottimizzazioni base
- `-O2`: Ottimizzazioni standard (consigliato per release)
- `-O3`: Ottimizzazioni aggressive
- `-Os`: Ottimizza per dimensione

#### Specificare lo Standard C
```bash
gcc -std=c11 hello.c -o hello
```

**Standard disponibili:**
- `-std=c89` o `-std=c90`: ANSI C (vecchio)
- `-std=c99`: C99 (comune)
- `-std=c11`: C11 (consigliato)
- `-std=c17`: C17 (correzioni di bug di C11)
- `-std=c23`: C23 (più recente, richiede GCC 13+)

**Perché specificare lo standard?**
- Garantisce portabilità del codice
- Evita feature non standard
- Previene problemi con compilatori diversi

#### Compilazione Completa per Sviluppo
```bash
gcc -Wall -Wextra -Wpedantic -std=c11 -g hello.c -o hello
```

Questa è la combinazione ideale per lo sviluppo:
- Tutti i warning abilitati
- Standard C11
- Informazioni di debug

#### Compilazione per Produzione
```bash
gcc -Wall -Wextra -std=c11 -O2 -DNDEBUG hello.c -o hello
```

**Differenze:**
- `-O2`: Ottimizzazione per performance
- `-DNDEBUG`: Disabilita le asserzioni (macro `assert`)
- Niente `-g`: Nessuna info di debug (binario più piccolo)

### Debug del Primo Programma

Creiamo un programma con un piccolo bug da debuggare:

```c
#include <stdio.h>

int main(void)
{
    int x = 10;
    int y = 0;
    int risultato = x / y;  // INTENTIONAL BUG: divisione per zero (per dimostrazione debug)!
    
    printf("Risultato: %d\n", risultato);
    return 0;
}
```

#### Compilazione con Debug
```bash
gcc -g -Wall bug.c -o bug
```

#### Esecuzione e Crash
```bash
./bug
# Floating point exception (core dumped)
```

#### Debug con GDB

**Avvia GDB:**
```bash
gdb ./bug
```

**Comandi GDB:**
```gdb
(gdb) break main
Breakpoint 1 at 0x1149: file bug.c, line 5.

(gdb) run
Starting program: ./bug

Breakpoint 1, main () at bug.c:5
5           int x = 10;

(gdb) next
6           int y = 0;

(gdb) next
7           int risultato = x / y;

(gdb) print x
$1 = 10

(gdb) print y
$2 = 0

(gdb) print y == 0
$3 = 1

(gdb) quit
```

**Scoperta:** `y` è 0, causando la divisione per zero!

**Correzione:**
```c
#include <stdio.h>

int main(void)
{
    int x = 10;
    int y = 0;
    
    if (y == 0) {
        printf("Errore: divisione per zero!\n");
        return 1;
    }
    
    int risultato = x / y;
    printf("Risultato: %d\n", risultato);
    return 0;
}
```

Per una guida completa ai comandi di compilazione, consulta il [Glossario Comandi](../GLOSSARIO_COMANDI.md).

## Prossimi Passi

Nel prossimo modulo, esploreremo i fondamenti del linguaggio: sintassi, tipi di dati, operatori e strutture di controllo.

---

[← Torna al Sommario](../README.md) | [Prossimo: Fondamenti →](../02_Fondamenti/README.md)
