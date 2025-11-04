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

Compila ed esegui:
```bash
# Compila
gcc hello.c -o hello

# Esegui
./hello
```

### Opzioni di Compilazione Utili

```bash
# Con warning e debugging
gcc -Wall -Wextra -g hello.c -o hello

# Con ottimizzazione
gcc -O2 hello.c -o hello

# C standard specifico
gcc -std=c11 hello.c -o hello
```

## Prossimi Passi

Nel prossimo modulo, esploreremo i fondamenti del linguaggio: sintassi, tipi di dati, operatori e strutture di controllo.

---

[← Torna al Sommario](../README.md) | [Prossimo: Fondamenti →](../02_Fondamenti/README.md)
