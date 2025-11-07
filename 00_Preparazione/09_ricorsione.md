# Lezione 9 - Ricorsione

## 🎯 Obiettivi della Lezione

Alla fine di questa lezione sarai in grado di:
- Comprendere il concetto di ricorsione e come funziona
- Identificare caso base e caso ricorsivo
- Visualizzare lo stack delle chiamate ricorsive
- Convertire tra ricorsione e iterazione
- Debuggare problemi ricorsivi (stack overflow, infinite loop)
- Riconoscere quando usare ricorsione vs iterazione

---

## 1. PROBLEMA: Definizioni Autoriferenti

### Scenario: Fattoriale

Come definiresti il fattoriale di n (n!)?

```
DEFINIZIONE ITERATIVA:
n! = n × (n-1) × (n-2) × ... × 2 × 1

5! = 5 × 4 × 3 × 2 × 1 = 120

MA ESISTE UN PATTERN:
5! = 5 × (4!)
4! = 4 × (3!)
3! = 3 × (2!)
2! = 2 × (1!)
1! = 1
```

**OSSERVAZIONE:** n! è definito in termini di (n-1)!

**DEFINIZIONE RICORSIVA:**
```
n! = {
    1           se n = 0 o n = 1  ← CASO BASE
    n × (n-1)!  se n > 1           ← CASO RICORSIVO
}
```

Questa è **RICORSIONE**: una funzione definita in termini di se stessa!

---

## 2. ANATOMIA DELLA RICORSIONE

### 2.1 Componenti Essenziali

```
OGNI FUNZIONE RICORSIVA DEVE AVERE:

1. CASO BASE (Base Case)
   - Condizione di terminazione
   - Risposta DIRETTA senza ricorsione
   - DEVE esistere, altrimenti → ricorsione infinita!

2. CASO RICORSIVO (Recursive Case)
   - Chiamata alla funzione stessa
   - Con input "più piccolo" o "più semplice"
   - DEVE avvicinarsi al caso base!

3. PROGRESSIONE VERSO IL CASO BASE
   - Ogni chiamata ricorsiva deve avvicinare al caso base
   - Altrimenti → ricorsione infinita!
```

### 2.2 Esempio: Fattoriale

```
FUNZIONE fattoriale(n):
    // CASO BASE
    SE n == 0 O n == 1:
        RETURN 1
    
    // CASO RICORSIVO
    ALTRIMENTI:
        RETURN n × fattoriale(n - 1)

TRACCIAMENTO di fattoriale(5):

fattoriale(5)
│
├─ 5 × fattoriale(4)
│      │
│      ├─ 4 × fattoriale(3)
│      │      │
│      │      ├─ 3 × fattoriale(2)
│      │      │      │
│      │      │      ├─ 2 × fattoriale(1)
│      │      │      │      │
│      │      │      │      └─ CASO BASE! → 1
│      │      │      │
│      │      │      └─ 2 × 1 = 2
│      │      │
│      │      └─ 3 × 2 = 6
│      │
│      └─ 4 × 6 = 24
│
└─ 5 × 24 = 120

RISULTATO: 120
```

---

## 3. LO STACK DELLE CHIAMATE (Call Stack)

### 3.1 Come Funziona in Memoria

```
CODICE:
int fattoriale(int n) {
    if (n <= 1) return 1;
    return n * fattoriale(n - 1);
}

CHIAMATA: fattoriale(4)

STACK DURANTE L'ESECUZIONE:

┌─ FASE 1: ESPANSIONE (PUSH) ────────────────────┐

CHIAMATA fattoriale(4):
┌──────────────────┐
│ fattoriale(4)    │ ← TOP
│ n = 4            │
│ return: pending  │
└──────────────────┘

CHIAMA fattoriale(3):
┌──────────────────┐
│ fattoriale(3)    │ ← TOP
│ n = 3            │
│ return: pending  │
├──────────────────┤
│ fattoriale(4)    │
│ n = 4            │
│ return: pending  │
└──────────────────┘

CHIAMA fattoriale(2):
┌──────────────────┐
│ fattoriale(2)    │ ← TOP
│ n = 2            │
│ return: pending  │
├──────────────────┤
│ fattoriale(3)    │
│ n = 3            │
├──────────────────┤
│ fattoriale(4)    │
│ n = 4            │
└──────────────────┘

CHIAMA fattoriale(1):
┌──────────────────┐
│ fattoriale(1)    │ ← TOP
│ n = 1            │
│ return: 1 ✓      │ ← CASO BASE!
├──────────────────┤
│ fattoriale(2)    │
│ n = 2            │
├──────────────────┤
│ fattoriale(3)    │
│ n = 3            │
├──────────────────┤
│ fattoriale(4)    │
│ n = 4            │
└──────────────────┘

┌─ FASE 2: CONTRAZIONE (POP) ─────────────────────┐

fattoriale(1) ritorna 1:
┌──────────────────┐
│ fattoriale(2)    │ ← TOP
│ n = 2            │
│ return: 2×1 = 2  │
├──────────────────┤
│ fattoriale(3)    │
│ n = 3            │
├──────────────────┤
│ fattoriale(4)    │
│ n = 4            │
└──────────────────┘

fattoriale(2) ritorna 2:
┌──────────────────┐
│ fattoriale(3)    │ ← TOP
│ n = 3            │
│ return: 3×2 = 6  │
├──────────────────┤
│ fattoriale(4)    │
│ n = 4            │
└──────────────────┘

fattoriale(3) ritorna 6:
┌──────────────────┐
│ fattoriale(4)    │ ← TOP
│ n = 4            │
│ return: 4×6 = 24 │
└──────────────────┘

fattoriale(4) ritorna 24:
STACK VUOTO → RISULTATO: 24
```

### 3.2 Costo della Ricorsione

```
OGNI CHIAMATA RICORSIVA:
✓ Alloca stack frame (variabili locali, indirizzo ritorno)
✓ Salva contesto corrente
✓ Salta al nuovo codice

MEMORIA STACK:
- fattoriale(n) usa O(n) stack frames
- Ogni frame: alcuni byte
- Stack limitato (tipicamente 1-8 MB)

RISCHIO: STACK OVERFLOW se troppa profondità!
```

---

## 4. ESEMPI CLASSICI

### 4.1 Fibonacci

```
DEFINIZIONE:
Fibonacci(n) = {
    0              se n = 0
    1              se n = 1
    Fib(n-1) + Fib(n-2)  se n > 1
}

Sequenza: 0, 1, 1, 2, 3, 5, 8, 13, 21, ...

FUNZIONE fibonacci(n):
    // CASI BASE
    SE n == 0:
        RETURN 0
    SE n == 1:
        RETURN 1
    
    // CASO RICORSIVO
    RETURN fibonacci(n-1) + fibonacci(n-2)

TRACCIAMENTO di fibonacci(5):

                    fib(5)
                   /      \
              fib(4)      fib(3)
             /     \       /    \
        fib(3)  fib(2)  fib(2) fib(1)
        /   \    /  \    /  \
    fib(2) fib(1) f(1) f(0) f(1) f(0)
    /  \
 fib(1) fib(0)

PROBLEMA: MOLTISSIME CHIAMATE RIPETUTE!
fib(3) calcolato 2 volte
fib(2) calcolato 3 volte
...

COMPLESSITÀ: O(2^n) ⚠️ ESPONENZIALE!

SOLUZIONE: Memoization o programmazione dinamica
(memorizza risultati già calcolati)
```

### 4.2 Somma Array

```
PROBLEMA: Somma tutti gli elementi di un array

APPROCCIO RICORSIVO:
sum(arr, n) = {
    0                se n = 0  ← array vuoto
    arr[n-1] + sum(arr, n-1)   ← ultimo elemento + somma dei restanti
}

FUNZIONE somma_array(arr, n):
    // CASO BASE
    SE n == 0:
        RETURN 0
    
    // CASO RICORSIVO
    RETURN arr[n-1] + somma_array(arr, n-1)

ESEMPIO: somma_array([1,2,3,4], 4)

somma([1,2,3,4], 4)
= 4 + somma([1,2,3], 3)
= 4 + (3 + somma([1,2], 2))
= 4 + (3 + (2 + somma([1], 1)))
= 4 + (3 + (2 + (1 + somma([], 0))))
= 4 + (3 + (2 + (1 + 0)))
= 4 + (3 + (2 + 1))
= 4 + (3 + 3)
= 4 + 6
= 10 ✓
```

### 4.3 Ricerca Binaria Ricorsiva

```
FUNZIONE ricerca_binaria_ricorsiva(arr, target, left, right):
    // CASO BASE 1: Intervallo vuoto
    SE left > right:
        RETURN -1  // Non trovato
    
    mid = (left + right) / 2
    
    // CASO BASE 2: Elemento trovato
    SE arr[mid] == target:
        RETURN mid
    
    // CASO RICORSIVO
    SE target < arr[mid]:
        // Cerca nella metà sinistra
        RETURN ricerca_binaria_ricorsiva(arr, target, left, mid-1)
    ALTRIMENTI:
        // Cerca nella metà destra
        RETURN ricerca_binaria_ricorsiva(arr, target, mid+1, right)

ESEMPIO: Cerca 7 in [1, 3, 5, 7, 9, 11]

ricerca([1,3,5,7,9,11], 7, 0, 5)
  mid = 2, arr[2] = 5
  7 > 5 → ricerca([1,3,5,7,9,11], 7, 3, 5)
    mid = 4, arr[4] = 9
    7 < 9 → ricerca([1,3,5,7,9,11], 7, 3, 3)
      mid = 3, arr[3] = 7
      7 == 7 → RETURN 3 ✓

COMPLESSITÀ: O(log n) - stessa della versione iterativa!
```

### 4.4 Torre di Hanoi

```
PROBLEMA: Spostare n dischi da torre A a torre C usando torre B

REGOLE:
1. Muovi un disco alla volta
2. Disco più grande non può stare sopra disco più piccolo

SOLUZIONE RICORSIVA:
- Sposta n-1 dischi da A a B (usando C)
- Sposta disco n da A a C
- Sposta n-1 dischi da B a C (usando A)

FUNZIONE hanoi(n, from, to, aux):
    // CASO BASE
    SE n == 1:
        STAMPA "Sposta disco 1 da", from, "a", to
        RETURN
    
    // CASO RICORSIVO
    hanoi(n-1, from, aux, to)      // Step 1
    STAMPA "Sposta disco", n, "da", from, "a", to  // Step 2
    hanoi(n-1, aux, to, from)      // Step 3

ESEMPIO: hanoi(3, 'A', 'C', 'B')

OUTPUT:
Sposta disco 1 da A a C
Sposta disco 2 da A a B
Sposta disco 1 da C a B
Sposta disco 3 da A a C
Sposta disco 1 da B a A
Sposta disco 2 da B a C
Sposta disco 1 da A a C

COMPLESSITÀ: O(2^n) - 2^n - 1 mosse per n dischi
```

---

## 5. RICORSIONE vs ITERAZIONE

### 5.1 Confronto

```
┌─────────────────┬───────────────────┬────────────────────┐
│ CARATTERISTICA  │    RICORSIONE     │    ITERAZIONE      │
├─────────────────┼───────────────────┼────────────────────┤
│ Leggibilità     │ ✓ Spesso elegante │ ✗ Più verbosa      │
│ Memoria         │ ✗ O(n) stack      │ ✓ O(1) tipicamente │
│ Performance     │ ✗ Overhead call   │ ✓ Più veloce       │
│ Problemi dividi │ ✓ Naturale        │ ✗ Meno naturale    │
│ e conquista     │                   │                    │
│ Alberi/grafi    │ ✓ Molto naturale  │ ✗ Serve stack/queue│
│ Debug           │ ✗ Stack complesso │ ✓ Più semplice     │
│ Stack overflow  │ ✗ Possibile       │ ✓ No rischio       │
└─────────────────┴───────────────────┴────────────────────┘

🎯 USA RICORSIONE QUANDO:
  ✅ Problema naturalmente ricorsivo (alberi, grafi)
  ✅ Divide-and-conquer (merge sort, quick sort)
  ✅ Backtracking
  ✅ Codice più leggibile
  ✅ Profondità limitata

🎯 USA ITERAZIONE QUANDO:
  ✅ Performance critica
  ✅ Memoria limitata
  ✅ Profondità potenzialmente alta
  ✅ Problema lineare semplice
```

### 5.2 Conversione Ricorsione → Iterazione

**FATTORIALE RICORSIVO:**
```
FUNZIONE fattoriale_ricorsivo(n):
    SE n <= 1:
        RETURN 1
    RETURN n × fattoriale_ricorsivo(n-1)
```

**FATTORIALE ITERATIVO:**
```
FUNZIONE fattoriale_iterativo(n):
    risultato = 1
    PER i DA 2 A n:
        risultato = risultato × i
    RETURN risultato

VANTAGGI:
✓ Usa O(1) memoria (no stack)
✓ Più veloce (no overhead chiamate)
```

**FIBONACCI RICORSIVO CON MEMOIZATION:**
```
memo = ARRAY di dimensione n, inizializzato a -1

FUNZIONE fibonacci_memo(n):
    // CASI BASE
    SE n == 0:
        RETURN 0
    SE n == 1:
        RETURN 1
    
    // Controlla se già calcolato
    SE memo[n] != -1:
        RETURN memo[n]
    
    // Calcola e memorizza
    memo[n] = fibonacci_memo(n-1) + fibonacci_memo(n-2)
    RETURN memo[n]

COMPLESSITÀ: O(n) invece di O(2^n)!
Ogni fib(i) calcolato UNA SOLA VOLTA.
```

---

## 6. ⚠️ ERRORI MORTALI

### ERRORE #1: Nessun Caso Base

```
// ❌ DISASTRO!
int countdown(int n) {
    printf("%d\n", n);
    return countdown(n - 1);  // NESSUN CASO BASE!
}

/* COSA SUCCEDE:
 * countdown(3) → countdown(2) → countdown(1) → countdown(0) →
 * countdown(-1) → countdown(-2) → ... → INFINITO!
 * 
 * STACK:
 * ┌────────────┐
 * │ countdown  │
 * ├────────────┤
 * │ countdown  │
 * ├────────────┤
 * │ ...        │ ← Cresce all'infinito
 * ├────────────┤
 * │ countdown  │
 * └────────────┘
 * 
 * RISULTATO: STACK OVERFLOW! Segmentation fault
 */

// ✅ SOLUZIONE: Aggiungi caso base
int countdown(int n) {
    if (n <= 0) {  // ← CASO BASE!
        printf("Blastoff!\n");
        return 0;
    }
    printf("%d\n", n);
    return countdown(n - 1);
}
```

### ERRORE #2: Non Avvicinarsi al Caso Base

```
// ❌ ERRORE: Incrementa invece di decrementare
int fattoriale(int n) {
    if (n == 0) return 1;
    return n * fattoriale(n + 1);  // ← BUG! n+1 invece di n-1
}

/* COSA SUCCEDE:
 * fattoriale(5) → fattoriale(6) → fattoriale(7) → ...
 * NON raggiungerà MAI il caso base (n == 0)!
 * STACK OVERFLOW!
 */

// ✅ CORRETTO:
int fattoriale(int n) {
    if (n == 0) return 1;
    return n * fattoriale(n - 1);  // ✓ Decrementa!
}
```

### ERRORE #3: Stack Overflow su Input Grandi

```
// ❌ PROBLEMA: Ricorsione troppo profonda
int fibonacci(int n) {
    if (n <= 1) return n;
    return fibonacci(n-1) + fibonacci(n-2);
}

// fibonacci(50) → TROPPO LENTO + possibile stack overflow!

/* STACK per fib(50):
 * Profondità max ≈ 50 (OK)
 * MA numero chiamate ≈ 2^50 (IMPOSSIBILE!)
 * 
 * Tempo: MINUTI/ORE!
 */

// ✅ SOLUZIONE 1: Memoization
int memo[MAX_N];
int fibonacci_memo(int n) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    memo[n] = fibonacci_memo(n-1) + fibonacci_memo(n-2);
    return memo[n];
}

// ✅ SOLUZIONE 2: Iterazione
int fibonacci_iter(int n) {
    if (n <= 1) return n;
    int prev = 0, curr = 1;
    for (int i = 2; i <= n; i++) {
        int next = prev + curr;
        prev = curr;
        curr = next;
    }
    return curr;
}
```

### ERRORE #4: Modificare Parametri Globali

```
// ❌ PROBLEMA: Usa variabile globale
int counter = 0;

void increment_recursive(int n) {
    if (n == 0) return;
    counter++;  // Modifica globale
    increment_recursive(n - 1);
}

/* PROBLEMA:
 * Difficile da debuggare
 * Effetti collaterali (side effects)
 * Non riutilizzabile
 * Non thread-safe
 */

// ✅ MEGLIO: Usa return value
int count_recursive(int n) {
    if (n == 0) return 0;
    return 1 + count_recursive(n - 1);
}
```

---

## 7. 🔧 ESERCIZI

### 🟢 LIVELLO 1 - Comprensione

**Esercizio 1:** Traccia l'esecuzione di `potenza(2, 3)`:
```
potenza(base, exp) = {
    1                    se exp = 0
    base × potenza(base, exp-1)  se exp > 0
}
```

<details>
<summary>Soluzione</summary>

```
potenza(2, 3)
= 2 × potenza(2, 2)
= 2 × (2 × potenza(2, 1))
= 2 × (2 × (2 × potenza(2, 0)))
= 2 × (2 × (2 × 1))          ← CASO BASE
= 2 × (2 × 2)
= 2 × 4
= 8 ✓

STACK:
┌─────────────┐
│ potenza(2,0)│ → 1
├─────────────┤
│ potenza(2,1)│ → 2×1 = 2
├─────────────┤
│ potenza(2,2)│ → 2×2 = 4
├─────────────┤
│ potenza(2,3)│ → 2×4 = 8
└─────────────┘
```
</details>

**Esercizio 2:** Cosa stampa questo codice?
```
void stampa(int n) {
    if (n == 0) return;
    printf("%d ", n);
    stampa(n - 1);
    printf("%d ", n);
}

stampa(3);
```

<details>
<summary>Soluzione</summary>

```
TRACCIAMENTO:
stampa(3): print 3 → stampa(2) → print 3
  stampa(2): print 2 → stampa(1) → print 2
    stampa(1): print 1 → stampa(0) → print 1
      stampa(0): return

OUTPUT: 3 2 1 1 2 3

SPIEGAZIONE:
Stampa n PRIMA della chiamata ricorsiva → 3 2 1 (andata)
Stampa n DOPO la chiamata ricorsiva → 1 2 3 (ritorno)
```
</details>

### 🟡 LIVELLO 2 - Applicazione

**Esercizio 3:** Scrivi funzione ricorsiva per invertire una stringa.

<details>
<summary>Soluzione</summary>

```
FUNZIONE inverti(str, start, end):
    // CASO BASE
    SE start >= end:
        RETURN
    
    // CASO RICORSIVO
    // Scambia str[start] con str[end]
    temp = str[start]
    str[start] = str[end]
    str[end] = temp
    
    // Ricorri su sottostringa interna
    inverti(str, start+1, end-1)

ESEMPIO: inverti("HELLO", 0, 4)
inverti("HELLO", 0, 4): scambia H↔O → "OELLH"
  inverti("OELLH", 1, 3): scambia E↔L → "OLELO"
    inverti("OLELO", 2, 2): start==end → RETURN
  RETURN
RETURN

RISULTATO: "OLLEH" ✓
```
</details>

### 🟠 LIVELLO 3 - Analisi

**Esercizio 4:** Calcola la complessità di questa funzione:
```
int misterio(int n) {
    if (n <= 1) return 1;
    return misterio(n/2) + misterio(n/2);
}
```

<details>
<summary>Soluzione</summary>

```
ANALISI:
- Ogni chiamata divide n per 2
- Due chiamate ricorsive per livello

ALBERO RICORSIONE:
                misterio(n)
               /           \
        misterio(n/2)   misterio(n/2)
         /      \          /      \
    mis(n/4) mis(n/4) mis(n/4) mis(n/4)
    ...

PROFONDITÀ: log n (dimezza ogni volta)
NODI PER LIVELLO:
- Livello 0: 1 nodo
- Livello 1: 2 nodi
- Livello 2: 4 nodi
- ...
- Livello k: 2^k nodi

TOTALE NODI: 1 + 2 + 4 + ... + 2^(log n)
           = 2^(log n + 1) - 1
           = 2n - 1

COMPLESSITÀ: O(n)

NOTA: Questa funzione ha ridondanza!
misterio(n/2) calcolato DUE VOLTE identiche.
Potrebbe essere O(log n) calcolando una sola volta:

int misterio_ottimizzato(int n) {
    if (n <= 1) return 1;
    int temp = misterio_ottimizzato(n/2);
    return temp + temp;  // O(log n)
}
```
</details>

### 🔴 LIVELLO 4 - Sintesi

**Esercizio 5:** Implementa merge sort ricorsivo.

<details>
<summary>Soluzione</summary>

```
FUNZIONE merge_sort(arr, left, right):
    // CASO BASE: array di 0 o 1 elemento
    SE left >= right:
        RETURN
    
    // CASO RICORSIVO
    mid = (left + right) / 2
    
    // Divide
    merge_sort(arr, left, mid)      // Ordina metà sinistra
    merge_sort(arr, mid+1, right)   // Ordina metà destra
    
    // Conquista
    merge(arr, left, mid, right)    // Unisci le due metà

FUNZIONE merge(arr, left, mid, right):
    // Crea array temporanei
    n1 = mid - left + 1
    n2 = right - mid
    
    left_arr[n1]
    right_arr[n2]
    
    // Copia dati
    PER i DA 0 A n1-1:
        left_arr[i] = arr[left + i]
    PER j DA 0 A n2-1:
        right_arr[j] = arr[mid + 1 + j]
    
    // Merge
    i = 0, j = 0, k = left
    
    MENTRE i < n1 E j < n2:
        SE left_arr[i] <= right_arr[j]:
            arr[k] = left_arr[i]
            i++
        ALTRIMENTI:
            arr[k] = right_arr[j]
            j++
        k++
    
    // Copia rimanenti
    MENTRE i < n1:
        arr[k] = left_arr[i]
        i++, k++
    
    MENTRE j < n2:
        arr[k] = right_arr[j]
        j++, k++

ESEMPIO: merge_sort([38, 27, 43, 3], 0, 3)

                [38, 27, 43, 3]
               /               \
         [38, 27]             [43, 3]
         /     \              /     \
      [38]    [27]         [43]    [3]
        \     /              \     /
       [27, 38]             [3, 43]
           \                 /
          [3, 27, 38, 43]

COMPLESSITÀ:
- Profondità: O(log n)
- Ogni livello: O(n) per merge
- TOTALE: O(n log n) ✓
```
</details>

---

## 8. 🎓 PUNTI CHIAVE

**RICORSIONE:**
- ✅ Elegante per problemi naturalmente ricorsivi
- ✅ Essenziale per alberi, grafi, divide-and-conquer
- ⚠️ Richiede SEMPRE caso base
- ⚠️ Usa memoria stack: O(profondità)
- ⚠️ Rischio stack overflow

**COMPONENTI FONDAMENTALI:**
1. Caso base (terminazione)
2. Caso ricorsivo (chiamata a se stessa)
3. Progressione verso caso base

**OTTIMIZZAZIONI:**
- Memoization: memorizza risultati già calcolati
- Tail recursion: alcune ottimizzazioni del compilatore
- Conversione a iterazione: se performance critiche

---

## 9. DEBUG: Problemi Comuni

### Problema 1: Stack Overflow

```
SINTOMO: "Segmentation fault" o "Stack overflow"

CAUSA:
1. Nessun caso base
2. Caso base mai raggiunto
3. Input troppo grande

DEBUG:
1. Aggiungi print per tracciare chiamate
2. Verifica caso base esiste e viene raggiunto
3. Limita profondità ricorsione (failsafe)
4. Converti a iterazione per input grandi
```

### Problema 2: Performance Scadenti

```
SINTOMO: Funzione ricorsiva lentissima

CAUSA: Ridondanza (es: fibonacci naive)

SOLUZIONE:
1. Memoization (memorizza risultati)
2. Bottom-up DP invece di top-down ricorsivo
3. Converti a iterazione
```

---

## 📚 Prossimi Passi

Hai completato le strutture dati fondamentali! Ora sei pronto per:
- [Lezione 10: Metodologia di Debug](10_debugging.md) - Debug sistematico
- [Modulo 01: Introduzione al C](../01_Introduzione/README.md) - Inizia con C!

---

[← Hash Table](08_hash_table.md) | [Torna al Modulo](README.md) | [Debugging →](10_debugging.md)
