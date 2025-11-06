# Modulo 09 - C e Database

## Introduzione

Questo modulo ti insegna come integrare database nelle tue applicazioni C, con focus particolare su SQLite, un database leggero, embedded e perfetto per applicazioni C. Imparerai a gestire dati persistenti, eseguire query complesse e costruire applicazioni data-driven robuste.

## 🎯 Obiettivi del Modulo

Alla fine di questo modulo sarai in grado di:
- Integrare SQLite in applicazioni C
- Eseguire operazioni CRUD (Create, Read, Update, Delete)
- Gestire transazioni e errori in modo sicuro
- Ottimizzare query per performance
- Debuggare problemi con i database
- Costruire applicazioni reali con persistenza dei dati

## 📚 Struttura del Modulo

### [1. Introduzione ai Database con C](01_introduzione_database.md)
- Cos'è un database e perché usarlo
- Database relazionali vs non relazionali
- Perché SQLite è perfetto per C
- Setup dell'ambiente di sviluppo
- Primo programma con database
- **Debug**: Problemi comuni di setup

### [2. SQLite Basics: Connessione e Query Semplici](02_sqlite_basics.md)
- Apertura e chiusura connessione
- Esecuzione di query semplici
- Gestione errori SQLite
- Lettura dei risultati
- **Debug passo-passo**: Tracciare le query SQL

### [3. Operazioni CRUD Complete](03_crud_operations.md)
- CREATE: Creare tabelle e inserire dati
- READ: Leggere e filtrare dati
- UPDATE: Aggiornare record esistenti
- DELETE: Rimuovere dati
- Esempi pratici per ogni operazione
- **Debug**: Errori comuni nelle operazioni CRUD

### [4. Prepared Statements e SQL Injection](04_prepared_statements.md)
- Cos'è l'SQL injection e perché è pericoloso
- Prepared statements per sicurezza
- Binding di parametri
- Best practices di sicurezza
- **Debug**: Identificare vulnerabilità SQL

### [5. Transazioni e Gestione Errori](05_transactions.md)
- Cos'è una transazione
- BEGIN, COMMIT, ROLLBACK
- Gestione errori robusta
- Atomicità delle operazioni
- **Debug**: Problemi con transazioni incomplete

### [6. Query Avanzate e JOIN](06_advanced_queries.md)
- SELECT con WHERE, ORDER BY, LIMIT
- Aggregazioni: COUNT, SUM, AVG, MAX, MIN
- JOIN: INNER, LEFT, RIGHT
- Subquery e query nidificate
- **Debug**: Ottimizzare query lente

### [7. Indici e Ottimizzazione](07_indexes_optimization.md)
- Cos'è un indice e come funziona
- Quando creare indici
- EXPLAIN QUERY PLAN
- Ottimizzazione delle performance
- **Debug**: Identificare colli di bottiglia

### [8. Progetto Pratico: Sistema di Gestione Biblioteca](08_progetto_biblioteca.md)
- Design del database (schema)
- Implementazione completa
- Interfaccia utente testuale
- Gestione libri, utenti, prestiti
- **Debug guidato**: Risolvere problemi reali

### [9. Progetto Pratico: Sistema Gestionale Negozio](09_progetto_negozio.md)
- Database per e-commerce
- Gestione prodotti, ordini, clienti
- Calcolo inventario e statistiche
- Report e query complesse
- **Debug avanzato**: Performance e concorrenza

### [10. Best Practices e Debugging Database](10_best_practices_debug.md)
- Pattern di design per database in C
- Gestione della memoria
- Thread safety
- Logging e debugging
- Errori comuni e soluzioni
- Tools per debugging SQL

## 💡 Approccio Didattico Progressivo

Questo modulo segue la stessa filosofia didattica del corso:

### Livello 1: Fondamenti (Lezioni 1-3)
- Concetti base di database
- Setup e prima applicazione
- Operazioni CRUD semplici
- Esempi minimali e chiari

### Livello 2: Sicurezza e Robustezza (Lezioni 4-5)
- Prepared statements
- Gestione errori
- Transazioni
- Codice production-ready

### Livello 3: Avanzato (Lezioni 6-7)
- Query complesse
- JOIN e aggregazioni
- Ottimizzazione
- Performance tuning

### Livello 4: Maestria (Lezioni 8-10)
- Progetti completi
- Applicazioni reali
- Best practices professionali
- Debug avanzato

## 🔧 Prerequisiti

Prima di iniziare questo modulo, assicurati di aver completato:
- ✅ Modulo 02: Fondamenti (puntatori, stringhe, strutture)
- ✅ Modulo 03: Intermedio (struct, allocazione dinamica, file I/O)
- ✅ Conoscenze base di SQL (fornite nel modulo)

## 📦 Setup Richiesto

### Installazione SQLite

**Linux (Ubuntu/Debian)**:
```bash
sudo apt update
sudo apt install sqlite3 libsqlite3-dev
```

**macOS**:
```bash
# SQLite è pre-installato
# Per le librerie di sviluppo:
brew install sqlite3
```

**Windows**:
```bash
# Con MSYS2/MinGW:
pacman -S mingw-w64-x86_64-sqlite3

# Oppure scarica da: https://www.sqlite.org/download.html
```

### Verifica Installazione

```bash
# Verifica SQLite
sqlite3 --version

# Verifica librerie (Linux)
pkg-config --libs sqlite3
```

### Compilazione con SQLite

```bash
# Compila un programma con SQLite
gcc -Wall -Wextra -std=c11 program.c -o program -lsqlite3

# Con debug info
gcc -g -Wall -Wextra -std=c11 program.c -o program -lsqlite3
```

## 📝 Esempio Introduttivo

Per dare un'idea di cosa imparerai, ecco un esempio completo minimale:

```c
#include <stdio.h>
#include <sqlite3.h>

int main(void)
{
    sqlite3 *db;
    char *err_msg = NULL;
    
    /* Apri (o crea) database */
    int rc = sqlite3_open("test.db", &db);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore apertura: %s\n", sqlite3_errmsg(db));
        return 1;
    }
    
    /* Crea tabella */
    const char *sql = "CREATE TABLE IF NOT EXISTS Users("
                      "id INTEGER PRIMARY KEY, "
                      "name TEXT NOT NULL, "
                      "age INTEGER);";
    
    rc = sqlite3_exec(db, sql, NULL, NULL, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore SQL: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }
    
    /* Inserisci dati */
    sql = "INSERT INTO Users (name, age) VALUES ('Mario', 30);";
    rc = sqlite3_exec(db, sql, NULL, NULL, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore inserimento: %s\n", err_msg);
        sqlite3_free(err_msg);
    }
    
    printf("Database creato e popolato con successo!\n");
    
    /* Chiudi database */
    sqlite3_close(db);
    
    return 0;
}
```

**Compilazione**:
```bash
gcc esempio.c -o esempio -lsqlite3
./esempio
```

Questo è solo l'inizio! Nel corso del modulo imparerai a:
- Leggere e visualizzare i dati
- Usare query parametrizzate (sicure)
- Gestire errori in modo robusto
- Costruire applicazioni complete

## 🎓 Perché Database in C?

### Vantaggi dell'Integrazione Database in C

1. **Performance**: Accesso diretto ai dati senza overhead di linguaggi interpretati
2. **Controllo**: Gestione fine della memoria e delle risorse
3. **Portabilità**: SQLite funziona ovunque, nessuna configurazione server
4. **Embedded Systems**: Perfetto per dispositivi con risorse limitate
5. **Legacy Code**: Molti sistemi esistenti sono in C

### Applicazioni Reali

- **Sistemi Embedded**: Router, IoT devices
- **Mobile Apps**: SQLite è usato in Android e iOS
- **Desktop Applications**: Browser (Firefox, Chrome), Email clients
- **Gaming**: Salvataggio stati di gioco
- **Sistemi Operativi**: Configurazioni di sistema

## 🔍 Focus sul Debug

Ogni lezione include sezioni dedicate al debugging:

### Problemi Comuni Trattati

1. **Errori di Connessione**: Database locked, file permissions
2. **SQL Syntax**: Errori nelle query
3. **Memory Leaks**: Mancata liberazione risorse SQLite
4. **Race Conditions**: Accesso concorrente
5. **Data Corruption**: Transazioni incomplete
6. **Performance Issues**: Query non ottimizzate

### Strumenti di Debug

- **sqlite3 CLI**: Tool interattivo per esplorare database
- **EXPLAIN QUERY PLAN**: Analizzare esecuzione query
- **Valgrind**: Trovare memory leaks
- **GDB**: Debugger step-by-step
- **Logging**: Tracciare tutte le operazioni SQL

## 📊 Progressione delle Competenze

```
Lezione 1-2:  Connetti e esegui query basilari
              ↓
Lezione 3-4:  CRUD completo e sicurezza
              ↓
Lezione 5-6:  Gestione errori e query avanzate
              ↓
Lezione 7:    Ottimizzazione performance
              ↓
Lezione 8-9:  Progetti completi realistici
              ↓
Lezione 10:   Professionalizzazione e best practices
```

## 💪 Cosa Costruirai

### Progetto 1: Sistema Biblioteca (Lezione 8)
- Database con tabelle multiple
- Relazioni tra entità
- CRUD completo
- Ricerca e filtri
- Report statistiche

### Progetto 2: Gestionale Negozio (Lezione 9)
- E-commerce completo
- Gestione inventario
- Ordini e clienti
- Calcoli aggregati
- Dashboard analytics

## 🚀 Inizia il Viaggio

Questo modulo ti porterà da zero a costruire applicazioni C professionali con persistenza dei dati. Ogni concetto è spiegato con:

- ✅ Teoria chiara e concisa
- ✅ Esempi pratici commentati
- ✅ Esercizi progressivi
- ✅ Debug guidato passo-passo
- ✅ Best practices industriali
- ✅ Codice production-ready

## Nota Importante

Questo modulo assume che tu abbia familiarità con:
- Puntatori e gestione memoria in C
- Strutture (`struct`)
- Allocazione dinamica (`malloc`, `free`)
- Gestione stringhe
- File I/O basilare

Se questi concetti non sono chiari, ripassa i moduli precedenti prima di continuare.

## Prossimi Passi

Inizia con la [Lezione 1: Introduzione ai Database](01_introduzione_database.md) per comprendere i concetti fondamentali prima di scrivere codice.

---

[Torna al Sommario Principale](../README.md) | [Inizia: Introduzione Database →](01_introduzione_database.md)
