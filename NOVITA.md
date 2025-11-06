# 🆕 Novità - Moduli Preparatori e Database

## Sommario delle Novità

Questo documento descrive i nuovi moduli aggiunti al corso che seguono una **didattica progressiva per livelli** con focus sul **debugging sistematico**.

## 📦 Modulo 00 - Preparazione: Algoritmi e Strutture Dati

### Perché Questo Modulo?

Questo modulo **preparatorio** va studiato **prima** di iniziare il C. L'obiettivo è:
- ✅ Costruire solide basi algoritmiche
- ✅ Imparare a ragionare in modo computazionale
- ✅ Acquisire tecniche di debugging sistematico
- ✅ Separare la logica algoritmica dalla sintassi del linguaggio

### Contenuti Completi

#### 1. **Pensiero Computazionale** (Lezione 1)
- Cos'è un algoritmo e come costruirlo
- Decomposizione di problemi complessi
- Riconoscimento di pattern
- Astrazione e generalizzazione
- **Debug**: Come ragionare su un problema prima di scrivere codice

#### 2. **Complessità degli Algoritmi** (Lezione 2)
- Notazione Big-O completa (O(1), O(log N), O(N), O(N²), O(2^N))
- Analisi tempo e spazio
- Casi migliore, medio, peggiore
- Confronto tra algoritmi
- **Debug**: Identificare colli di bottiglia nelle performance

#### 3. **Algoritmi di Ricerca** (Lezione 3)
- Ricerca lineare con tracciamento completo
- Ricerca binaria con visualizzazioni
- Confronto performance
- Varianti (prima/ultima occorrenza)
- **Debug passo-passo**: Tracciare l'esecuzione degli algoritmi

#### 4. **Algoritmi di Ordinamento** (Lezione 4)
- Bubble Sort (semplice ma O(N²))
- Selection Sort (pochi scambi)
- Insertion Sort (ottimo per array quasi ordinati)
- Merge Sort (sempre O(N log N), stabile)
- Quick Sort (veloce in media, in-place)
- Confronto completo e quando usare ognuno
- **Debug**: Trovare errori negli algoritmi di ordinamento

#### 10. **Metodologia di Debug** (Lezione 10)
- Processo sistematico di debugging in 5 fasi
- Tecniche: print debugging, asserzioni, rubber duck
- Errori comuni e come evitarli
- Caso di studio completo
- Strumenti mentali per problem solving
- **15 pagine** di metodologie professionali

#### 11. **Esercizi Progressivi** (Lezione 11)
- **4 livelli di difficoltà**: Principiante → Intermedio → Avanzato → Esperto
- Oltre **15 esercizi** con soluzioni complete
- Tracciamenti passo-passo
- Progetti completi (calcolatrice, scheduling)
- Auto-valutazione e checklist

### Approccio Didattico

Il modulo segue **4 livelli progressivi**:

```
Livello 1: Concetti Base
  ↓ Esempi semplici e visualizzazioni
  
Livello 2: Applicazione Pratica
  ↓ Pseudocodice e casi concreti
  
Livello 3: Approfondimento
  ↓ Strutture complesse e ottimizzazione
  
Livello 4: Maestria
  ↓ Metodologie professionali e autonomia
```

## 💾 Modulo 09 - C e Database

### Perché Database in C?

- **Performance**: Accesso diretto senza overhead
- **Controllo**: Gestione fine di memoria e risorse
- **Portabilità**: SQLite funziona ovunque
- **Embedded Systems**: Perfetto per dispositivi con risorse limitate
- **Applicazioni Reali**: Browser, mobile apps, gaming

### Contenuti Completi

#### 1. **Introduzione Database** (Lezione 1)
- Cos'è un database e perché usarlo
- Database relazionali vs non-relazionali
- Perché SQLite è perfetto per C
- Setup completo dell'ambiente
- Primo programma con database funzionante
- **Debug**: Problemi comuni di setup (5+ scenari)

#### 3. **Operazioni CRUD Complete** (Lezione 3)
- **CREATE**: Inserimento singolo e multiplo con transazioni
- **READ**: Callback per elaborare risultati, query con filtri
- **UPDATE**: Aggiornamento singolo e multiplo con condizioni
- **DELETE**: Eliminazione sicura con conferma
- Programma CRUD completo interattivo (200+ righe commentate)
- **Debug**: SQL injection, memory leaks, gestione errori

### Struttura Modulo Completo

Il modulo include 10 lezioni pianificate:

1. ✅ Introduzione Database (completa - 13k caratteri)
2. 📝 SQLite Basics e Callback
3. ✅ CRUD Operations (completa - 21k caratteri)
4. 📝 Prepared Statements e Sicurezza
5. 📝 Transazioni e Gestione Errori
6. 📝 Query Avanzate e JOIN
7. 📝 Indici e Ottimizzazione
8. 📝 Progetto: Sistema Biblioteca
9. 📝 Progetto: Gestionale Negozio
10. 📝 Best Practices e Debug Database

### Esempi Pratici Inclusi

**Primo Programma Database** (hello_db.c):
```c
// Crea database, tabella, inserisce record
// Gestione errori completa
// Chiusura sicura risorse
```

**Sistema CRUD Completo** (crud_complete.c):
```c
// Menu interattivo
// Operazioni CREATE, READ, UPDATE, DELETE
// Gestione input utente
// Visualizzazione risultati formattati
```

## 📚 Percorso di Studio Consigliato

### Per Principianti Assoluti

```
1. Modulo 00 - Preparazione
   ↓ Costruisci fondamenta algoritmiche (3-5 giorni)
   
2. Modulo 01 - Introduzione C
   ↓ Impara sintassi base (3-4 giorni)
   
3. Modulo 02 - Fondamenti C
   ↓ Tipi, operatori, strutture controllo (1 settimana)
   
4. Modulo 03-04 - Intermedio/Avanzato
   ↓ Puntatori, memoria, strutture (2 settimane)
   
5. Modulo 09 - Database
   ↓ Applicazioni pratiche (1 settimana)
   
6. Moduli 05-08
   ↓ Assembly, laboratori, esercizi
```

### Per Studenti con Esperienza

Se conosci già un linguaggio di programmazione:

```
1. Modulo 00 (Lezioni 2, 3, 4, 10, 11)
   ↓ Ripasso algoritmi e debug (1-2 giorni)
   
2. Modulo 01-02 - Veloce
   ↓ Sintassi C (2-3 giorni)
   
3. Modulo 03-04
   ↓ Puntatori e memoria (1 settimana)
   
4. Modulo 09 - Database
   ↓ Integrazione SQLite (3-4 giorni)
   
5. Progetti personalizzati
```

## 🎯 Obiettivi di Apprendimento

Dopo aver completato questi moduli, sarai in grado di:

### Dal Modulo 00
- ✅ Analizzare complessità algoritmi (Big-O)
- ✅ Implementare ricerca e ordinamento efficienti
- ✅ Debuggare codice sistematicamente
- ✅ Risolvere problemi algoritmici complessi
- ✅ Scegliere strutture dati appropriate

### Dal Modulo 09
- ✅ Integrare SQLite in applicazioni C
- ✅ Eseguire operazioni CRUD complete
- ✅ Gestire errori e transazioni
- ✅ Scrivere query sicure (prepared statements)
- ✅ Ottimizzare database per performance
- ✅ Costruire applicazioni data-driven reali

## 📊 Statistiche

### Modulo 00 - Preparazione
- **Lezioni completate**: 6 su 11
- **Pagine di contenuto**: ~40 pagine
- **Esempi pratici**: 50+
- **Esercizi con soluzioni**: 15+
- **Progetti completi**: 2

### Modulo 09 - Database
- **Lezioni completate**: 2 su 10
- **Pagine di contenuto**: ~25 pagine
- **Esempi di codice**: 20+
- **Programmi completi**: 3
- **Scenari di debug**: 10+

### Totale Novità
- **Caratteri totali**: ~100,000
- **Righe di codice esempio**: 500+
- **Ore di studio stimate**: 15-20 ore

## 🔍 Caratteristiche Uniche

### Focus sul Debug

Ogni lezione include:
- ✅ Sezione dedicata al debugging
- ✅ Errori comuni con soluzioni
- ✅ Tracciamenti passo-passo
- ✅ Tecniche professionali

### Approccio Progressivo

- ✅ Difficoltà crescente graduata
- ✅ Molti esempi visivi
- ✅ Spiegazioni approfondite
- ✅ Esercizi con soluzioni complete

### Praticità

- ✅ Codice compilabile e testabile
- ✅ Progetti realistici
- ✅ Best practices industriali
- ✅ Gestione errori robusta

## 💡 Come Utilizzare i Nuovi Moduli

### Modulo 00
1. **Leggi sequenzialmente** le lezioni 1-4, 10-11
2. **Fai gli esercizi** progressivi (Lezione 11)
3. **Traccia algoritmi** su carta prima di implementare
4. **Non usare ancora C** - focus su logica

### Modulo 09
1. **Setup ambiente** (Lezione 1)
2. **Compila ed esegui** tutti gli esempi
3. **Modifica il codice** per sperimentare
4. **Costruisci progetti** personali

## 🚀 Prossimi Sviluppi

### Modulo 00 - Da Completare
- [ ] Lezione 5: Array e Liste
- [ ] Lezione 6: Stack e Queue
- [ ] Lezione 7: Alberi
- [ ] Lezione 8: Hash Table
- [ ] Lezione 9: Ricorsione

### Modulo 09 - Da Completare
- [ ] Lezione 2: SQLite Basics
- [ ] Lezione 4: Prepared Statements
- [ ] Lezione 5: Transazioni
- [ ] Lezione 6-7: Query Avanzate e Ottimizzazione
- [ ] Lezioni 8-9: Progetti Completi

## 📝 Note Importanti

### Prerequisiti Modulo 00
- ✅ Nessuno! Adatto a principianti assoluti
- ✅ Conoscenza base di matematica
- ✅ Voglia di imparare e sperimentare

### Prerequisiti Modulo 09
- ✅ Modulo 02 completato (puntatori, stringhe)
- ✅ Modulo 03 completato (struct, allocazione dinamica)
- ✅ Confidenza con compilazione C

### Compilazione Esempi Database

```bash
# Installa SQLite
sudo apt install sqlite3 libsqlite3-dev  # Linux
brew install sqlite3                      # macOS

# Compila programmi
gcc -Wall -Wextra -std=c11 program.c -o program -lsqlite3

# Nota il flag -lsqlite3 alla fine!
```

## 🎓 Filosofia Didattica

Questi moduli seguono il principio:

> **"Impara a pensare algoritmicamente prima di imparare un linguaggio"**

### Perché?

Molti studenti faticano con la programmazione perché cercano di imparare contemporaneamente:
1. Sintassi del linguaggio
2. Concetti di programmazione
3. Algoritmi e strutture dati
4. Pensiero computazionale
5. Debugging

**La nostra soluzione**: Separa queste competenze!

- **Modulo 00**: Pensiero algoritmico (senza sintassi)
- **Moduli 01-04**: Sintassi e concetti C
- **Modulo 09**: Applicazioni pratiche

## 📞 Supporto e Feedback

Se hai domande, suggerimenti o trovi errori:
- Apri una Issue su GitHub
- Contribuisci con Pull Request
- Consulta il [CONTRIBUTING.md](CONTRIBUTING.md)

## 🏆 Risultati Attesi

Dopo questi moduli, sarai capace di:

1. **Pensare Algoritmicamente**
   - Decomporre problemi complessi
   - Analizzare complessità
   - Ottimizzare soluzioni

2. **Debuggare Efficacemente**
   - Approccio sistematico
   - Identificare bug rapidamente
   - Prevenire errori comuni

3. **Costruire Applicazioni Reali**
   - Integrare database
   - Gestire dati persistenti
   - Scrivere codice production-ready

## 📅 Data Aggiornamento

Ultimo aggiornamento: Novembre 2025

## 🌟 Conclusione

Questi nuovi moduli rappresentano **oltre 50 pagine** di contenuto educativo di alta qualità, progettato per:

- ✅ Costruire basi solide
- ✅ Insegnare metodologie professionali
- ✅ Preparare a progetti reali
- ✅ Sviluppare autonomia nel problem solving

**Buono studio!** 🚀

---

[Torna al README Principale](README.md) | [Modulo 00](00_Preparazione/README.md) | [Modulo 09](09_Database/README.md)
