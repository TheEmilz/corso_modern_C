# Lezione 5 - Strutture Dati: Array e Liste

## 🎯 Obiettivi della Lezione

Alla fine di questa lezione sarai in grado di:
- Comprendere la differenza tra array statici e dinamici
- Visualizzare come array e liste sono organizzati in memoria
- Scegliere tra array e liste in base al caso d'uso
- Riconoscere i trade-off in termini di performance e memoria
- Debuggare problemi comuni con array e liste

---

## 1. PROBLEMA: Perché Servono le Strutture Dati?

Immagina di dover gestire una lista di 1000 studenti. Potresti creare 1000 variabili separate?

```
studente1 = "Mario Rossi"
studente2 = "Giulia Bianchi"
studente3 = "Luca Verdi"
...
studente1000 = "Sara Neri"
```

**Problemi:**
- ❌ Impossibile da gestire
- ❌ Non puoi iterare facilmente
- ❌ Non puoi aggiungere/rimuovere studenti dinamicamente

**SOLUZIONE**: Usare strutture dati che raggruppano elementi correlati!

---

## 2. ARRAY: La Struttura Dati Fondamentale

### 2.1 Cos'è un Array?

Un **array** è una sequenza di elementi dello stesso tipo, memorizzati in **posizioni di memoria contigue**.

### 2.2 Visualizzazione in Memoria

```
ARRAY: int numeri[5] = {10, 20, 30, 40, 50};

MEMORIA (indirizzi crescenti →):
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│ numeri[0]│ numeri[1]│ numeri[2]│ numeri[3]│ numeri[4]│
├──────────┼──────────┼──────────┼──────────┼──────────┤
│    10    │    20    │    30    │    40    │    50    │
├──────────┼──────────┼──────────┼──────────┼──────────┤
│ 0x1000   │ 0x1004   │ 0x1008   │ 0x100C   │ 0x1010   │
└──────────┴──────────┴──────────┴──────────┴──────────┘
         ↑            ↑
         +4 bytes     +4 bytes (ogni int = 4 byte)

CARATTERISTICHE CHIAVE:
✓ Memoria CONTIGUA (un blocco unico)
✓ Accesso DIRETTO tramite indice: numeri[2] → 0x1000 + (2 × 4) = 0x1008
✓ Dimensione FISSA (5 elementi)
```

### 2.3 Accesso agli Elementi: O(1) ⚡

**Formula magica:**
```
Indirizzo(array[i]) = Indirizzo_Base + (i × Dimensione_Elemento)
```

**Esempio:**
```
numeri[3] si trova a:
  0x1000 + (3 × 4 byte) = 0x100C
  
Tempo: O(1) - ISTANTANEO!
Non importa se cerchi l'elemento 0 o l'elemento 1000, 
il calcolo è sempre lo stesso.
```

### 2.4 Array Statici vs Dinamici

**ARRAY STATICO** (dimensione fissa, nota a compile-time):
```
int arr_statico[100];  // 100 elementi, memoria sullo stack

MEMORIA:
┌─ Stack ─────────────────────┐
│ arr_statico: [0][1]...[99]  │ ← 100 × 4 = 400 byte allocati
└─────────────────────────────┘

PRO:
✓ Veloce (già allocato)
✓ Nessun overhead di gestione
✓ Deallocazione automatica

CONTRO:
✗ Dimensione fissa
✗ Rischio stack overflow se troppo grande
```

**ARRAY DINAMICO** (dimensione decisa a runtime):
```
int *arr_dinamico = malloc(n * sizeof(int));  // n determinato a runtime

MEMORIA:
┌─ Stack ──────────────────┐
│ arr_dinamico: [puntatore]│ ──┐
└──────────────────────────┘   │
                               │
┌─ Heap ───────────────────────▼───┐
│ [0][1][2]...[n-1]                │ ← n × 4 byte allocati
└──────────────────────────────────┘

PRO:
✓ Dimensione flessibile
✓ Può essere molto grande (heap > stack)

CONTRO:
✗ Richiede malloc/free (gestione manuale)
✗ Overhead di allocazione
✗ Rischio memory leak se non liberato
```

---

## 3. LISTE CONCATENATE (Linked Lists)

### 3.1 Il Problema degli Array

**Scenario:** Hai un array di 1000 elementi ordinati. Devi inserire un nuovo elemento all'inizio.

```
ARRAY: [1][2][3][4]...[1000]
       ↑
       Inserisci 0 qui

OPERAZIONE: Devi spostare TUTTI i 1000 elementi!
[1][2][3]... → [ ][1][2][3]...
                ↑
                0

Complessità: O(n) ⚠️ LENTO!
```

**SOLUZIONE: Lista Concatenata!**

### 3.2 Struttura della Lista Concatenata

Ogni elemento (nodo) contiene:
1. **Dato**: Il valore memorizzato
2. **Puntatore**: Indirizzo del prossimo nodo

```
NODO:
┌──────────┬──────────┐
│  Dato    │  Next    │
│   10     │   • ─────┼──→ Prossimo nodo
└──────────┴──────────┘
```

### 3.3 Visualizzazione Completa in Memoria

**SINGLY LINKED LIST** (lista semplicemente concatenata):

```
LISTA: 10 → 20 → 30 → NULL

MEMORIA (indirizzi NON contigui!):
┌─ Heap @ 0x2000 ─────────┐    ┌─ Heap @ 0x3000 ─────────┐
│ Dato: 10                 │    │ Dato: 20                 │
│ Next: 0x3000  ───────────┼───→│ Next: 0x4500  ───────────┼───┐
└──────────────────────────┘    └──────────────────────────┘   │
                                                                │
┌─ Head (puntatore) ────┐    ┌─ Heap @ 0x4500 ─────────┐      │
│ 0x2000  ──────────────┼───→│ Dato: 30                 │◄─────┘
└───────────────────────┘    │ Next: NULL (fine lista)  │
                             └──────────────────────────┘

CARATTERISTICHE:
✗ Memoria SPARSA (nodi ovunque nell'heap)
✗ Accesso SEQUENZIALE (devi seguire i puntatori)
✓ Inserimento/Rimozione FACILE (cambi solo puntatori)
```

**DOUBLY LINKED LIST** (lista doppiamente concatenata):

```
LISTA: NULL ← 10 ⇄ 20 ⇄ 30 → NULL

NODO:
┌──────────┬──────────┬──────────┐
│  Prev    │  Dato    │  Next    │
│   •      │   20     │   •      │
└────┼─────┴──────────┴─────┼────┘
     │                       │
     ↓                       ↓
  Nodo precedente       Nodo successivo

PRO:
✓ Navigazione bidirezionale
✓ Rimozione più efficiente (hai già il nodo precedente)

CONTRO:
✗ 8 byte extra per nodo (puntatore aggiuntivo)
✗ Maggiore complessità di gestione
```

### 3.4 Operazioni sulle Liste

#### INSERIMENTO ALL'INIZIO: O(1) ⚡

```
PRIMA:
Head → [10] → [20] → [30] → NULL

INSERIMENTO di 5:

PASSO 1: Crea nuovo nodo
nuovo_nodo: [5] → NULL

PASSO 2: Nuovo nodo punta al primo elemento attuale
nuovo_nodo: [5] → [10] → [20] → [30] → NULL

PASSO 3: Head punta al nuovo nodo
Head → [5] → [10] → [20] → [30] → NULL

OPERAZIONI: 3 passi, sempre O(1)!
```

#### ACCESSO ALL'ELEMENTO i: O(n) ⚠️

```
RICERCA elemento in posizione 3:

Head → [10] → [20] → [30] → [40] → NULL
       ↓ 0    ↓ 1    ↓ 2    ↓ 3

PASSI:
1. Parti da Head (posizione 0)
2. Segui Next (posizione 1)
3. Segui Next (posizione 2)
4. Segui Next (posizione 3) ← TROVATO!

Nel caso peggiore: devi attraversare TUTTA la lista!
```

---

## 4. CONFRONTO: Array vs Lista

```
┌─────────────────────┬─────────────────┬──────────────────────┐
│    OPERAZIONE       │      ARRAY      │    LINKED LIST       │
├─────────────────────┼─────────────────┼──────────────────────┤
│ Accesso [i]         │ O(1) ⚡ VELOCE  │ O(n) ⚠️ LENTO        │
│ Ricerca elemento    │ O(n) non ord.   │ O(n)                 │
│                     │ O(log n) ord.   │                      │
│ Inserimento inizio  │ O(n) (shift)    │ O(1) ⚡ VELOCE       │
│ Inserimento fine    │ O(1) se spazio  │ O(n) o O(1)*         │
│ Rimozione inizio    │ O(n) (shift)    │ O(1) ⚡ VELOCE       │
│ Memoria             │ Contigua        │ Sparsa               │
│ Cache efficiency    │ Alta ✓          │ Bassa ✗              │
│ Overhead memoria    │ 0 byte          │ 8 byte/nodo (ptr)    │
│ Dimensione          │ Fissa/Realloc   │ Dinamica             │
└─────────────────────┴─────────────────┴──────────────────────┘

* O(1) se hai puntatore alla coda (doubly linked list)

🎯 USA ARRAY QUANDO:
  ✅ Accessi casuali frequenti (es. arr[i])
  ✅ Dimensione nota o raramente cambia
  ✅ Memoria contigue importante (cache)
  ✅ Semplicità di gestione

🎯 USA LINKED LIST QUANDO:
  ✅ Molti inserimenti/rimozioni
  ✅ Dimensione altamente variabile
  ✅ Accesso principalmente sequenziale
  ✅ Dimensione massima sconosciuta
```

---

## 5. ⚠️ ERRORI MORTALI

### ERRORE #1: Buffer Overflow (Array)

```
// ❌ DISASTRO!
int arr[5];
arr[10] = 42;  // SCRITTURA FUORI BOUNDS!

/* COSA SUCCEDE:
 * ┌─ Array arr ──────┐
 * │ [0][1][2][3][4]  │
 * └──────────────────┘
 *                     ↓ arr[10] scrive QUI!
 *                  ┌──────┐
 *                  │  ??? │ ← Memoria casuale!
 *                  └──────┘
 * 
 * CONSEGUENZE:
 *   - Crash (segmentation fault)
 *   - Corruzione di altre variabili
 *   - Exploit di sicurezza (buffer overflow attack!)
 *   - Comportamento imprevedibile
 */

// ✅ SOLUZIONE: Controlla sempre i bounds
#define ARRAY_SIZE 5
int arr[ARRAY_SIZE];
int index = 10;

if (index >= 0 && index < ARRAY_SIZE) {
    arr[index] = 42;  // Sicuro
} else {
    fprintf(stderr, "Errore: indice %d fuori range!\n", index);
}
```

### ERRORE #2: Memory Leak (Lista)

```
// ❌ DISASTRO!
struct Node *head = create_list();
// ... uso la lista ...
// Fine programma SENZA free()!

/* MEMORIA DOPO IL PROGRAMMA:
 * ┌─ Heap ──────────────────────┐
 * │ [Nodo1] [Nodo2] [Nodo3]...  │ ← MEMORIA PERSA!
 * └─────────────────────────────┘
 * 
 * Ogni nodo occupa memoria che NON viene restituita al sistema.
 * Se crei/distruggi liste ripetutamente: ESAURIMENTO MEMORIA!
 */

// ✅ SOLUZIONE: Libera sempre la memoria
void free_list(struct Node *head) {
    struct Node *current = head;
    while (current != NULL) {
        struct Node *next = current->next;  // Salva il prossimo
        free(current);                      // Libera l'attuale
        current = next;                     // Vai al prossimo
    }
}

// Uso:
struct Node *head = create_list();
// ... uso la lista ...
free_list(head);  // ← FONDAMENTALE!
head = NULL;      // Evita dangling pointer
```

### ERRORE #3: Dangling Pointer (Lista)

```
// ❌ DISASTRO!
struct Node *node = head->next;
free(head);
printf("%d\n", node->data);  // PERICOLO! node potrebbe essere invalido

/* SE head->next ERA PARTE DELLA STESSA ALLOCAZIONE:
 * 
 * PRIMA free():              DOPO free():
 * ┌──────────┐              ┌──────────┐
 * │ head     │              │ head     │ ← Memoria liberata!
 * ├──────────┤              ├──────────┤
 * │ next ────┼──→ node      │ ???  ????│ ← Memoria invalida!
 * └──────────┘              └──────────┘
 *                                  ↑
 *                           node punta qui! ❌
 */

// ✅ SOLUZIONE: Controlla sempre prima di usare
if (head != NULL && head->next != NULL) {
    struct Node *node = head->next;
    // Usa node solo se head non viene liberato
}
```

### ERRORE #4: Off-by-One

```
// ❌ ERRORE COMUNE
int arr[10];
for (int i = 0; i <= 10; i++) {  // ← BUG: i va da 0 a 10 (11 elementi!)
    arr[i] = i;
}
// Quando i=10: arr[10] = OVERFLOW!

// ✅ CORRETTO
for (int i = 0; i < 10; i++) {  // i va da 0 a 9 ✓
    arr[i] = i;
}
```

---

## 6. ESEMPIO PRATICO: Quando Usare Cosa?

### Scenario 1: Database di Studenti con Accessi Casuali
**Scelta: ARRAY**

```
Operazioni frequenti:
  - Accesso studente per ID: studenti[id]  → O(1)
  - Stampa tutti gli studenti              → O(n)
  - Dimensione: 1000 studenti (fissa)

PERFETTO per array!
```

### Scenario 2: Editor di Testo (Inserimenti Frequenti)
**Scelta: LISTA**

```
Operazioni frequenti:
  - Inserisci carattere in mezzo al testo  → O(1) con lista
  - Rimuovi carattere                      → O(1) con lista
  - Dimensione: variabile (il testo cresce/diminuisce)

Array richiederebbe shift continui! O(n) per ogni inserimento.
```

### Scenario 3: Implementazione di Stack
**Scelta: ARRAY (se dimensione massima nota) o LISTA**

```
Stack con array:
  - Push: O(1)
  - Pop: O(1)
  - Semplice da implementare

Stack con lista:
  - Push: O(1)
  - Pop: O(1)
  - Nessun limite di dimensione

Entrambi validi! Dipende dai requisiti.
```

---

## 7. 🔧 ESERCIZI

### 🟢 LIVELLO 1 - Comprensione

**Esercizio 1:** Dato l'array `int arr[5] = {2, 4, 6, 8, 10}` con base a 0x1000, calcola:
- Indirizzo di `arr[3]`
- Complessità temporale per accedere a `arr[3]`

<details>
<summary>Soluzione</summary>

```
Indirizzo: 0x1000 + (3 × 4 byte) = 0x100C
Complessità: O(1) - accesso diretto
```
</details>

**Esercizio 2:** Perché l'inserimento all'inizio di una lista è O(1) mentre in un array è O(n)?

<details>
<summary>Soluzione</summary>

```
LISTA:
  - Crei nuovo nodo
  - Nuovo nodo → primo elemento attuale
  - Head → nuovo nodo
  TOTALE: 3 operazioni fisse → O(1)

ARRAY:
  - Devi spostare TUTTI gli elementi di una posizione
  - Se hai n elementi, fai n spostamenti
  TOTALE: n operazioni → O(n)
```
</details>

### 🟡 LIVELLO 2 - Applicazione

**Esercizio 3:** Scrivi pseudocodice per invertire un array in-place (senza array aggiuntivo).

<details>
<summary>Soluzione</summary>

```
FUNZIONE inverti_array(arr, n):
    inizio = 0
    fine = n - 1
    
    MENTRE inizio < fine:
        // Scambia arr[inizio] con arr[fine]
        temp = arr[inizio]
        arr[inizio] = arr[fine]
        arr[fine] = temp
        
        inizio++
        fine--

Complessità: O(n) tempo, O(1) spazio
```
</details>

### 🟠 LIVELLO 3 - Analisi

**Esercizio 4:** Trova l'errore in questo codice:

```
int *create_array(int n) {
    int arr[n];
    for (int i = 0; i < n; i++) {
        arr[i] = i;
    }
    return arr;  // Bug?
}
```

<details>
<summary>Soluzione</summary>

```
❌ ERRORE: Ritorna puntatore a variabile locale!

arr è allocato sullo STACK della funzione.
Quando la funzione termina, lo stack viene "ripulito".

MEMORIA:
DURANTE create_array():
┌─ Stack create_array ────┐
│ arr: [0][1][2]...[n-1]  │
└─────────────────────────┘

DOPO return:
┌─ Stack ─────────────────┐
│ ???  (memoria invalida) │ ← arr non esiste più!
└─────────────────────────┘

✅ SOLUZIONE: Usa malloc()
int *create_array(int n) {
    int *arr = malloc(n * sizeof(int));
    if (arr == NULL) return NULL;
    for (int i = 0; i < n; i++) {
        arr[i] = i;
    }
    return arr;  // ✓ arr è nell'heap, persiste!
}
```
</details>

### 🔴 LIVELLO 4 - Sintesi

**Esercizio 5:** Implementa una funzione che rimuove duplicati da una lista ordinata in O(n) tempo.

<details>
<summary>Soluzione</summary>

```
FUNZIONE rimuovi_duplicati(head):
    SE head == NULL O head->next == NULL:
        RETURN head  // Lista vuota o un solo elemento
    
    current = head
    
    MENTRE current->next != NULL:
        SE current->data == current->next->data:
            // Duplicato trovato!
            duplicate = current->next
            current->next = current->next->next  // Salta il duplicato
            LIBERA(duplicate)
        ALTRIMENTI:
            current = current->next  // Vai avanti
    
    RETURN head

ESEMPIO:
Input:  1 → 1 → 2 → 3 → 3 → NULL
Output: 1 → 2 → 3 → NULL

Complessità: O(n) - attraversi la lista una volta sola
```
</details>

---

## 8. 🎓 PUNTI CHIAVE

**ARRAY:**
- ✅ Memoria contigua → accesso O(1)
- ✅ Cache-friendly → performance ottima
- ✅ Semplice da usare
- ❌ Dimensione fissa o costosa da modificare
- ❌ Inserimento/rimozione lenti: O(n)

**LISTE CONCATENATE:**
- ✅ Inserimento/rimozione veloci: O(1)
- ✅ Dimensione dinamica
- ❌ Accesso sequenziale: O(n)
- ❌ Overhead memoria (puntatori)
- ❌ Cache-unfriendly

**REGOLA D'ORO:**
> Se accedi spesso per indice → **ARRAY**
> Se inserisci/rimuovi spesso → **LISTA**

---

## 9. DEBUG: Problemi Comuni

### Problema 1: "Segmentation Fault" con Array

```
SINTOMO: Il programma crasha

CAUSA PROBABILE:
✗ Accesso fuori bounds (arr[i] con i >= dimensione)
✗ Scrittura su array non inizializzato

DEBUG:
1. Usa valgrind: valgrind ./programma
2. Aggiungi controlli bounds:
   assert(i >= 0 && i < SIZE);
3. Compila con -fsanitize=address
```

### Problema 2: Memory Leak con Liste

```
SINTOMO: Memoria cresce nel tempo

CAUSA PROBABILE:
✗ Non liberi i nodi dopo l'uso
✗ Perdi riferimento alla lista senza liberarla

DEBUG:
1. Valgrind: valgrind --leak-check=full ./programma
2. Ogni malloc() deve avere un free() corrispondente
3. Usa tool come AddressSanitizer
```

---

## 📚 Prossimi Passi

Ora che comprendi array e liste, sei pronto per:
- [Lezione 6: Stack e Queue](06_stack_queue.md) - Strutture dati basate su liste/array
- [Lezione 7: Alberi](07_alberi.md) - Strutture dati gerarchiche

---

[← Ordinamento](04_ordinamento.md) | [Torna al Modulo](README.md) | [Stack e Queue →](06_stack_queue.md)
