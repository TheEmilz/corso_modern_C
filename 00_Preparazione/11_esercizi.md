# Lezione 11: Esercizi Progressivi

## Introduzione

Questa lezione contiene esercizi di difficoltà crescente che coprono tutti i concetti appresi nel modulo. Ogni esercizio include:
- Descrizione del problema
- Livello di difficoltà
- Suggerimenti
- Soluzione completa commentata

## 🟢 Livello 1: Principiante

### Esercizio 1.1: Somma Array

**Problema**: Scrivi un algoritmo che calcola la somma di tutti gli elementi in un array.

**Input**: Array di numeri interi
**Output**: Somma totale

**Esempio**:
```
Input: [1, 2, 3, 4, 5]
Output: 15
```

**Suggerimento**: Usa un accumulatore inizializzato a 0.

<details>
<summary>Soluzione</summary>

```
FUNZIONE SommaArray(array, n)
    // Inizializza accumulatore
    somma ← 0
    
    // Itera su tutti gli elementi
    PER i da 0 a n-1:
        somma ← somma + array[i]
    
    RITORNA somma
FINE

// Tracciamento per [1, 2, 3, 4, 5]:
// i=0: somma = 0 + 1 = 1
// i=1: somma = 1 + 2 = 3
// i=2: somma = 3 + 3 = 6
// i=3: somma = 6 + 4 = 10
// i=4: somma = 10 + 5 = 15

Complessità: O(N) tempo, O(1) spazio
```
</details>

### Esercizio 1.2: Trova Massimo e Minimo

**Problema**: Trova sia il massimo che il minimo in un array.

**Input**: Array di numeri
**Output**: Coppia (minimo, massimo)

**Esempio**:
```
Input: [3, 7, 2, 9, 1, 5]
Output: (1, 9)
```

<details>
<summary>Soluzione</summary>

```
FUNZIONE TrovaMinMax(array, n)
    SE n = 0:
        ERRORE "Array vuoto"
    
    // Inizializza con primo elemento
    minimo ← array[0]
    massimo ← array[0]
    
    // Cerca tra gli altri elementi
    PER i da 1 a n-1:
        SE array[i] < minimo:
            minimo ← array[i]
        
        SE array[i] > massimo:
            massimo ← array[i]
    
    RITORNA (minimo, massimo)
FINE

Complessità: O(N) tempo, O(1) spazio
```
</details>

### Esercizio 1.3: Conta Pari e Dispari

**Problema**: Conta quanti numeri pari e dispari ci sono in un array.

**Input**: Array di interi
**Output**: (contatore_pari, contatore_dispari)

<details>
<summary>Soluzione</summary>

```
FUNZIONE ContaParieDispari(array, n)
    pari ← 0
    dispari ← 0
    
    PER i da 0 a n-1:
        SE array[i] % 2 = 0:
            pari ← pari + 1
        ALTRIMENTI:
            dispari ← dispari + 1
    
    RITORNA (pari, dispari)
FINE

// Esempio: [1, 2, 3, 4, 5, 6]
// Pari: 2, 4, 6 → conta = 3
// Dispari: 1, 3, 5 → conta = 3
```
</details>

## 🟡 Livello 2: Intermedio

### Esercizio 2.1: Rimuovi Duplicati

**Problema**: Dato un array ordinato, rimuovi i duplicati in-place.

**Input**: Array ordinato con possibili duplicati
**Output**: Numero di elementi unici

**Esempio**:
```
Input: [1, 1, 2, 2, 2, 3, 4, 4, 5]
Output: 5 (array diventa [1, 2, 3, 4, 5, ...])
```

**Suggerimento**: Usa due puntatori: uno per la posizione di scrittura, uno per la lettura.

<details>
<summary>Soluzione</summary>

```
FUNZIONE RimuoviDuplicati(array, n)
    SE n ≤ 1:
        RITORNA n
    
    // write_idx indica dove scrivere il prossimo elemento unico
    write_idx ← 1
    
    PER read_idx da 1 a n-1:
        // Se elemento diverso dal precedente, copialo
        SE array[read_idx] ≠ array[read_idx - 1]:
            array[write_idx] ← array[read_idx]
            write_idx ← write_idx + 1
    
    RITORNA write_idx  // Numero di elementi unici
FINE

// Tracciamento per [1, 1, 2, 2, 2, 3]:
// read=1: 1=1, skip
// read=2: 2≠1, array[1]=2, write=2
// read=3: 2=2, skip
// read=4: 2=2, skip
// read=5: 3≠2, array[2]=3, write=3
// Risultato: [1, 2, 3, ...]

Complessità: O(N) tempo, O(1) spazio
```
</details>

### Esercizio 2.2: Ruota Array

**Problema**: Ruota un array a destra di k posizioni.

**Input**: Array e numero di rotazioni k
**Output**: Array ruotato

**Esempio**:
```
Input: [1, 2, 3, 4, 5], k=2
Output: [4, 5, 1, 2, 3]
```

<details>
<summary>Soluzione</summary>

```
FUNZIONE RuotaArray(array, n, k)
    // Normalizza k (k può essere > n)
    k ← k % n
    
    // Inverti tutto l'array
    Inverti(array, 0, n-1)
    
    // Inverti primi k elementi
    Inverti(array, 0, k-1)
    
    // Inverti elementi rimanenti
    Inverti(array, k, n-1)
FINE

FUNZIONE Inverti(array, inizio, fine)
    MENTRE inizio < fine:
        Scambia array[inizio] e array[fine]
        inizio ← inizio + 1
        fine ← fine - 1
FINE

// Esempio [1,2,3,4,5], k=2:
// 1. Inverti tutto: [5,4,3,2,1]
// 2. Inverti primi 2: [4,5,3,2,1]
// 3. Inverti ultimi 3: [4,5,1,2,3]

Complessità: O(N) tempo, O(1) spazio
```
</details>

### Esercizio 2.3: Trova Elemento Mancante

**Problema**: Dato un array contenente n numeri da 1 a n+1 con un numero mancante, trova il numero mancante.

**Input**: Array di n numeri
**Output**: Numero mancante

**Esempio**:
```
Input: [1, 2, 4, 5, 6]  (n=5, range 1-6)
Output: 3
```

<details>
<summary>Soluzione</summary>

```
FUNZIONE TrovaElementoMancante(array, n)
    // Somma attesa dei numeri da 1 a n+1
    somma_attesa ← (n + 1) * (n + 2) / 2
    
    // Somma effettiva degli elementi nell'array
    somma_reale ← 0
    PER i da 0 a n-1:
        somma_reale ← somma_reale + array[i]
    
    // La differenza è il numero mancante
    RITORNA somma_attesa - somma_reale
FINE

// Esempio [1,2,4,5,6]:
// n=5, range 1 a 6
// somma_attesa = 6*7/2 = 21
// somma_reale = 1+2+4+5+6 = 18
// mancante = 21-18 = 3

Complessità: O(N) tempo, O(1) spazio
```
</details>

## 🔴 Livello 3: Avanzato

### Esercizio 3.1: Two Sum

**Problema**: Trova due numeri in un array che sommati danno un target.

**Input**: Array e target
**Output**: Indici dei due numeri

**Esempio**:
```
Input: array = [2, 7, 11, 15], target = 9
Output: (0, 1)  // perché array[0] + array[1] = 2 + 7 = 9
```

<details>
<summary>Soluzione con Hash (O(N))</summary>

```
FUNZIONE TwoSum(array, n, target)
    // Map: valore → indice
    visti ← HashMap vuota
    
    PER i da 0 a n-1:
        complemento ← target - array[i]
        
        SE complemento è in visti:
            RITORNA (visti[complemento], i)
        
        visti[array[i]] ← i
    
    RITORNA -1  // Non trovato
FINE

// Tracciamento [2,7,11,15], target=9:
// i=0: complemento=9-2=7, non in visti
//      visti={2:0}
// i=1: complemento=9-7=2, trovato in visti!
//      RITORNA (0, 1)

Complessità: O(N) tempo, O(N) spazio
```

**Soluzione alternativa** (senza hash, O(N²)):
```
PER i da 0 a n-1:
    PER j da i+1 a n-1:
        SE array[i] + array[j] = target:
            RITORNA (i, j)
```
</details>

### Esercizio 3.2: Merge Intervalli

**Problema**: Dato un array di intervalli, fondi tutti gli intervalli sovrapposti.

**Input**: Array di intervalli [inizio, fine]
**Output**: Array di intervalli fusi

**Esempio**:
```
Input: [[1,3], [2,6], [8,10], [15,18]]
Output: [[1,6], [8,10], [15,18]]
```

<details>
<summary>Soluzione</summary>

```
FUNZIONE MergeIntervalli(intervalli, n)
    SE n = 0:
        RITORNA array vuoto
    
    // Ordina intervalli per punto di inizio
    Ordina(intervalli per primo elemento)
    
    risultato ← [intervalli[0]]
    
    PER i da 1 a n-1:
        ultimo ← ultimo intervallo in risultato
        corrente ← intervalli[i]
        
        // Se si sovrappongono
        SE corrente.inizio ≤ ultimo.fine:
            // Fondi: aggiorna fine dell'ultimo
            ultimo.fine ← MAX(ultimo.fine, corrente.fine)
        ALTRIMENTI:
            // Non si sovrappongono: aggiungi nuovo
            Aggiungi corrente a risultato
    
    RITORNA risultato
FINE

// Esempio [[1,3],[2,6],[8,10]]:
// Dopo ordinamento: [[1,3],[2,6],[8,10]]
// 
// risultato=[[1,3]]
// i=1: [2,6].inizio(2) ≤ [1,3].fine(3)
//      Fondi: [1,MAX(3,6)]=[1,6]
// i=2: [8,10].inizio(8) > [1,6].fine(6)
//      Aggiungi: risultato=[[1,6],[8,10]]

Complessità: O(N log N) per ordinamento
```
</details>

### Esercizio 3.3: Sottoarray con Somma Massima

**Problema**: Trova il sottoarray contiguo con la somma massima (Kadane's Algorithm).

**Input**: Array di numeri (possono essere negativi)
**Output**: Somma massima

**Esempio**:
```
Input: [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Output: 6  (sottoarray [4, -1, 2, 1])
```

<details>
<summary>Soluzione</summary>

```
FUNZIONE MaxSottoarraySomma(array, n)
    max_corrente ← array[0]
    max_globale ← array[0]
    
    PER i da 1 a n-1:
        // Scegli: estendi sottoarray o inizia nuovo
        max_corrente ← MAX(array[i], max_corrente + array[i])
        
        // Aggiorna massimo globale
        max_globale ← MAX(max_globale, max_corrente)
    
    RITORNA max_globale
FINE

// Tracciamento [-2,1,-3,4,-1,2,1,-5,4]:
// i=0: max_curr=-2, max_glob=-2
// i=1: max_curr=MAX(1,-2+1)=1, max_glob=1
// i=2: max_curr=MAX(-3,1-3)=-2, max_glob=1
// i=3: max_curr=MAX(4,-2+4)=4, max_glob=4
// i=4: max_curr=MAX(-1,4-1)=3, max_glob=4
// i=5: max_curr=MAX(2,3+2)=5, max_glob=5
// i=6: max_curr=MAX(1,5+1)=6, max_glob=6
// i=7: max_curr=MAX(-5,6-5)=1, max_glob=6
// i=8: max_curr=MAX(4,1+4)=5, max_glob=6

Complessità: O(N) tempo, O(1) spazio
```
</details>

## 🟣 Livello 4: Esperto

### Esercizio 4.1: Mediana di Due Array Ordinati

**Problema**: Trova la mediana di due array ordinati.

**Input**: Due array ordinati
**Output**: Mediana

**Esempio**:
```
Input: arr1 = [1, 3], arr2 = [2]
Output: 2.0

Input: arr1 = [1, 2], arr2 = [3, 4]
Output: 2.5  (media di 2 e 3)
```

<details>
<summary>Soluzione (Approccio Semplice O(N+M))</summary>

```
FUNZIONE TrovaMediana(arr1, n1, arr2, n2)
    // Fondi i due array ordinati
    merged ← Array di dimensione (n1 + n2)
    i ← 0, j ← 0, k ← 0
    
    MENTRE i < n1 AND j < n2:
        SE arr1[i] < arr2[j]:
            merged[k] ← arr1[i]
            i++
        ALTRIMENTI:
            merged[k] ← arr2[j]
            j++
        k++
    
    // Copia rimanenti
    MENTRE i < n1:
        merged[k] ← arr1[i]
        i++, k++
    
    MENTRE j < n2:
        merged[k] ← arr2[j]
        j++, k++
    
    // Calcola mediana
    totale ← n1 + n2
    SE totale % 2 = 1:
        RITORNA merged[totale / 2]
    ALTRIMENTI:
        mid1 ← merged[totale / 2 - 1]
        mid2 ← merged[totale / 2]
        RITORNA (mid1 + mid2) / 2.0
FINE

Complessità: O(N+M) tempo, O(N+M) spazio
```

**Nota**: Esiste soluzione O(log(min(N,M))) con ricerca binaria (molto avanzata!)
</details>

### Esercizio 4.2: N-Queens Problem

**Problema**: Posiziona N regine su una scacchiera N×N in modo che nessuna minacci un'altra.

**Input**: N (dimensione scacchiera)
**Output**: Una disposizione valida

<details>
<summary>Soluzione con Backtracking</summary>

```
FUNZIONE RisolviNQueens(n)
    board ← Matrice n×n inizializzata a 0
    SE PlaceQueen(board, 0, n):
        RITORNA board
    ALTRIMENTI:
        RITORNA "Nessuna soluzione"
FINE

FUNZIONE PlaceQueen(board, riga, n)
    // Caso base: tutte le regine piazzate
    SE riga = n:
        RITORNA Vero
    
    // Prova ogni colonna
    PER col da 0 a n-1:
        SE IsSafe(board, riga, col, n):
            // Piazza regina
            board[riga][col] ← 1
            
            // Ricorsione: prova riga successiva
            SE PlaceQueen(board, riga + 1, n):
                RITORNA Vero
            
            // Backtrack
            board[riga][col] ← 0
    
    RITORNA Falso
FINE

FUNZIONE IsSafe(board, riga, col, n)
    // Controlla colonna
    PER i da 0 a riga-1:
        SE board[i][col] = 1:
            RITORNA Falso
    
    // Controlla diagonale principale
    i ← riga - 1, j ← col - 1
    MENTRE i ≥ 0 AND j ≥ 0:
        SE board[i][j] = 1:
            RITORNA Falso
        i--, j--
    
    // Controlla diagonale secondaria
    i ← riga - 1, j ← col + 1
    MENTRE i ≥ 0 AND j < n:
        SE board[i][j] = 1:
            RITORNA Falso
        i--, j++
    
    RITORNA Vero
FINE

Complessità: O(N!) - molto costoso per N grande
```
</details>

## Progetti Completi

### Progetto 1: Calcolatrice con Stack

Implementa una calcolatrice che valuta espressioni matematiche usando uno stack.

**Esempio**:
```
Input: "3 + 4 * 2"
Output: 11
```

**Suggerimenti**:
- Usa due stack: uno per numeri, uno per operatori
- Implementa precedenza degli operatori
- Gestisci parentesi

### Progetto 2: Sistema di Scheduling

Implementa un algoritmo di scheduling che minimizza il tempo di attesa.

**Input**: Array di task con durata
**Output**: Ordine di esecuzione ottimale

**Suggerimento**: Ordina per durata crescente (Shortest Job First)

## Checklist di Auto-Valutazione

Dopo aver completato questi esercizi, dovresti essere in grado di:

✅ Analizzare la complessità di algoritmi
✅ Implementare algoritmi di ricerca e ordinamento
✅ Usare tecniche come two pointers, sliding window
✅ Applicare divide-et-impera
✅ Risolvere problemi con hash table
✅ Utilizzare backtracking per problemi combinatori
✅ Debuggare algoritmi sistematicamente
✅ Scegliere la struttura dati appropriata
✅ Ottimizzare soluzioni per tempo/spazio

## Risorse per Esercitarsi

### Online Judge
- LeetCode (leetcode.com)
- HackerRank (hackerrank.com)
- Codeforces (codeforces.com)

### Libri Consigliati
- "Algorithms" di Sedgewick e Wayne
- "Introduction to Algorithms" (CLRS)
- "Cracking the Coding Interview"

## Prossimi Passi

Ora sei pronto per iniziare il [Modulo 01 - Introduzione al C](../01_Introduzione/README.md)!

Con le competenze algoritmiche e di debugging acquisite in questo modulo, sarai molto più preparato per:
- Comprendere come implementare questi algoritmi in C
- Debuggare problemi di memoria e puntatori
- Ottimizzare codice C per performance
- Costruire applicazioni complesse

---

[← Debugging](10_debugging.md) | [Torna al Modulo](README.md) | [Inizia C →](../01_Introduzione/README.md)
