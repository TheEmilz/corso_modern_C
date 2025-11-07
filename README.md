# Corso Moderno di Linguaggio C
## Da Principiante ad Avanzato

Benvenuti al corso completo di programmazione C, che copre dalla teoria di base fino alle tecniche avanzate, con confronti con linguaggi moderni e riferimenti alla programmazione a basso livello.

## 📚 Struttura del Corso

### 📘 Risorse Fondamentali
- **[Glossario Comandi](GLOSSARIO_COMANDI.md)** - Guida completa alla compilazione e ai comandi GCC
- **[Quick Reference](QUICK_REFERENCE.md)** - Riferimento rapido della sintassi C

### [00 - Preparazione: Algoritmi e Strutture Dati](00_Preparazione/)
**NUOVO! Modulo preparatorio da studiare prima del C**
- Pensiero computazionale e problem solving
- Analisi della complessità degli algoritmi
- Algoritmi di ricerca e ordinamento
- Strutture dati fondamentali (array, liste, stack, queue, alberi, hash table)
- Ricorsione e tecniche algoritmiche
- **Metodologia di debug sistematico**
- Esercizi progressivi con soluzioni guidate

### [01 - Introduzione](01_Introduzione/)
- Storia del linguaggio C
- Evoluzione e standard (K&R, ANSI C, C99, C11, C17, C23)
- Perché imparare C oggi
- Setup ambiente di sviluppo
- Spiegazione dettagliata del primo programma
- Debug con GDB passo-passo

### [02 - Fondamenti](02_Fondamenti/)
- Sintassi di base
- Tipi di dati e variabili con spiegazioni dettagliate
- Operatori aritmetici con esempi pratici
- Strutture di controllo annotate
- Funzioni con esempi completi
- Array e stringhe con diagrammi di memoria
- Esempi pratici completi e commentati
- Esercizi con soluzioni step-by-step

### [02.5 - Il Modello Mentale della Memoria](02.5_Modello_Memoria/)
**NUOVO! Modulo avanzato per sviluppare visualizzazione profonda della memoria**
- Stack vs Heap: anatomia completa con stack frames
- Allineamento e Padding: ottimizzazione struct e cache line
- Endianness: Little-Endian vs Big-Endian, rappresentazione binaria
- Segmenti di Memoria: .text, .data, .bss, .rodata, protezioni
- **Visualizzazioni ASCII dettagliate** di ogni concetto
- **Tools pratici**: gdb, valgrind, objdump, readelf
- **Obiettivo**: Sviluppare il "sesto senso della memoria"

### [03 - Livello Intermedio](03_Intermedio/)
- Puntatori e riferimenti
- Allocazione dinamica della memoria
- Strutture e union
- Typedef ed enum
- Preprocessore
- File I/O

### [03.5 - Undefined Behavior: Il Nemico Invisibile](03.5_Undefined_Behavior/)
**NUOVO! Guida completa per evitare comportamenti indefiniti**
- Catalogo completo: 10+ tipi di UB comuni in C
- Buffer Overflow, Use-After-Free, NULL Dereference
- Signed Integer Overflow, Uninitialized Variables
- Sequence Point Violations, Division by Zero
- Strict Aliasing, Data Races, e altro
- **Detection**: AddressSanitizer, UBSan, Valgrind, TSan
- **Prevenzione**: Best practices e pattern difensivi
- **Tools**: Compiler warnings, static analysis

### [04 - Livello Avanzato](04_Avanzato/)
- Puntatori avanzati (puntatori a funzioni, array di puntatori)
- Manipolazione bit
- Ottimizzazione del codice
- Gestione errori avanzata
- Multi-threading (pthreads)
- Gestione memoria avanzata
- **NUOVO! [Fil-C Compiler](04_Avanzato/fil_c_compiler.md)**: Memory safety, garbage collection, capabilities

### [05 - Assembly e Low-Level](05_Assembly_LowLevel/)
- Relazione tra C e Assembly
- Inline Assembly
- Architettura del processore
- Chiamate di sistema
- Performance e ottimizzazione

### [06 - Comparazioni con Altri Linguaggi](06_Comparazioni/)
- C vs Python
- C vs Java
- C vs Rust
- Quando usare C
- Vantaggi e svantaggi

### [07 - Mini Laboratori](07_Laboratori/)
- Lab 1: Sistema di gestione file
- Lab 2: Implementazione di strutture dati
- Lab 3: Network programming
- Lab 4: Sistema embedded simulato
- Lab 5: Parser e compiler basics

### [08 - Esercizi e Sfide](08_Esercizi/)
- Esercizi per principianti
- Esercizi intermedi
- Esercizi avanzati
- Progetti finali

### [09 - C e Database](09_Database/)
**NUOVO! Programmazione database con C**
- Integrazione SQLite in applicazioni C
- Operazioni CRUD (Create, Read, Update, Delete) complete
- Prepared statements e sicurezza SQL
- Transazioni e gestione errori robusta
- Query avanzate e JOIN
- Ottimizzazione e indici
- **Progetti completi**: Sistema biblioteca e gestionale negozio
- **Debug database**: Problemi comuni e soluzioni

## 🎯 Obiettivi del Corso

Alla fine di questo corso sarai in grado di:
- Scrivere programmi C efficienti e sicuri
- Comprendere la gestione della memoria a basso livello
- Leggere e scrivere codice Assembly basilare
- Scegliere il linguaggio appropriato per diversi scenari
- Debuggare problemi complessi
- Ottimizzare codice per performance

## 🛠 Requisiti

- Computer con Linux, macOS, o Windows (WSL)
- Compilatore GCC o Clang
- Editor di testo o IDE (VS Code, CLion, Vim)
- Conoscenze base di informatica

## 📖 Come Usare Questo Corso

### Percorso Consigliato per Principianti Assoluti

1. **Inizia dal Modulo 00** - Preparazione: Costruisci le fondamenta algoritmiche
2. **Passa al Modulo 01-02** - Impara la sintassi base del C
3. **Continua con Moduli 03-04** - Approfondisci concetti intermedi e avanzati
4. **Esplora Moduli 05-06** - Assembly e confronti con altri linguaggi
5. **Pratica con Moduli 07-08** - Laboratori ed esercizi
6. **Specializzati con Modulo 09** - Database e applicazioni reali

### Best Practices

1. Segui i moduli in ordine sequenziale
2. Compila ed esegui tutti gli esempi
3. **Fai debug attivo** - Non limitarti a leggere, sperimenta!
4. Completa gli esercizi di ogni sezione
5. Costruisci i progetti nei mini laboratori
6. Sperimenta e modifica il codice

## 📝 Licenza

MIT License - vedi [LICENSE](LICENSE) per dettagli

## 👥 Contribuire

Contributi, correzioni e suggerimenti sono benvenuti! Apri una issue o una pull request.

---

**Nota**: Questo è un corso completo e richiede dedizione. Prenditi il tempo necessario per comprendere ogni concetto prima di procedere.
