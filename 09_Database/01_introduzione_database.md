# Lezione 1: Introduzione ai Database con C

## Cos'è un Database?

Un **database** è una collezione organizzata di dati strutturati, gestita da un sistema che permette di:
- **Memorizzare** dati in modo persistente
- **Recuperare** dati efficientemente
- **Modificare** dati in modo sicuro
- **Gestire** grandi quantità di informazioni

### Analogia: Libreria vs File Sparsi

**Senza Database (file sparsi)**:
```
note_mario.txt
note_anna.txt
note_luigi.txt
...
```
- Difficile cercare informazioni
- Nessuna struttura comune
- Rischio di corruzione dati
- Nessun controllo accessi multipli

**Con Database (libreria organizzata)**:
```
Database: Rubrica
  Tabella: Contatti
    | ID | Nome   | Telefono    | Email           |
    |----|--------|-------------|-----------------|
    | 1  | Mario  | 123-456789  | mario@email.com |
    | 2  | Anna   | 987-654321  | anna@email.com  |
    | 3  | Luigi  | 555-123456  | luigi@email.com |
```
- Struttura uniforme
- Ricerca rapida
- Integrità garantita
- Accesso concorrente sicuro

## Tipi di Database

### Database Relazionali (SQL)

Organizzano i dati in **tabelle** con righe e colonne.

**Esempi**: MySQL, PostgreSQL, SQLite, Oracle

**Caratteristiche**:
- Relazioni tra tabelle (JOIN)
- Schema definito
- ACID compliance (Atomicity, Consistency, Isolation, Durability)
- Linguaggio SQL standard

### Database Non-Relazionali (NoSQL)

Organizzano i dati in formati diversi (documenti, chiave-valore, grafi).

**Esempi**: MongoDB, Redis, Cassandra

**Caratteristiche**:
- Flessibilità schema
- Scalabilità orizzontale
- Performance su operazioni specifiche

### Quale Scegliere?

Per questo corso usiamo **database relazionali** perché:
- ✅ Struttura chiara e prevedibile
- ✅ SQL è uno standard industriale
- ✅ Adatto alla maggior parte delle applicazioni
- ✅ Ottime garanzie di integrità

## Perché SQLite?

**SQLite** è un database relazionale **embedded** perfetto per C:

### Vantaggi di SQLite

1. **Zero Configuration**: Nessun server da configurare
2. **Single File**: Tutto il database in un file
3. **Lightweight**: Libreria piccola (~600KB)
4. **Fast**: Ottimizzato per performance locale
5. **Reliable**: Usato da miliardi di dispositivi
6. **Portable**: Funziona ovunque
7. **Public Domain**: Completamente libero

### Quando Usare SQLite

✅ **Usa SQLite per**:
- Applicazioni desktop
- App mobile (Android/iOS)
- Sistemi embedded
- Testing e prototyping
- File di configurazione strutturati
- Cache locale
- Applicazioni single-user

❌ **Non usare SQLite per**:
- Applicazioni web ad alto traffico
- Scritture concorrenti massive
- Database centralizzato per molti client
- Database > 1TB (limite teorico molto alto ma non ottimale)

Per questi casi, usa PostgreSQL, MySQL, o altre soluzioni server-based.

### SQLite vs Altri Database

| Feature | SQLite | MySQL/PostgreSQL |
|---------|--------|------------------|
| Setup | Nessuno | Server da configurare |
| File | Singolo file | Molti file di sistema |
| Concorrenza | Limitata (lock file) | Eccellente (row-level locks) |
| Networking | No (locale) | Sì (client-server) |
| Dimensione | ~600KB | ~200MB+ |
| Complessità | Minima | Media/Alta |
| Uso tipico | Embedded, mobile | Web apps, enterprise |

## Setup Ambiente di Sviluppo

### Passo 1: Installazione SQLite

**Linux (Ubuntu/Debian)**:
```bash
sudo apt update
sudo apt install sqlite3 libsqlite3-dev

# Verifica installazione
sqlite3 --version
# Output: 3.xx.x ...
```

**macOS**:
```bash
# SQLite è già installato
sqlite3 --version

# Se servono le librerie di sviluppo:
brew install sqlite3
```

**Windows (MSYS2/MinGW)**:
```bash
pacman -S mingw-w64-x86_64-sqlite3
```

### Passo 2: Verifica Compilazione

Crea un file di test `test_sqlite.c`:

```c
#include <stdio.h>
#include <sqlite3.h>

int main(void)
{
    printf("SQLite version: %s\n", sqlite3_libversion());
    return 0;
}
```

**Compila e esegui**:
```bash
gcc test_sqlite.c -o test_sqlite -lsqlite3
./test_sqlite

# Output atteso: SQLite version: 3.xx.x
```

**Note**:
- `-lsqlite3`: Link con la libreria SQLite
- Se errore "sqlite3.h not found": librerie dev non installate
- Se errore "undefined reference": manca `-lsqlite3`

### Passo 3: Tool CLI SQLite

SQLite include un tool interattivo:

```bash
# Apri database (crea se non esiste)
sqlite3 test.db

# Comandi utili:
.tables          # Lista tabelle
.schema          # Mostra struttura
.help            # Guida comandi
.quit            # Esci
```

**Esempio sessione**:
```sql
sqlite> CREATE TABLE test (id INTEGER, name TEXT);
sqlite> INSERT INTO test VALUES (1, 'Mario');
sqlite> SELECT * FROM test;
1|Mario
sqlite> .quit
```

## Primo Programma: Hello Database

### Codice Completo Commentato

```c
/*
 * hello_db.c - Primo programma con SQLite
 * Crea un database, una tabella e inserisce un record
 */

#include <stdio.h>
#include <sqlite3.h>

int main(void)
{
    /* Puntatore al database */
    sqlite3 *db;
    
    /* Puntatore per messaggi di errore */
    char *err_msg = NULL;
    
    /* Codice di ritorno */
    int rc;
    
    /* PASSO 1: Apri (o crea) il database */
    rc = sqlite3_open("hello.db", &db);
    
    /* Controllo errore apertura */
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Impossibile aprire database: %s\n", 
                sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }
    
    printf("Database aperto con successo\n");
    
    /* PASSO 2: Crea una tabella */
    const char *sql_create = 
        "CREATE TABLE IF NOT EXISTS Greetings("
        "id INTEGER PRIMARY KEY AUTOINCREMENT, "
        "message TEXT NOT NULL);";
    
    rc = sqlite3_exec(db, sql_create, NULL, NULL, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore SQL: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }
    
    printf("Tabella creata\n");
    
    /* PASSO 3: Inserisci un record */
    const char *sql_insert = 
        "INSERT INTO Greetings (message) VALUES ('Hello, Database!');";
    
    rc = sqlite3_exec(db, sql_insert, NULL, NULL, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore inserimento: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }
    
    printf("Record inserito con successo\n");
    
    /* PASSO 4: Chiudi il database */
    sqlite3_close(db);
    printf("Database chiuso\n");
    
    return 0;
}
```

### Spiegazione Dettagliata

#### Includi Header SQLite
```c
#include <sqlite3.h>
```
- Contiene tutte le dichiarazioni necessarie per SQLite
- Definisce tipi come `sqlite3`, funzioni come `sqlite3_open`

#### Dichiarazioni
```c
sqlite3 *db;
char *err_msg = NULL;
int rc;
```
- `sqlite3 *db`: Handle al database (puntatore opaco)
- `err_msg`: Per messaggi di errore allocati da SQLite
- `rc`: Return code (SQLITE_OK = 0 = successo)

#### Apertura Database
```c
rc = sqlite3_open("hello.db", &db);
```
- Primo parametro: nome file database
- Secondo parametro: puntatore al puntatore (output)
- Se il file non esiste, viene creato
- Ritorna `SQLITE_OK` (0) se successo

**Controllo Errore**:
```c
if (rc != SQLITE_OK) {
    fprintf(stderr, "Impossibile aprire database: %s\n", 
            sqlite3_errmsg(db));
    sqlite3_close(db);  // Chiudi anche in caso di errore!
    return 1;
}
```
- `sqlite3_errmsg(db)`: Ottiene descrizione errore
- **Importante**: Chiama sempre `sqlite3_close()` anche se open fallisce

#### Esecuzione SQL
```c
rc = sqlite3_exec(db, sql_create, NULL, NULL, &err_msg);
```

**Parametri**:
1. `db`: Handle database
2. `sql_create`: Stringa SQL da eseguire
3. `NULL`: Callback function (per query SELECT)
4. `NULL`: Primo argomento per callback
5. `&err_msg`: Output per errori

**SQL per Creare Tabella**:
```sql
CREATE TABLE IF NOT EXISTS Greetings(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    message TEXT NOT NULL
);
```
- `IF NOT EXISTS`: Non da errore se tabella esiste già
- `INTEGER PRIMARY KEY AUTOINCREMENT`: ID auto-incrementante
- `TEXT NOT NULL`: Campo obbligatorio

#### Liberazione Memoria Errori
```c
sqlite3_free(err_msg);
```
- `err_msg` è allocato da SQLite
- **Devi** liberarlo con `sqlite3_free()`, non `free()`

#### Chiusura Database
```c
sqlite3_close(db);
```
- Libera risorse
- Scrive buffer su disco
- **Sempre** necessario

### Compilazione ed Esecuzione

```bash
# Compila
gcc -Wall -Wextra -std=c11 hello_db.c -o hello_db -lsqlite3

# Esegui
./hello_db
```

**Output atteso**:
```
Database aperto con successo
Tabella creata
Record inserito con successo
Database chiuso
```

### Verifica con CLI

```bash
# Apri il database creato
sqlite3 hello.db

# Mostra tabelle
sqlite> .tables
Greetings

# Mostra struttura
sqlite> .schema Greetings
CREATE TABLE Greetings(
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  message TEXT NOT NULL
);

# Mostra dati
sqlite> SELECT * FROM Greetings;
1|Hello, Database!

sqlite> .quit
```

## Debug: Problemi Comuni di Setup

### Problema 1: Compilatore Non Trova Header

**Errore**:
```
fatal error: sqlite3.h: No such file or directory
```

**Soluzione**:
```bash
# Ubuntu/Debian
sudo apt install libsqlite3-dev

# macOS
brew install sqlite3

# Verifica path
find /usr -name "sqlite3.h" 2>/dev/null
```

### Problema 2: Linker Non Trova Libreria

**Errore**:
```
undefined reference to `sqlite3_open'
```

**Causa**: Manca flag `-lsqlite3`

**Soluzione**:
```bash
# SBAGLIATO
gcc hello_db.c -o hello_db

# CORRETTO
gcc hello_db.c -o hello_db -lsqlite3
```

### Problema 3: Database Locked

**Errore**:
```
database is locked
```

**Cause**:
- Altro processo ha il database aperto
- Transazione non completata
- File system read-only

**Debug**:
```bash
# Verifica chi sta usando il file
lsof hello.db

# Controlla permessi
ls -l hello.db

# Prova ad aprire con CLI
sqlite3 hello.db "SELECT 1;"
```

### Problema 4: Permission Denied

**Errore**:
```
unable to open database file
```

**Soluzione**:
```bash
# Verifica permessi directory
ls -ld .

# Assicurati di avere permessi write
chmod 755 .

# Verifica anche il file database se esiste
chmod 644 hello.db
```

### Problema 5: Memory Leak

**Problema**: Non liberi `err_msg`

**Debug con Valgrind**:
```bash
gcc -g hello_db.c -o hello_db -lsqlite3
valgrind --leak-check=full ./hello_db
```

**Fix**:
```c
if (rc != SQLITE_OK) {
    fprintf(stderr, "Errore: %s\n", err_msg);
    sqlite3_free(err_msg);  // NON dimenticare!
    // ...
}
```

## Concetti Chiave SQLite

### Handle al Database

```c
sqlite3 *db;
```
- Puntatore opaco (non accedi ai campi direttamente)
- Rappresenta una connessione al database
- Creato da `sqlite3_open()`
- Distrutto da `sqlite3_close()`

### Return Codes

SQLite ritorna codici per indicare successo/fallimento:

```c
SQLITE_OK      = 0   // Successo
SQLITE_ERROR   = 1   // Errore SQL generico
SQLITE_BUSY    = 5   // Database locked
SQLITE_LOCKED  = 6   // Tabella locked
SQLITE_NOMEM   = 7   // Memoria insufficiente
SQLITE_READONLY= 8   // Database read-only
// ... molti altri
```

**Best Practice**: Controlla sempre il return code!

### Messaggi di Errore

```c
// Ottieni messaggio dell'ultimo errore
const char *msg = sqlite3_errmsg(db);

// Ottieni codice errore esteso
int extended_code = sqlite3_extended_errcode(db);
```

### Thread Safety

SQLite può essere:
- **Single-thread**: Non thread-safe
- **Multi-thread**: Thread-safe con limitazioni
- **Serialized**: Completamente thread-safe (default)

Verifica modalità:
```c
int mode = sqlite3_threadsafe();
// 0 = single-thread
// 1 = serialized
// 2 = multi-thread
```

## Esercizi Pratici

### Esercizio 1: Modifica il Programma

Modifica `hello_db.c` per:
1. Inserire 3 messaggi diversi
2. Stampare un messaggio dopo ogni inserimento
3. Gestire gli errori per ogni operazione

### Esercizio 2: Crea Tabella Personalizzata

Crea un programma che:
1. Crea una tabella `Students` con campi: id, name, age
2. Inserisce 3 studenti
3. Stampa messaggio di conferma

**Schema suggerito**:
```sql
CREATE TABLE Students(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    age INTEGER CHECK(age > 0 AND age < 150)
);
```

### Esercizio 3: Gestione Errori Robusta

Implementa una funzione helper per eseguire SQL con gestione errori:

```c
int execute_sql(sqlite3 *db, const char *sql, const char *operation)
{
    char *err_msg = NULL;
    int rc = sqlite3_exec(db, sql, NULL, NULL, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore in %s: %s\n", operation, err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    printf("%s completata\n", operation);
    return 0;
}
```

Usa questa funzione per semplificare il codice principale.

## Punti Chiave

✅ SQLite è perfetto per applicazioni C embedded
✅ Usa `sqlite3_open()` per aprire/creare database
✅ Usa `sqlite3_exec()` per eseguire SQL
✅ **Sempre** controlla i return code
✅ **Sempre** libera `err_msg` con `sqlite3_free()`
✅ **Sempre** chiudi il database con `sqlite3_close()`
✅ Il flag `-lsqlite3` è necessario per la compilazione

## Prossimi Passi

Nel prossimo modulo esploreremo:
- Come leggere i risultati delle query SELECT
- Uso di callback per processare dati
- Query parametrizzate
- Gestione più robusta degli errori

---

[← Torna al Modulo](README.md) | [Prossimo: SQLite Basics →](02_sqlite_basics.md)
