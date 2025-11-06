# Lezione 2: Complessità e Analisi degli Algoritmi

## Perché Misurare l'Efficienza?

Immagina di avere due ricette per lo stesso piatto:
- **Ricetta A**: 10 passaggi, 30 minuti
- **Ricetta B**: 20 passaggi, 15 minuti

Quale sceglieresti? Dipende! Se hai poco tempo, la B. Se vuoi qualcosa di più semplice, la A.

Lo stesso vale per gli algoritmi: vogliamo sapere **quanto tempo** e **quanta memoria** richiedono.

## Notazione Big-O

La **notazione Big-O** descrive come cresce il tempo di esecuzione di un algoritmo al crescere della dimensione dell'input.

### Analogia: Cercare un Libro

**Scenario**: Hai una libreria con N libri e vuoi trovare "Il Signore degli Anelli"

**Strategia 1: Ricerca Lineare**
- Controlli ogni libro uno per uno
- Nel caso peggiore: controlli tutti gli N libri
- Tempo: **O(N)** - lineare

**Strategia 2: Libri Ordinati Alfabeticamente**
- Vai direttamente alla sezione "T"
- Cerchi solo in quella sezione
- Tempo: **O(log N)** - logaritmico

**Strategia 3: Conosci Esattamente la Posizione**
- Vai direttamente allo scaffale giusto
- Tempo: **O(1)** - costante

### Complessità Comuni (dal Migliore al Peggiore)

```
O(1)         < O(log N) < O(N) < O(N log N) < O(N²) < O(2^N) < O(N!)
Costante     Logaritmica  Lineare  Linearitmica  Quadratica  Esponenziale  Fattoriale
```

**Visualizzazione per N = 100**:
```
O(1):        1 operazione
O(log N):    ~7 operazioni      (log₂ 100 ≈ 6.64)
O(N):        100 operazioni
O(N log N):  ~664 operazioni
O(N²):       10,000 operazioni
O(2^N):      1,267,650,600,228,229,401,496,703,205,376 operazioni (impraticabile!)
```

## O(1) - Tempo Costante

L'algoritmo esegue lo **stesso numero di operazioni** indipendentemente dalla dimensione dell'input.

### Esempio: Accesso a un Array

```
FUNZIONE PrimoElemento(array)
    RITORNA array[0]
FINE
```

**Analisi**:
- Operazione singola: accesso diretto
- Non dipende dalla dimensione dell'array
- **Complessità: O(1)**

### Esempio: Calcolo con Formula

```
FUNZIONE SommaN(n)
    // Somma dei primi n numeri: 1 + 2 + ... + n
    RITORNA n * (n + 1) / 2
FINE
```

**Confronto con Approccio Naive**:
```
// Approccio O(N)
somma ← 0
PER i da 1 a n:
    somma ← somma + i

// Approccio O(1) con formula di Gauss
somma ← n * (n + 1) / 2
```

## O(N) - Tempo Lineare

Il tempo cresce **proporzionalmente** alla dimensione dell'input.

### Esempio: Somma di Array

```
FUNZIONE SommaArray(array, n)
    somma ← 0
    PER i da 0 a n-1:
        somma ← somma + array[i]
    RITORNA somma
FINE
```

**Analisi**:
- Un'operazione per ogni elemento
- N elementi → N operazioni
- **Complessità: O(N)**

### Esempio: Ricerca Lineare

```
FUNZIONE CercaElemento(array, n, target)
    PER i da 0 a n-1:
        SE array[i] = target:
            RITORNA i
    RITORNA -1  // Non trovato
FINE
```

**Analisi dei Casi**:
- **Caso migliore**: O(1) - target è il primo elemento
- **Caso peggiore**: O(N) - target è l'ultimo o non c'è
- **Caso medio**: O(N/2) ≈ O(N)

## O(log N) - Tempo Logaritmico

Il tempo cresce **logaritmicamente**: ogni passo dimezza il problema.

### Esempio: Ricerca Binaria

```
FUNZIONE RicercaBinaria(array_ordinato, n, target)
    sinistra ← 0
    destra ← n - 1
    
    MENTRE sinistra ≤ destra:
        medio ← (sinistra + destra) / 2
        
        SE array[medio] = target:
            RITORNA medio
        ALTRIMENTI SE array[medio] < target:
            sinistra ← medio + 1
        ALTRIMENTI:
            destra ← medio - 1
    
    RITORNA -1
FINE
```

**Esempio di Esecuzione**:
```
Array: [1, 3, 5, 7, 9, 11, 13, 15]
Target: 7

Passo 1: range [0, 7], medio = 3, array[3] = 7 ✓ Trovato!

Confronta con ricerca lineare:
- Ricerca lineare: 4 confronti
- Ricerca binaria: 1 confronto
```

**Altro Esempio**:
```
Array: [1, 3, 5, 7, 9, 11, 13, 15, 17, 19, 21, 23, 25, 27, 29, 31]
Target: 23

Passo 1: range [0, 15], medio = 7, array[7] = 15 < 23 → cerca a destra
Passo 2: range [8, 15], medio = 11, array[11] = 23 ✓ Trovato!

- Array di 16 elementi
- Ricerca lineare: fino a 12 confronti
- Ricerca binaria: 2 confronti
```

**Analisi**:
- Ogni passo dimezza lo spazio di ricerca
- N → N/2 → N/4 → N/8 → ... → 1
- Numero di passi: log₂(N)
- **Complessità: O(log N)**

## O(N²) - Tempo Quadratico

Il tempo cresce con il **quadrato** della dimensione dell'input. Comune in algoritmi con cicli annidati.

### Esempio: Bubble Sort

```
FUNZIONE BubbleSort(array, n)
    PER i da 0 a n-1:              // Ciclo esterno: N volte
        PER j da 0 a n-i-2:        // Ciclo interno: N volte
            SE array[j] > array[j+1]:
                Scambia array[j] e array[j+1]
FINE
```

**Analisi**:
- Ciclo esterno: N iterazioni
- Ciclo interno: N iterazioni per ogni iterazione esterna
- Totale: N × N = N²
- **Complessità: O(N²)**

**Conteggio Operazioni**:
```
N = 5:
Iterazione 1: 4 confronti
Iterazione 2: 3 confronti
Iterazione 3: 2 confronti
Iterazione 4: 1 confronto
Totale: 4+3+2+1 = 10 ≈ N²/2
```

### Esempio: Duplicati

```
FUNZIONE HaDuplicati(array, n)
    PER i da 0 a n-1:
        PER j da i+1 a n-1:
            SE array[i] = array[j]:
                RITORNA Vero
    RITORNA Falso
FINE
```

**Complessità: O(N²)** - confronta ogni coppia di elementi

## O(N log N) - Tempo Linearitmico

Comune negli algoritmi di ordinamento efficienti.

### Esempio: Merge Sort (Concettuale)

```
FUNZIONE MergeSort(array)
    SE lunghezza(array) ≤ 1:
        RITORNA array
    
    medio ← lunghezza(array) / 2
    sinistra ← MergeSort(prima metà)
    destra ← MergeSort(seconda metà)
    
    RITORNA Merge(sinistra, destra)
FINE
```

**Analisi**:
- Divide l'array in due parti: log(N) livelli
- Ogni livello processa N elementi (merge)
- Totale: N × log(N)
- **Complessità: O(N log N)**

**Confronto Ordinamento**:
```
N = 1000
Bubble Sort (O(N²)):     ~1,000,000 operazioni
Merge Sort (O(N log N)): ~10,000 operazioni
```

## Spazio vs Tempo

Gli algoritmi hanno due tipi di complessità:

### Complessità Temporale
Quanto **tempo** impiega l'algoritmo

### Complessità Spaziale
Quanta **memoria** usa l'algoritmo

### Esempio: Inversione Array

**Approccio 1: Nuovo Array (O(N) spazio)**
```
FUNZIONE Inverti1(array, n)
    nuovo ← nuovo array di dimensione n
    PER i da 0 a n-1:
        nuovo[i] ← array[n-1-i]
    RITORNA nuovo
FINE

Tempo: O(N)
Spazio: O(N) - array aggiuntivo
```

**Approccio 2: In-Place (O(1) spazio)**
```
FUNZIONE Inverti2(array, n)
    PER i da 0 a n/2:
        Scambia array[i] con array[n-1-i]
FINE

Tempo: O(N)
Spazio: O(1) - solo variabili temporanee
```

**Trade-off**: L'approccio 2 è più efficiente in spazio ma modifica l'array originale.

## Debug: Identificare Colli di Bottiglia

### Tecnica 1: Conta le Operazioni

**Esempio**:
```
FUNZIONE Esempio(n)
    PER i da 0 a n:           // O(N)
        Stampa i
    
    PER i da 0 a n:           // O(N)
        PER j da 0 a n:       // O(N)
            Stampa i, j       // Totale O(N²)
FINE
```

**Analisi**:
- Prima parte: O(N)
- Seconda parte: O(N²)
- **Totale: O(N²)** - dominato dalla parte peggiore

### Tecnica 2: Misura il Tempo Reale

**Test con Input Crescenti**:
```
N = 10:    tempo = 0.1s
N = 100:   tempo = 1s     (10x input → 10x tempo) → O(N)?
N = 1000:  tempo = 100s   (10x input → 100x tempo) → O(N²)!
```

Se raddoppiando l'input:
- Tempo raddoppia → O(N)
- Tempo quadruplica → O(N²)
- Tempo aumenta poco → O(log N)

### Tecnica 3: Profiling Mentale

**Domande da Porsi**:
1. Ci sono cicli? Quanti livelli di annidamento?
2. La dimensione del problema si riduce a ogni passo?
3. Creo copie dei dati o lavoro in-place?

**Esempio Debug**:
```
FUNZIONE MisterioX(array, n)
    PER i da 0 a n:              // Ciclo O(N)
        SE CondizioneCostosa():  // O(N) dentro il ciclo!
            // ...
FINE

Complessità totale: O(N²)
Ottimizzazione: Muovi CondizioneCostosa() fuori dal ciclo
```

## Casi Migliore, Medio e Peggiore

### Esempio: Ricerca Lineare

```
Array: [5, 2, 8, 1, 9]
Target: 1

Caso migliore: Target è il primo elemento
- Confronti: 1
- Complessità: O(1)

Caso peggiore: Target è l'ultimo o non c'è
- Confronti: N
- Complessità: O(N)

Caso medio: Target è in posizione casuale
- Confronti: N/2
- Complessità: O(N) - ignoriamo costanti
```

**Quale Usare?**
- Di solito analizziamo il **caso peggiore** (garanzia)
- Il **caso medio** è utile per algoritmi randomizzati
- Il **caso migliore** è raramente significativo

## Confronto Algoritmi: Esempio Pratico

### Problema: Trovare Duplicati

**Soluzione 1: Doppio Loop O(N²)**
```
PER ogni elemento i:
    PER ogni elemento j dopo i:
        SE i = j:
            RITORNA Vero
```

**Soluzione 2: Ordinamento + Scansione O(N log N)**
```
Ordina l'array                // O(N log N)
PER ogni coppia di adiacenti: // O(N)
    SE sono uguali:
        RITORNA Vero
```

**Soluzione 3: Hash Set O(N)**
```
set ← insieme vuoto
PER ogni elemento:          // O(N)
    SE elemento in set:     // O(1) con hash
        RITORNA Vero
    Aggiungi elemento a set
```

**Confronto**:
```
N = 10,000

Soluzione 1: ~100,000,000 operazioni
Soluzione 2: ~133,000 operazioni
Soluzione 3: ~10,000 operazioni

Soluzione 3 è la migliore per tempo!
Ma usa O(N) spazio aggiuntivo vs O(1)
```

## Esercizi Pratici

### Esercizio 1: Analizza la Complessità

```
FUNZIONE Esercizio1(array, n)
    somma ← 0
    PER i da 0 a n-1:
        somma ← somma + array[i]
    RITORNA somma
FINE
```

**Domanda**: Qual è la complessità temporale?

<details>
<summary>Risposta</summary>
O(N) - un ciclo che itera N volte
</details>

### Esercizio 2: Migliora l'Efficienza

```
FUNZIONE Esercizio2(n)
    risultato ← 0
    PER i da 0 a n:
        PER j da 0 a n:
            risultato ← risultato + i * j
    RITORNA risultato
FINE
```

**Domanda**: Come puoi ottimizzare questo algoritmo?

<details>
<summary>Suggerimento</summary>
Nota che stai sommando i*j per tutte le coppie (i,j).
Puoi usare una formula matematica?
</details>

### Esercizio 3: Scegli l'Algoritmo

Hai bisogno di cercare un elemento in una lista molto grande che verrà interrogata milioni di volte.

**Opzione A**: Tieni la lista non ordinata, usa ricerca lineare O(N)
**Opzione B**: Ordina una volta O(N log N), poi usa ricerca binaria O(log N)
**Opzione C**: Usa una hash table O(N) spazio, ricerca O(1)

**Domanda**: Quale scegli e perché?

## Punti Chiave

✅ Big-O descrive come cresce il tempo al crescere dell'input
✅ O(1) < O(log N) < O(N) < O(N log N) < O(N²) < O(2^N)
✅ Cicli annidati spesso indicano O(N²)
✅ Dividere il problema a metà suggerisce O(log N)
✅ Considera sia tempo che spazio
✅ Di solito analizziamo il caso peggiore
✅ Confronta algoritmi per scegliere il migliore

## Regole Pratiche

1. **Operazione singola**: O(1)
2. **Un ciclo**: O(N)
3. **Cicli annidati**: O(N²) o O(N³)...
4. **Dividere a metà**: O(log N)
5. **Ordinamento efficiente**: O(N log N)
6. **Tutte le sottoinsiemi**: O(2^N)

---

[← Pensiero Computazionale](01_pensiero_computazionale.md) | [Torna al Modulo](README.md) | [Prossimo: Algoritmi di Ricerca →](03_ricerca.md)
