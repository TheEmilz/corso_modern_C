# Lezione 4: Algoritmi di Ordinamento

## Introduzione

L'**ordinamento** è il processo di organizzare elementi in un ordine specifico (crescente o decrescente). È una delle operazioni più studiate in informatica perché:
- È fondamentale per molte applicazioni
- Migliora l'efficienza di altre operazioni (es: ricerca binaria)
- Esistono molti algoritmi con trade-off diversi

## Perché Ordinare?

### Vantaggi dei Dati Ordinati

1. **Ricerca più veloce**: Ricerca binaria O(log N) vs lineare O(N)
2. **Duplicati facili da trovare**: Elementi uguali sono adiacenti
3. **Mediana e statistiche**: Più facili da calcolare
4. **Presentazione**: Dati più leggibili per utenti
5. **Algoritmi dipendenti**: Molti algoritmi richiedono dati ordinati

### Esempi Pratici

- Rubrica telefonica ordinata alfabeticamente
- Classifica studenti per voto
- Prodotti ordinati per prezzo
- File ordinati per data
- Risultati di ricerca per rilevanza

## Bubble Sort - Ordinamento a Bolle

L'algoritmo più semplice ma non il più efficiente.

### Concetto

Come bolle in un liquido che salgono in superficie, gli elementi "più leggeri" (più piccoli) salgono verso l'inizio.

### Come Funziona

1. Confronta elementi adiacenti
2. Se sono nel ordine sbagliato, scambiali
3. Ripeti fino a quando nessuno scambio è necessario

### Pseudocodice

```
FUNZIONE BubbleSort(array, n)
    PER i da 0 a n-1:
        scambiato ← Falso
        
        PER j da 0 a n-i-2:
            SE array[j] > array[j+1]:
                Scambia array[j] e array[j+1]
                scambiato ← Vero
        
        SE NON scambiato:
            BREAK  // Array già ordinato
FINE
```

### Esempio Passo-Passo

```
Array iniziale: [5, 2, 8, 1, 9]

PASSATA 1:
  Confronta 5 e 2: [2, 5, 8, 1, 9]  // Scambio
  Confronta 5 e 8: [2, 5, 8, 1, 9]  // OK
  Confronta 8 e 1: [2, 5, 1, 8, 9]  // Scambio
  Confronta 8 e 9: [2, 5, 1, 8, 9]  // OK
  Risultato: [2, 5, 1, 8, 9]  // 9 è al posto giusto

PASSATA 2:
  Confronta 2 e 5: [2, 5, 1, 8, 9]  // OK
  Confronta 5 e 1: [2, 1, 5, 8, 9]  // Scambio
  Confronta 5 e 8: [2, 1, 5, 8, 9]  // OK
  Risultato: [2, 1, 5, 8, 9]  // 8 è al posto giusto

PASSATA 3:
  Confronta 2 e 1: [1, 2, 5, 8, 9]  // Scambio
  Confronta 2 e 5: [1, 2, 5, 8, 9]  // OK
  Risultato: [1, 2, 5, 8, 9]  // 5 è al posto giusto

PASSATA 4:
  Confronta 1 e 2: [1, 2, 5, 8, 9]  // OK
  Nessuno scambio → Array ordinato!

Array finale: [1, 2, 5, 8, 9]
```

### Analisi

- **Complessità temporale**:
  - Caso migliore: O(N) - array già ordinato
  - Caso medio: O(N²)
  - Caso peggiore: O(N²) - array ordinato al contrario
- **Complessità spaziale**: O(1) - in-place
- **Stabile**: Sì - mantiene l'ordine relativo di elementi uguali

### Quando Usarlo

✅ **Usa Bubble Sort per**:
- Array molto piccoli (< 10 elementi)
- Scopo didattico
- Array quasi ordinati

❌ **Non usare per**:
- Array grandi
- Performance critiche

## Selection Sort - Ordinamento per Selezione

Trova ripetutamente il minimo e lo posiziona all'inizio.

### Concetto

Seleziona il minimo dalla parte non ordinata e spostalo nella parte ordinata.

### Pseudocodice

```
FUNZIONE SelectionSort(array, n)
    PER i da 0 a n-1:
        min_idx ← i
        
        // Trova il minimo nella parte non ordinata
        PER j da i+1 a n-1:
            SE array[j] < array[min_idx]:
                min_idx ← j
        
        // Scambia minimo con elemento corrente
        SE min_idx ≠ i:
            Scambia array[i] e array[min_idx]
FINE
```

### Esempio Visivo

```
Array: [64, 25, 12, 22, 11]

ITERAZIONE 1: Trova minimo in [64, 25, 12, 22, 11]
  Minimo = 11 (indice 4)
  [11, 25, 12, 22, 64]

ITERAZIONE 2: Trova minimo in [25, 12, 22, 64]
  Minimo = 12 (indice 2)
  [11, 12, 25, 22, 64]

ITERAZIONE 3: Trova minimo in [25, 22, 64]
  Minimo = 22 (indice 3)
  [11, 12, 22, 25, 64]

ITERAZIONE 4: Trova minimo in [25, 64]
  Minimo = 25 (già al posto)
  [11, 12, 22, 25, 64]

Array ordinato: [11, 12, 22, 25, 64]
```

### Analisi

- **Complessità temporale**: O(N²) - sempre, anche se array ordinato
- **Complessità spaziale**: O(1)
- **Stabile**: No
- **Scambi**: O(N) - meno di Bubble Sort

## Insertion Sort - Ordinamento per Inserimento

Come ordinare carte da gioco: inserisci ogni carta nella posizione corretta.

### Concetto

Costruisci gradualmente l'array ordinato inserendo ogni elemento nella sua posizione corretta.

### Pseudocodice

```
FUNZIONE InsertionSort(array, n)
    PER i da 1 a n-1:
        chiave ← array[i]
        j ← i - 1
        
        // Sposta elementi maggiori di chiave
        MENTRE j ≥ 0 AND array[j] > chiave:
            array[j+1] ← array[j]
            j ← j - 1
        
        // Inserisci chiave nella posizione corretta
        array[j+1] ← chiave
FINE
```

### Esempio Dettagliato

```
Array: [5, 2, 4, 6, 1, 3]

[5] | 2, 4, 6, 1, 3       // Parte ordinata | Parte non ordinata

ITERAZIONE 1: Inserisci 2
  Confronta 2 con 5: 2 < 5, sposta 5
  [2, 5] | 4, 6, 1, 3

ITERAZIONE 2: Inserisci 4
  Confronta 4 con 5: 4 < 5, sposta 5
  Confronta 4 con 2: 4 > 2, inserisci
  [2, 4, 5] | 6, 1, 3

ITERAZIONE 3: Inserisci 6
  Confronta 6 con 5: 6 > 5, già al posto
  [2, 4, 5, 6] | 1, 3

ITERAZIONE 4: Inserisci 1
  Confronta 1 con 6: 1 < 6, sposta 6
  Confronta 1 con 5: 1 < 5, sposta 5
  Confronta 1 con 4: 1 < 4, sposta 4
  Confronta 1 con 2: 1 < 2, sposta 2
  [1, 2, 4, 5, 6] | 3

ITERAZIONE 5: Inserisci 3
  Confronta 3 con 6: 3 < 6, sposta 6
  Confronta 3 con 5: 3 < 5, sposta 5
  Confronta 3 con 4: 3 < 4, sposta 4
  Confronta 3 con 2: 3 > 2, inserisci
  [1, 2, 3, 4, 5, 6]

Array ordinato!
```

### Analisi

- **Complessità temporale**:
  - Caso migliore: O(N) - array già ordinato
  - Caso medio: O(N²)
  - Caso peggiore: O(N²)
- **Complessità spaziale**: O(1)
- **Stabile**: Sì
- **Adattivo**: Sì - veloce su array quasi ordinati

### Quando Usarlo

✅ **Ottimo per**:
- Array piccoli (< 50 elementi)
- Array quasi ordinati
- Ordinamento "online" (elementi arrivano uno alla volta)

## Merge Sort - Ordinamento per Fusione

Algoritmo divide-et-impera efficiente.

### Concetto

1. **Dividi**: Dividi array a metà
2. **Conquista**: Ordina ricorsivamente le due metà
3. **Combina**: Fondi le due metà ordinate

### Pseudocodice

```
FUNZIONE MergeSort(array, sinistra, destra)
    SE sinistra < destra:
        medio ← (sinistra + destra) / 2
        
        // Ordina le due metà
        MergeSort(array, sinistra, medio)
        MergeSort(array, medio+1, destra)
        
        // Fondi le metà ordinate
        Merge(array, sinistra, medio, destra)
FINE

FUNZIONE Merge(array, sinistra, medio, destra)
    n1 ← medio - sinistra + 1
    n2 ← destra - medio
    
    // Crea array temporanei
    Left ← array[sinistra...medio]
    Right ← array[medio+1...destra]
    
    // Fondi i due array
    i ← 0, j ← 0, k ← sinistra
    
    MENTRE i < n1 AND j < n2:
        SE Left[i] ≤ Right[j]:
            array[k] ← Left[i]
            i++
        ALTRIMENTI:
            array[k] ← Right[j]
            j++
        k++
    
    // Copia elementi rimanenti
    MENTRE i < n1:
        array[k] ← Left[i]
        i++, k++
    
    MENTRE j < n2:
        array[k] ← Right[j]
        j++, k++
FINE
```

### Esempio Visivo

```
Array originale: [38, 27, 43, 3, 9, 82, 10]

DIVIDI:
        [38, 27, 43, 3, 9, 82, 10]
         /                       \
    [38, 27, 43]            [3, 9, 82, 10]
     /        \              /           \
  [38]   [27, 43]        [3, 9]      [82, 10]
          /    \         /    \        /    \
       [27]   [43]     [3]   [9]    [82]  [10]

CONQUISTA (Merge):
       [27]   [43]     [3]   [9]    [82]  [10]
          \    /         \    /        \    /
        [27, 43]        [3, 9]        [10, 82]
             \            /                |
          [27, 43]   [3, 9, 10, 82]
                  \      /
              [3, 9, 10, 27, 43, 82]

Totale livelli: log₂(7) ≈ 3
```

### Analisi

- **Complessità temporale**: O(N log N) - sempre!
- **Complessità spaziale**: O(N) - array temporanei
- **Stabile**: Sì
- **Non adattivo**: Stessa complessità anche su array ordinati

## Quick Sort - Ordinamento Rapido

Uno degli algoritmi più usati in pratica.

### Concetto

1. Scegli un **pivot**
2. Partiziona: elementi < pivot a sinistra, >= pivot a destra
3. Ordina ricorsivamente le due partizioni

### Pseudocodice

```
FUNZIONE QuickSort(array, low, high)
    SE low < high:
        pivot_idx ← Partition(array, low, high)
        QuickSort(array, low, pivot_idx - 1)
        QuickSort(array, pivot_idx + 1, high)
FINE

FUNZIONE Partition(array, low, high)
    pivot ← array[high]  // Scegli ultimo elemento come pivot
    i ← low - 1
    
    PER j da low a high-1:
        SE array[j] < pivot:
            i++
            Scambia array[i] e array[j]
    
    Scambia array[i+1] e array[high]
    RITORNA i + 1
FINE
```

### Esempio

```
Array: [10, 80, 30, 90, 40, 50, 70]
Pivot: 70 (ultimo elemento)

PARTIZIONE:
i=-1, j=0: 10<70 → i=0, scambia array[0] con array[0]
           [10, 80, 30, 90, 40, 50, 70]

i=0, j=1: 80≥70 → nessuno scambio
          [10, 80, 30, 90, 40, 50, 70]

i=0, j=2: 30<70 → i=1, scambia array[1] con array[2]
          [10, 30, 80, 90, 40, 50, 70]

i=1, j=3: 90≥70 → nessuno scambio

i=1, j=4: 40<70 → i=2, scambia array[2] con array[4]
          [10, 30, 40, 90, 80, 50, 70]

i=2, j=5: 50<70 → i=3, scambia array[3] con array[5]
          [10, 30, 40, 50, 80, 90, 70]

Posiziona pivot: scambia array[4] con array[6]
                 [10, 30, 40, 50, 70, 90, 80]

Pivot 70 è nella posizione corretta (indice 4)

Ricorsione su [10, 30, 40, 50] e [90, 80]
```

### Analisi

- **Complessità temporale**:
  - Caso migliore: O(N log N) - pivot sempre al centro
  - Caso medio: O(N log N)
  - Caso peggiore: O(N²) - pivot sempre minimo/massimo
- **Complessità spaziale**: O(log N) - stack ricorsione
- **Non stabile**
- **In-place**

### Ottimizzazione: Randomized Quick Sort

```
FUNZIONE RandomizedPartition(array, low, high)
    // Scegli pivot casuale
    random_idx ← Random(low, high)
    Scambia array[random_idx] e array[high]
    RITORNA Partition(array, low, high)
FINE
```

Questo rende molto improbabile il caso peggiore.

## Confronto Algoritmi

| Algoritmo | Migliore | Medio | Peggiore | Spazio | Stabile |
|-----------|----------|-------|----------|--------|---------|
| Bubble Sort | O(N) | O(N²) | O(N²) | O(1) | Sì |
| Selection Sort | O(N²) | O(N²) | O(N²) | O(1) | No |
| Insertion Sort | O(N) | O(N²) | O(N²) | O(1) | Sì |
| Merge Sort | O(N log N) | O(N log N) | O(N log N) | O(N) | Sì |
| Quick Sort | O(N log N) | O(N log N) | O(N²) | O(log N) | No |

### Grafico Performance (N = 10,000)

```
Bubble Sort:     ████████████████████████████████████████  50,000,000
Selection Sort:  ████████████████████████████████████████  50,000,000
Insertion Sort:  ████████████████████████████████████████  25,000,000
Merge Sort:      ██                                        133,000
Quick Sort:      █                                         100,000
```

## Quale Algoritmo Scegliere?

### Array Piccoli (< 50 elementi)

**Insertion Sort** - Semplice, veloce, basso overhead

### Array Medio-Grandi

**Quick Sort** (con pivot random) - Veloce in media, in-place

### Quando Serve Stabilità

**Merge Sort** - Sempre O(N log N), stabile

### Array Quasi Ordinati

**Insertion Sort** - O(N) su array quasi ordinati

### Quando Memoria è Limitata

**Quick Sort** o **Heap Sort** - O(log N) o O(1) spazio

## Debug: Errori Comuni

### Errore 1: Bubble Sort Non Termina

```
// SBAGLIATO
PER i da 0 a n:  // Dovrebbe essere n-1
    PER j da 0 a n-1:  // Dovrebbe essere n-i-2
```

**Fix**: Correggi i limiti dei loop

### Errore 2: Merge Sort - Array Non Liberati

```
// Memory leak!
Left ← alloca memoria
Right ← alloca memoria
// ... merge ...
// Dimenticato: libera Left e Right
```

**Fix**: Sempre libera memoria allocata

### Errore 3: Quick Sort - Pivot Sbagliato

```
// Loop infinito se low = high
QuickSort(array, low, pivot_idx)    // Dovrebbe essere pivot_idx-1
QuickSort(array, pivot_idx, high)   // Dovrebbe essere pivot_idx+1
```

## Esercizi Pratici

### Esercizio 1: Traccia Bubble Sort

Traccia passo-passo Bubble Sort per:
```
Array: [3, 1, 4, 1, 5]
```

### Esercizio 2: Implementa Selection Sort

Scrivi pseudocodice completo per Selection Sort.

### Esercizio 3: Analizza Complessità

Perché Insertion Sort è O(N) su array ordinati ma O(N²) nel caso peggiore?

### Esercizio 4: Ottimizza Bubble Sort

Come puoi migliorare Bubble Sort per terminare prima su array quasi ordinati?

## Punti Chiave

✅ Bubble Sort: Semplice ma O(N²), OK per piccoli array
✅ Selection Sort: Sempre O(N²), pochi scambi
✅ Insertion Sort: O(N) su array quasi ordinati, ottimo per piccoli
✅ Merge Sort: Sempre O(N log N), stabile, usa O(N) spazio
✅ Quick Sort: O(N log N) in media, in-place, non stabile
✅ Per array grandi, usa Merge Sort o Quick Sort
✅ Per array piccoli o quasi ordinati, usa Insertion Sort

---

[← Ricerca](03_ricerca.md) | [Torna al Modulo](README.md) | [Prossimo: Array e Liste →](05_array_liste.md)
