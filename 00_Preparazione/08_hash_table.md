# Lezione 8 - Strutture Dati: Hash Table

## 🎯 Obiettivi della Lezione

Alla fine di questa lezione sarai in grado di:
- Comprendere il concetto di hashing e funzioni hash
- Visualizzare come funzionano le hash table
- Gestire le collisioni (chaining e open addressing)
- Analizzare la complessità delle operazioni
- Riconoscere quando usare hash table invece di array o BST

---

## 1. PROBLEMA: Ricerca Veloce con Chiavi Arbitrarie

### Scenario: Database Studenti

```
Hai 10000 studenti identificati da matricola (es: "ST2024001").
Devi cercare uno studente data la matricola.

SOLUZIONI POSSIBILI:

1. ARRAY:
   studenti[0] = "ST2024001"
   studenti[1] = "ST2024002"
   ...
   Ricerca: O(n) ⚠️ LENTO!

2. ARRAY ORDINATO + RICERCA BINARIA:
   Ricerca: O(log n) ✓ Meglio
   Inserimento: O(n) ⚠️ (shift elementi)

3. BST:
   Ricerca/Inserimento: O(log n) ✓
   Ma se possiamo fare meglio?

DOMANDA: Possiamo avere ricerca/inserimento O(1)?
```

**RISPOSTA:** SÌ! Con le **Hash Table**!

---

## 2. CONCETTO DI HASHING

### 2.1 L'Idea Fondamentale

**Hashing**: Trasformare una chiave arbitraria in un indice di array.

```
FUNZIONE HASH:
┌──────────────┐
│  "ST2024001" │ ─┐
└──────────────┘  │
                  ├─→ hash() ─→ 42
┌──────────────┐  │
│  "ST2024999" │ ─┘
└──────────────┘
        ↓
    Indice array

ARRAY (Hash Table):
┌─────┬─────┬─────┬────────┬─────┐
│  0  │  1  │ ... │   42   │ ... │
└─────┴─────┴─────┴────────┴─────┘
                      ↑
           Dati studente "ST2024001"

COMPLESSITÀ: O(1) ⚡ (accesso diretto!)
```

### 2.2 Proprietà di una Buona Funzione Hash

```
UNA FUNZIONE HASH IDEALE:

1. DETERMINISTICA
   hash("ST2024001") → 42 (sempre lo stesso risultato)

2. UNIFORME
   Distribuisce le chiavi uniformemente negli indici
   (evita troppi elementi nello stesso indice)

3. VELOCE
   Calcolo in O(1)

4. MINIMIZZA COLLISIONI
   Chiavi diverse → indici diversi (idealmente)
```

---

## 3. FUNZIONI HASH COMUNI

### 3.1 Metodo della Divisione

```
FORMULA: hash(chiave) = chiave % dimensione_tabella

ESEMPIO:
Tabella con 10 slot:
hash(23) = 23 % 10 = 3
hash(45) = 45 % 10 = 5
hash(123) = 123 % 10 = 3  ← COLLISIONE con 23!

PRO:
✓ Semplice e veloce

CONTRO:
✗ Dimensione tabella influenza distribuzione
   (meglio usare numeri primi)

BEST PRACTICE: dimensione_tabella = numero primo
```

### 3.2 Metodo della Moltiplicazione

```
FORMULA: hash(chiave) = floor(m × (chiave × A mod 1))
dove:
- m = dimensione tabella
- A = costante tra 0 e 1 (spesso 0.6180339... = (√5 - 1)/2)

ESEMPIO:
m = 100, A = 0.618
chiave = 123
hash(123) = floor(100 × (123 × 0.618 mod 1))
          = floor(100 × (76.014 mod 1))
          = floor(100 × 0.014)
          = 1

PRO:
✓ Funziona bene con qualsiasi m
✓ Distribuzione uniforme
```

### 3.3 Hash per Stringhe

```
METODO SEMPLICE (somma caratteri):
hash("abc") = ('a' + 'b' + 'c') % m
            = (97 + 98 + 99) % m
            = 294 % m

PROBLEMA: Anagrammi hanno stesso hash!
hash("abc") == hash("bca") ✗

METODO MIGLIORE (polinomiale):
hash(s) = (s[0] × p^0 + s[1] × p^1 + s[2] × p^2 + ...) % m

ESEMPIO con p = 31, m = 101:
hash("abc") = (97×31^0 + 98×31^1 + 99×31^2) % 101
            = (97×1 + 98×31 + 99×961) % 101
            = (97 + 3038 + 95139) % 101
            = 98274 % 101
            = 27

PRO:
✓ Ordine caratteri conta
✓ Buona distribuzione
```

---

## 4. COLLISIONI

### 4.1 Cos'è una Collisione?

```
COLLISIONE: Due chiavi diverse producono lo stesso hash.

ESEMPIO:
hash("ST2024001") = 42
hash("ST2024999") = 42  ← COLLISIONE!

┌─────────────────────────┐
│ Array[42] = ???         │ ← Chi ci va?
└─────────────────────────┘

INEVITABILI! (Pigeonhole Principle)
Se hai più chiavi che slot, DEVE esserci collisione.

SOLUZIONE: Metodi di risoluzione collisioni
```

### 4.2 Chaining (Concatenamento)

**IDEA:** Ogni slot contiene una **lista** di elementi.

```
HASH TABLE con CHAINING:

hash("Alice") = 3
hash("Bob") = 1
hash("Charlie") = 3  ← COLLISIONE con Alice!
hash("David") = 3    ← COLLISIONE!

┌───┬──────────────────────────┐
│ 0 │ → NULL                   │
├───┼──────────────────────────┤
│ 1 │ → [Bob] → NULL           │
├───┼──────────────────────────┤
│ 2 │ → NULL                   │
├───┼──────────────────────────┤
│ 3 │ → [Alice] → [Charlie] → [David] → NULL
├───┼──────────────────────────┤
│ 4 │ → NULL                   │
└───┴──────────────────────────┘
      ↑
   Liste concatenate!

OPERAZIONI:

INSERIMENTO:
1. Calcola hash(chiave)
2. Aggiungi nodo alla lista in table[hash]
Complessità: O(1) se inserisci in testa

RICERCA:
1. Calcola hash(chiave)
2. Cerca nella lista table[hash]
Complessità:
  - Caso migliore: O(1) - lista vuota o primo elemento
  - Caso peggiore: O(n) - tutti nella stessa lista!
  - Caso medio: O(1 + α) dove α = n/m (load factor)

LOAD FACTOR (α):
α = n/m
n = numero elementi
m = numero slot

ESEMPIO: 100 elementi, 10 slot → α = 10
Ogni lista ha mediamente 10 elementi → O(10) per ricerca!

BEST PRACTICE: Mantieni α < 1 (rehashing se supera)
```

### 4.3 Open Addressing (Indirizzamento Aperto)

**IDEA:** Tutte le chiavi sono nell'array (nessuna lista). Se uno slot è occupato, cerca il prossimo disponibile.

#### Linear Probing (Scansione Lineare)

```
STRATEGIA: Se slot i è occupato, prova i+1, i+2, i+3, ...

ESEMPIO:
Inserisci: 23, 43, 13, 27 (con hash(x) = x % 10)

hash(23) = 3 → table[3] = 23
┌───┬───┬───┬────┬───┐
│ 0 │ 1 │ 2 │ 23 │ 4 │
└───┴───┴───┴────┴───┘

hash(43) = 3 → OCCUPATO! Prova 4 → table[4] = 43
┌───┬───┬───┬────┬────┐
│ 0 │ 1 │ 2 │ 23 │ 43 │
└───┴───┴───┴────┴────┘

hash(13) = 3 → OCCUPATO! Prova 4 → OCCUPATO! Prova 5 → table[5] = 13
┌───┬───┬───┬────┬────┬────┐
│ 0 │ 1 │ 2 │ 23 │ 43 │ 13 │
└───┴───┴───┴────┴────┴────┘

hash(27) = 7 → table[7] = 27
┌───┬───┬───┬────┬────┬────┬───┬────┐
│ 0 │ 1 │ 2 │ 23 │ 43 │ 13 │ 6 │ 27 │
└───┴───┴───┴────┴────┴────┴───┴────┘

RICERCA di 13:
1. hash(13) = 3 → table[3] = 23 ≠ 13
2. Prova 4 → table[4] = 43 ≠ 13
3. Prova 5 → table[5] = 13 ✓ TROVATO!

PROBLEMA: PRIMARY CLUSTERING
┌───┬───┬───┬────┬────┬────┬────┬────┬───┐
│ 0 │ 1 │ 2 │ 23 │ 43 │ 13 │ 53 │ 27 │ 8 │
└───┴───┴───┴────┴────┴────┴────┴────┴───┘
              ↑─── CLUSTER ────↑

Lunghe sequenze contigue rallentano ricerca!
```

#### Quadratic Probing

```
STRATEGIA: Se slot i è occupato, prova i+1², i+2², i+3², ...

ESEMPIO:
hash(23) = 3 → OCCUPATO
Prova: 3 + 1² = 4
       3 + 2² = 7
       3 + 3² = 12 (mod m)
       ...

PRO: Riduce primary clustering
CONTRO: Secondary clustering (chiavi con stesso hash seguono stessa sequenza)
```

#### Double Hashing

```
STRATEGIA: Usa seconda funzione hash per calcolare il passo.

FORMULA:
hash1(k) = k % m
hash2(k) = 1 + (k % (m-1))

Se slot hash1(k) occupato:
Prova: hash1(k) + 1×hash2(k)
       hash1(k) + 2×hash2(k)
       hash1(k) + 3×hash2(k)
       ...

PRO: Migliore distribuzione, riduce clustering
```

---

## 5. REHASHING

### 5.1 Il Problema del Load Factor Alto

```
PROBLEMA: Quando α = n/m cresce, performance degradano.

ESEMPIO: m = 10, n = 15 → α = 1.5
Con chaining: liste mediamente lunghe 1.5
Con open addressing: molte collisioni

SOLUZIONE: REHASHING
Quando α supera soglia (es: 0.7):
1. Crea nuova tabella con m' = 2×m (o prossimo primo)
2. Riinserisci TUTTI gli elementi nella nuova tabella
3. Deallocca vecchia tabella
```

### 5.2 Procedura di Rehashing

```
PRIMA: m = 5, n = 4, α = 0.8
┌───┬──────┬───┬──────┬───┐
│ 0 │ [10] │ 2 │ [23] │ 4 │
│   │ [15] │   │      │   │
└───┴──────┴───┴──────┴───┘

REHASHING (m' = 11):

1. Crea nuova tabella vuota (dimensione 11)
2. Per ogni elemento nella vecchia tabella:
   - Ricalcola hash con m' = 11
   - Inserisci nella nuova tabella

DOPO: m = 11, n = 4, α = 0.36
┌───┬────┬───┬───┬───┬─────┬───┬───┬───┬───┬────┐
│ 0 │ 10 │ 2 │ 3 │ 4 │ 15  │ 6 │ 7 │ 8 │ 9 │ 23 │
└───┴────┴───┴───┴───┴─────┴───┴───┴───┴───┴────┘

α ridotto! Performance migliorate!

COSTO: O(n) per rehashing
Ma AMMORTIZZATO: O(1) per inserimento (raro evento)
```

---

## 6. CONFRONTO: Array vs BST vs Hash Table

```
┌─────────────────┬─────────────┬──────────────┬─────────────────┐
│   OPERAZIONE    │    ARRAY    │     BST      │   HASH TABLE    │
├─────────────────┼─────────────┼──────────────┼─────────────────┤
│ Ricerca         │ O(n)        │ O(log n)     │ O(1) medio ⚡   │
│ Inserimento     │ O(1) append │ O(log n)     │ O(1) medio ⚡   │
│ Rimozione       │ O(n)        │ O(log n)     │ O(1) medio ⚡   │
│ Min/Max         │ O(n)        │ O(log n)     │ O(n)            │
│ Ordinamento     │ O(n log n)  │ O(n) in-order│ Non supportato  │
│ Range query     │ O(n)        │ O(log n + k) │ Non efficiente  │
│ Memoria         │ Compatta    │ + puntatori  │ + spazio extra  │
└─────────────────┴─────────────┴──────────────┴─────────────────┘

🎯 USA HASH TABLE QUANDO:
  ✅ Ricerche/inserimenti/rimozioni frequenti
  ✅ Chiavi arbitrarie (non solo interi)
  ✅ NON serve ordinamento
  ✅ NON serve range query (trova tutti tra x e y)

🎯 USA BST QUANDO:
  ✅ Serve ordinamento
  ✅ Range query frequenti
  ✅ Min/Max frequenti

🎯 USA ARRAY QUANDO:
  ✅ Accesso per indice
  ✅ Dimensione piccola/fissa
  ✅ Poche operazioni dinamiche
```

---

## 7. ⚠️ ERRORI MORTALI

### ERRORE #1: Funzione Hash Scadente

```
// ❌ DISASTRO: Hash che ignora la maggior parte della chiave

int hash(const char *str, int m) {
    return str[0] % m;  // USA SOLO IL PRIMO CARATTERE!
}

/* PROBLEMA:
 * hash("Alice") = 'A' % m
 * hash("Anna") = 'A' % m
 * hash("Andrew") = 'A' % m
 * 
 * TUTTE LE STRINGHE CON 'A' → STESSA SLOT!
 * Tantissime collisioni → performance O(n)!
 */

// ✅ SOLUZIONE: Usa tutta la chiave
unsigned long hash(const char *str, int m) {
    unsigned long hash = 5381;
    int c;
    while ((c = *str++))
        hash = ((hash << 5) + hash) + c; // hash * 33 + c
    return hash % m;
}
```

### ERRORE #2: Dimensione Tabella Non Prima

```
// ❌ PROBLEMA: Dimensione = potenza di 2

int m = 16;  // 2^4
hash(k) = k % 16;

/* CON MODULO DI POTENZA DI 2:
 * Solo gli ultimi 4 bit di k contano!
 * k = ...xxxx (binario)
 * k % 16 usa solo xxxx
 * 
 * DISTRIBUZIONE NON UNIFORME se chiavi hanno pattern
 */

// ✅ MEGLIO: Usa numero primo
int m = 17;  // Primo
hash(k) = k % 17;

/* NUMERI PRIMI garantiscono migliore distribuzione
 * specialmente con metodo della divisione
 */
```

### ERRORE #3: Non Gestire Collisioni

```
// ❌ DISASTRO: Sovrascrive elementi

void insert(int key, int value) {
    int index = hash(key);
    table[index] = value;  // SOVRASCRIVE se già occupato!
}

/* INSERIMENTI:
 * insert(23, 100) → table[3] = 100
 * insert(43, 200) → hash(43) = 3
 *                → table[3] = 200  ← PERSO 100!
 */

// ✅ SOLUZIONE: Implementa risoluzione collisioni
void insert(int key, int value) {
    int index = hash(key);
    
    // Linear probing
    while (table[index].occupied) {
        index = (index + 1) % m;
    }
    
    table[index].key = key;
    table[index].value = value;
    table[index].occupied = true;
}
```

### ERRORE #4: Memory Leak con Chaining

```
// ❌ DISASTRO: Non libera le liste

void destroy_table(HashTable *ht) {
    free(ht->table);  // Libera array
    free(ht);         // Libera struttura
    // MA LE LISTE? ← LEAK!
}

/* MEMORIA:
 * ┌─ table[3] ────┐
 * │ → [A] → [B] → NULL  ← NON LIBERATI! LEAK!
 * └───────────────┘
 */

// ✅ SOLUZIONE: Libera ogni lista
void destroy_table(HashTable *ht) {
    for (int i = 0; i < ht->size; i++) {
        Node *current = ht->table[i];
        while (current != NULL) {
            Node *next = current->next;
            free(current);
            current = next;
        }
    }
    free(ht->table);
    free(ht);
}
```

---

## 8. 🔧 ESERCIZI

### 🟢 LIVELLO 1 - Comprensione

**Esercizio 1:** Calcola hash usando divisione (m = 7):
- hash(15)
- hash(22)
- hash(49)

<details>
<summary>Soluzione</summary>

```
hash(15) = 15 % 7 = 1
hash(22) = 22 % 7 = 1  ← COLLISIONE!
hash(49) = 49 % 7 = 0
```
</details>

**Esercizio 2:** Con linear probing, inserisci 15, 22, 8 (hash(x) = x % 7):

<details>
<summary>Soluzione</summary>

```
Tabella inizialmente vuota (size = 7):

Insert 15:
hash(15) = 1 → table[1] = 15
[_][15][_][_][_][_][_]

Insert 22:
hash(22) = 1 → OCCUPATO! Prova 2 → table[2] = 22
[_][15][22][_][_][_][_]

Insert 8:
hash(8) = 1 → OCCUPATO! Prova 2 → OCCUPATO! Prova 3 → table[3] = 8
[_][15][22][8][_][_][_]
```
</details>

### 🟡 LIVELLO 2 - Applicazione

**Esercizio 3:** Scrivi pseudocodice per ricerca con chaining.

<details>
<summary>Soluzione</summary>

```
FUNZIONE ricerca_chaining(table, key):
    index = hash(key) % table.size
    current = table[index]  // Prima nodo della lista
    
    MENTRE current != NULL:
        SE current.key == key:
            RETURN current.value  // Trovato!
        current = current.next
    
    RETURN NULL  // Non trovato

COMPLESSITÀ:
- Caso migliore: O(1) - primo elemento o lista vuota
- Caso peggiore: O(n) - tutti nella stessa lista
- Caso medio: O(1 + α) dove α = load factor
```
</details>

### 🟠 LIVELLO 3 - Analisi

**Esercizio 4:** Perché il rehashing ha complessità ammortizzata O(1)?

<details>
<summary>Soluzione</summary>

```
ANALISI AMMORTIZZATA:

Supponiamo di iniziare con m = 2 e raddoppiare quando α > 0.7.

Sequenza inserimenti:
1-2: Inserimenti normali → 2 operazioni
3-4: Rehash (m=4), poi inserimenti → 2 + 4 = 6 operazioni
5-8: Inserimenti normali → 4 operazioni
9-16: Rehash (m=8), poi inserimenti → 8 + 8 = 16 operazioni

TOTALE per 16 elementi:
2 + 6 + 4 + 16 = 28 operazioni

COSTO MEDIO: 28 / 16 = 1.75 ≈ O(1)

INTUIZIONE:
Anche se rehashing costa O(n), accade raramente.
La maggior parte degli inserimenti costa O(1).
Mediato su molte operazioni → O(1) ammortizzato.
```
</details>

### 🔴 LIVELLO 4 - Sintesi

**Esercizio 5:** Implementa una funzione hash per stringhe che minimizzi collisioni.

<details>
<summary>Soluzione</summary>

```
FUNZIONE hash_string(str, m):
    hash = 0
    p = 31  // Numero primo
    p_pow = 1
    
    PER ogni carattere c in str:
        // Converti c in valore numerico (es: 'a' = 1, 'b' = 2, ...)
        char_value = c - 'a' + 1
        
        // Aggiungi contributo pesato per posizione
        hash = (hash + char_value * p_pow) % m
        
        // Aggiorna potenza per prossima posizione
        p_pow = (p_pow * p) % m
    
    RETURN hash

ESEMPIO: hash_string("abc", 101) con p = 31

c = 'a': char_value = 1
         hash = (0 + 1 × 1) % 101 = 1
         p_pow = (1 × 31) % 101 = 31

c = 'b': char_value = 2
         hash = (1 + 2 × 31) % 101 = 63
         p_pow = (31 × 31) % 101 = 961 % 101 = 52

c = 'c': char_value = 3
         hash = (63 + 3 × 52) % 101 = 219 % 101 = 17
         p_pow = (52 × 31) % 101 = ...

RETURN 17

VANTAGGI:
✓ Usa tutti i caratteri
✓ Posizione conta (no anagrammi con stesso hash)
✓ Distribuzione uniforme
✓ Modulo previene overflow
```
</details>

---

## 9. 🎓 PUNTI CHIAVE

**HASH TABLE:**
- ✅ Ricerca/Inserimento/Rimozione: O(1) medio ⚡
- ✅ Chiavi arbitrarie (stringhe, oggetti, ...)
- ⚠️ NON ordinato, NO range query
- ⚠️ Performance dipende da funzione hash e load factor

**FUNZIONE HASH:**
- Deve essere deterministica, veloce, uniforme
- Minimizza collisioni
- Numeri primi per dimensione tabella

**COLLISIONI:**
- Chaining: Liste concatenate per slot
- Open Addressing: Cerca prossimo slot libero
- Rehashing: Ridimensiona quando α > soglia

**LOAD FACTOR (α):**
- α = n / m
- Mantieni α < 1 (meglio < 0.7)
- Rehashing quando supera soglia

---

## 10. DEBUG: Problemi Comuni

### Problema 1: Troppe Collisioni

```
SINTOMO: Performance O(n) invece di O(1)

CAUSA:
1. Funzione hash scadente
2. Load factor troppo alto
3. Dimensione tabella non ottimale

DEBUG:
1. Stampa distribuzione: quanti elementi per slot?
2. Verifica load factor: α = n/m
3. Testa funzione hash con dati reali
4. Rehash se α > 0.7
```

### Problema 2: Infinite Loop in Open Addressing

```
SINTOMO: Programma si blocca durante inserimento

CAUSA: Tabella piena, linear probing non trova slot libero

SOLUZIONE:
1. Controlla se tabella è piena prima di inserire
2. Implementa rehashing automatico
3. Limita numero tentativi (failsafe)
```

---

## 📚 Prossimi Passi

Ora che comprendi le hash table, sei pronto per:
- [Lezione 9: Ricorsione](09_ricorsione.md) - Padroneggiare tecniche ricorsive
- [Lezione 10: Debugging](10_debugging.md) - Metodologie sistematiche

---

[← Alberi](07_alberi.md) | [Torna al Modulo](README.md) | [Ricorsione →](09_ricorsione.md)
