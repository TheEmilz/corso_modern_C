# Lezione 3: Algoritmi di Ricerca

## Introduzione

La **ricerca** è una delle operazioni più comuni in programmazione: trovare un elemento specifico in una collezione di dati. La scelta dell'algoritmo di ricerca può avere un impatto drammatico sulle performance della tua applicazione.

## Ricerca Lineare (Sequenziale)

L'algoritmo più semplice: controlla ogni elemento uno per uno.

### Pseudocodice

```
FUNZIONE RicercaLineare(array, n, target)
    PER i da 0 a n-1:
        SE array[i] = target:
            RITORNA i  // Trovato! Ritorna l'indice
    RITORNA -1  // Non trovato
FINE
```

### Esempio Passo-Passo

```
Array: [15, 3, 7, 21, 9, 12, 5]
Target: 21

Passo 1: array[0] = 15, 15 ≠ 21, continua
Passo 2: array[1] = 3,  3 ≠ 21, continua
Passo 3: array[2] = 7,  7 ≠ 21, continua
Passo 4: array[3] = 21, 21 = 21, TROVATO!

Ritorna: 3 (indice dell'elemento)
```

### Analisi della Complessità

- **Caso migliore**: O(1) - elemento è il primo
- **Caso peggiore**: O(N) - elemento è l'ultimo o non c'è
- **Caso medio**: O(N/2) ≈ O(N)
- **Spazio**: O(1) - nessuna memoria aggiuntiva

### Vantaggi e Svantaggi

✅ **Vantaggi**:
- Molto semplice da implementare
- Funziona su array non ordinati
- Nessun preprocessing richiesto
- Efficiente per array piccoli

❌ **Svantaggi**:
- Lento per array grandi
- Non sfrutta eventuali ordinamenti

### Quando Usarla

- Array piccoli (< 100 elementi)
- Dati non ordinati
- Ricerca occasionale (poche volte)
- Semplicità è priorità

## Ricerca Binaria

Algoritmo efficiente che richiede array **ordinato**. Divide ripetutamente lo spazio di ricerca a metà.

### Concetto Chiave

Come cercare in un dizionario:
1. Apri a metà
2. Se la parola è prima, cerca nella prima metà
3. Se la parola è dopo, cerca nella seconda metà
4. Ripeti fino a trovare

### Pseudocodice (Iterativo)

```
FUNZIONE RicercaBinaria(array_ordinato, n, target)
    sinistra ← 0
    destra ← n - 1
    
    MENTRE sinistra ≤ destra:
        medio ← (sinistra + destra) / 2
        
        SE array[medio] = target:
            RITORNA medio  // Trovato!
        
        ALTRIMENTI SE array[medio] < target:
            sinistra ← medio + 1  // Cerca a destra
        
        ALTRIMENTI:
            destra ← medio - 1    // Cerca a sinistra
    
    RITORNA -1  // Non trovato
FINE
```

### Esempio Dettagliato

```
Array ordinato: [2, 5, 8, 12, 16, 23, 38, 45, 56, 67, 78]
Target: 23

ITERAZIONE 1:
  sinistra=0, destra=10
  medio = (0+10)/2 = 5
  array[5] = 23
  23 = 23 → TROVATO!
  
Ritorna: 5

Numero di confronti: 1
```

**Altro Esempio**:

```
Array ordinato: [2, 5, 8, 12, 16, 23, 38, 45, 56, 67, 78]
Target: 45

ITERAZIONE 1:
  sinistra=0, destra=10
  medio = 5
  array[5] = 23 < 45
  → Cerca a destra: sinistra=6

ITERAZIONE 2:
  sinistra=6, destra=10
  medio = (6+10)/2 = 8
  array[8] = 56 > 45
  → Cerca a sinistra: destra=7

ITERAZIONE 3:
  sinistra=6, destra=7
  medio = (6+7)/2 = 6
  array[6] = 38 < 45
  → Cerca a destra: sinistra=7

ITERAZIONE 4:
  sinistra=7, destra=7
  medio = 7
  array[7] = 45 = 45 → TROVATO!

Ritorna: 7

Numero di confronti: 4
Con ricerca lineare: 8 confronti
```

### Visualizzazione

```
Ricerca di 45 in [2, 5, 8, 12, 16, 23, 38, 45, 56, 67, 78]

Passo 1: Cerca in tutto l'array
[2, 5, 8, 12, 16, |23|, 38, 45, 56, 67, 78]
                   ↑ medio=23, 23<45, vai a destra

Passo 2: Cerca in [38, 45, 56, 67, 78]
[38, 45, |56|, 67, 78]
          ↑ medio=56, 56>45, vai a sinistra

Passo 3: Cerca in [38, 45]
[38, |45|]
      ↑ medio=45, 45=45, TROVATO!
```

### Pseudocodice (Ricorsivo)

```
FUNZIONE RicercaBinariaRicorsiva(array, sinistra, destra, target)
    SE sinistra > destra:
        RITORNA -1  // Caso base: non trovato
    
    medio ← (sinistra + destra) / 2
    
    SE array[medio] = target:
        RITORNA medio  // Caso base: trovato
    
    SE array[medio] < target:
        // Ricorsione: cerca a destra
        RITORNA RicercaBinariaRicorsiva(array, medio+1, destra, target)
    
    ALTRIMENTI:
        // Ricorsione: cerca a sinistra
        RITORNA RicercaBinariaRicorsiva(array, sinistra, medio-1, target)
FINE
```

### Analisi della Complessità

- **Tempo**: O(log N) - divide a metà ogni passo
- **Spazio (iterativo)**: O(1)
- **Spazio (ricorsivo)**: O(log N) - stack delle chiamate

**Confronto**:
```
N = 1,000:
  Lineare: ~1,000 confronti (peggiore)
  Binaria: ~10 confronti

N = 1,000,000:
  Lineare: ~1,000,000 confronti
  Binaria: ~20 confronti

N = 1,000,000,000:
  Lineare: ~1,000,000,000 confronti
  Binaria: ~30 confronti
```

### Prerequisito: Array Ordinato

⚠️ **IMPORTANTE**: Ricerca binaria funziona SOLO su array ordinati!

```
// SBAGLIATO - Array non ordinato
array = [15, 3, 7, 21, 9]
RicercaBinaria(array, 5, 7)  // Potrebbe non trovare 7!

// CORRETTO - Array ordinato
array = [3, 7, 9, 15, 21]
RicercaBinaria(array, 5, 7)  // Trova 7
```

## Confronto Algoritmi

### Tabella Comparativa

| Caratteristica | Lineare | Binaria |
|---------------|---------|---------|
| Array ordinato | No | **Sì (obbligatorio)** |
| Complessità | O(N) | O(log N) |
| Implementazione | Facile | Media |
| Array piccoli | Buona | OK |
| Array grandi | Lenta | Eccellente |

### Quando Scegliere?

**Usa Ricerca Lineare se**:
- Array non ordinato e non conviene ordinarlo
- Array molto piccolo (< 50 elementi)
- Poche ricerche da fare
- Semplicità è priorità

**Usa Ricerca Binaria se**:
- Array già ordinato
- Molte ricerche da fare
- Array medio-grande
- Performance è priorità

### Analisi Costi

```
Scenario: Array di 10,000 elementi

Opzione A: Ricerca Lineare
  - Nessun preprocessing
  - 100 ricerche
  - Costo: 100 × 5,000 = 500,000 operazioni

Opzione B: Ordina + Ricerca Binaria
  - Ordinamento: ~133,000 operazioni (O(N log N))
  - 100 ricerche × 14 = 1,400 operazioni
  - Costo totale: 133,000 + 1,400 = 134,400 operazioni

Opzione B è 3.7x più veloce!
```

## Debug Passo-Passo

### Debug Ricerca Lineare

**Problema**: La funzione non trova elementi che esistono.

```
FUNZIONE RicercaLineare_Bug(array, n, target)
    PER i da 0 a n:  // BUG: va oltre l'ultimo elemento!
        SE array[i] = target:
            RITORNA i
    RITORNA -1
FINE
```

**Tracciamento**:
```
Array: [5, 10, 15], n=3, target=15

i=0: array[0]=5, 5≠15
i=1: array[1]=10, 10≠15
i=2: array[2]=15, 15=15, ritorna 2 ✓
i=3: array[3]=??? ERRORE! Fuori dai limiti!
```

**Fix**: `PER i da 0 a n-1`

### Debug Ricerca Binaria

**Problema Comune 1: Overflow**

```
// SBAGLIATO
medio ← (sinistra + destra) / 2

// Problema: sinistra+destra può causare overflow!
// Se sinistra=2,000,000,000 e destra=2,000,000,000
// somma = 4,000,000,000 > INT_MAX (overflow!)
```

**Fix**:
```
// CORRETTO
medio ← sinistra + (destra - sinistra) / 2
```

**Problema Comune 2: Loop Infinito**

```
// SBAGLIATO
MENTRE sinistra < destra:  // Dovrebbe essere ≤
    medio ← (sinistra + destra) / 2
    SE array[medio] = target:
        RITORNA medio
    ALTRIMENTI SE array[medio] < target:
        sinistra ← medio  // BUG: dovrebbe essere medio+1
    ALTRIMENTI:
        destra ← medio    // BUG: dovrebbe essere medio-1
```

**Tracciamento**:
```
Array: [1, 2, 3], target=2

Iterazione 1:
  sinistra=0, destra=2
  medio=1
  array[1]=2, trovato! ✓

Tutto ok... proviamo con target=4 (non presente)

Iterazione 1:
  sinistra=0, destra=2
  medio=1, array[1]=2<4
  sinistra=1  // BUG! Dovrebbe essere 2

Iterazione 2:
  sinistra=1, destra=2
  medio=1, array[1]=2<4
  sinistra=1  // Stesso valore! LOOP INFINITO!

Il loop non termina mai perché sinistra non cambia
```

**Fix**:
```
sinistra ← medio + 1
destra ← medio - 1
```

## Varianti di Ricerca

### Ricerca del Primo Occorrenza

Se l'array ha duplicati, trova il **primo** elemento uguale al target.

```
FUNZIONE RicercaPrimaOccorrenza(array, n, target)
    risultato ← -1
    sinistra ← 0
    destra ← n - 1
    
    MENTRE sinistra ≤ destra:
        medio ← sinistra + (destra - sinistra) / 2
        
        SE array[medio] = target:
            risultato ← medio
            destra ← medio - 1  // Continua a cercare a sinistra
        
        ALTRIMENTI SE array[medio] < target:
            sinistra ← medio + 1
        
        ALTRIMENTI:
            destra ← medio - 1
    
    RITORNA risultato
FINE
```

**Esempio**:
```
Array: [1, 2, 2, 2, 3, 4, 5]
Target: 2

Ricerca binaria normale: potrebbe ritornare indice 1, 2, o 3
Ricerca prima occorrenza: ritorna sempre indice 1
```

### Ricerca dell'Ultima Occorrenza

```
FUNZIONE RicercaUltimaOccorrenza(array, n, target)
    risultato ← -1
    sinistra ← 0
    destra ← n - 1
    
    MENTRE sinistra ≤ destra:
        medio ← sinistra + (destra - sinistra) / 2
        
        SE array[medio] = target:
            risultato ← medio
            sinistra ← medio + 1  // Continua a cercare a destra
        
        ALTRIMENTI SE array[medio] < target:
            sinistra ← medio + 1
        
        ALTRIMENTI:
            destra ← medio - 1
    
    RITORNA risultato
FINE
```

## Esercizi Pratici

### Esercizio 1: Implementa Ricerca Lineare

Implementa (in pseudocodice) una funzione di ricerca lineare che ritorna:
- L'indice se trovato
- -1 se non trovato

Testa con:
- Array: [10, 20, 30, 40, 50]
- Target: 30 (dovrebbe ritornare 2)
- Target: 60 (dovrebbe ritornare -1)

### Esercizio 2: Traccia Ricerca Binaria

Traccia passo-passo la ricerca binaria per:
- Array: [1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
- Target: 13

Scrivi ogni iterazione con valori di sinistra, destra, medio.

### Esercizio 3: Trova il Bug

```
FUNZIONE RicercaBinaria_Bug(array, n, target)
    sinistra ← 0
    destra ← n
    
    MENTRE sinistra < destra:
        medio ← (sinistra + destra) / 2
        
        SE array[medio] = target:
            RITORNA medio
        
        SE array[medio] < target:
            sinistra ← medio + 1
        ALTRIMENTI:
            destra ← medio
    
    RITORNA -1
FINE
```

Ci sono almeno 2 bug. Quali sono?

<details>
<summary>Suggerimento</summary>

1. Controlla l'inizializzazione di `destra`
2. Controlla la condizione del MENTRE
</details>

### Esercizio 4: Conta Occorrenze

Scrivi un algoritmo che conta quante volte appare `target` in un array ordinato usando ricerca binaria modificata.

**Suggerimento**: Usa RicercaPrimaOccorrenza e RicercaUltimaOccorrenza.

## Punti Chiave

✅ Ricerca lineare: O(N), funziona su qualsiasi array
✅ Ricerca binaria: O(log N), richiede array ordinato
✅ Ricerca binaria è **esponenzialmente** più veloce per array grandi
✅ Controlla sempre overflow nel calcolo del medio
✅ Usa `medio+1` e `medio-1` per evitare loop infiniti
✅ Se fai molte ricerche, conviene ordinare prima
✅ Per array piccoli (<50), lineare può essere più semplice

## Regole del Debug

1. **Testa casi limite**: array vuoto, un elemento, target non presente
2. **Traccia valori**: stampa sinistra, destra, medio ogni iterazione
3. **Verifica prerequisiti**: array ordinato per ricerca binaria
4. **Controlla terminazione**: il loop deve sempre terminare

---

[← Complessità](02_complessita.md) | [Torna al Modulo](README.md) | [Prossimo: Ordinamento →](04_ordinamento.md)
