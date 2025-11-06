# Modulo 03 - Livello Intermedio

## Puntatori

### Introduzione ai Puntatori

I puntatori sono uno degli aspetti più potenti e più pericolosi del linguaggio C. Un puntatore è una variabile che contiene l'indirizzo di memoria di un'altra variabile.

**Perché i puntatori sono importanti:**
- Permettono l'allocazione dinamica della memoria
- Consentono passaggio per riferimento alle funzioni
- Permettono di creare strutture dati complesse (liste, alberi, grafi)
- Essenziali per lavorare con stringhe e array
- Fondamentali per comprendere come funziona la memoria

```c
#include <stdio.h>

int main(void)
{
    int x = 42;
    int *ptr;  // Dichiarazione di un puntatore a int
    
    ptr = &x;  // & = operatore "indirizzo di"
    
    printf("Valore di x: %d\n", x);
    printf("Indirizzo di x: %p\n", (void*)&x);
    printf("Valore di ptr: %p\n", (void*)ptr);
    printf("Valore puntato da ptr: %d\n", *ptr);  // * = dereferenziazione
    
    /* Modificare attraverso il puntatore */
    *ptr = 100;
    printf("Nuovo valore di x: %d\n", x);  // 100!
    
    return 0;
}
```

#### Spiegazione Dettagliata con Diagramma di Memoria

**Dichiarazione e inizializzazione:**
```c
int x = 42;
int *ptr;
ptr = &x;
```

**Rappresentazione in memoria:**
```
Variabile    Indirizzo    Valore
────────────────────────────────
x            0x1000       42
ptr          0x2000       0x1000  (indirizzo di x)
```

**Spiegazione visiva:**
```
     ptr                    x
   ┌──────┐              ┌──────┐
   │0x1000│─────────────>│  42  │
   └──────┘              └──────┘
   0x2000                0x1000
```

**Analisi linea per linea:**

1. **`int x = 42;`**
   - Crea una variabile intera `x` in memoria
   - Assegna il valore 42
   - Il sistema operativo alloca 4 byte (su sistemi a 32/64 bit)
   - Supponiamo che `x` sia all'indirizzo 0x1000

2. **`int *ptr;`**
   - Dichiara un puntatore a intero
   - `int *`: Significa "puntatore a int"
   - `ptr` può contenere l'indirizzo di una variabile `int`
   - Inizialmente contiene un valore casuale (garbage) - NON usarlo prima di inizializzarlo!

3. **`ptr = &x;`**
   - `&x`: Operatore "address-of" - ottiene l'indirizzo di `x`
   - Assegna l'indirizzo di `x` a `ptr`
   - Ora `ptr` "punta a" `x`

4. **`printf("Valore di x: %d\n", x);`**
   - Accesso diretto a `x`
   - Output: `Valore di x: 42`

5. **`printf("Indirizzo di x: %p\n", (void*)&x);`**
   - `%p`: Format specifier per puntatore
   - `&x`: Indirizzo di x
   - `(void*)`: Cast a void pointer (convenzione per `%p`)
   - Output: `Indirizzo di x: 0x1000` (esempio)

6. **`printf("Valore di ptr: %p\n", (void*)ptr);`**
   - Stampa il valore contenuto in `ptr` (che è un indirizzo)
   - Output: `Valore di ptr: 0x1000` (stesso di `&x`)

7. **`printf("Valore puntato da ptr: %d\n", *ptr);`**
   - `*ptr`: Operatore di dereferenziazione
   - "Vai all'indirizzo contenuto in `ptr` e leggi il valore"
   - Equivalente a leggere `x`
   - Output: `Valore puntato da ptr: 42`

8. **`*ptr = 100;`**
   - Dereferenzia `ptr` e assegna 100
   - Modifica il valore all'indirizzo puntato
   - Poiché `ptr` punta a `x`, modifica `x`!

9. **`printf("Nuovo valore di x: %d\n", x);`**
   - `x` ora vale 100 (modificato tramite `ptr`)
   - Output: `Nuovo valore di x: 100`

**Diagramma "prima e dopo":**

**PRIMA di `*ptr = 100;`:**
```
     ptr                    x
   ┌──────┐              ┌──────┐
   │0x1000│─────────────>│  42  │
   └──────┘              └──────┘
```

**DOPO `*ptr = 100;`:**
```
     ptr                    x
   ┌──────┐              ┌──────┐
   │0x1000│─────────────>│ 100  │
   └──────┘              └──────┘
```

#### Due Operatori Fondamentali

**Operatore `&` (Address-of):**
- Ottiene l'indirizzo di una variabile
- `&variabile` → indirizzo della variabile
- Esempio: Se `x` è a 0x1000, allora `&x` è 0x1000

**Operatore `*` (Dereferenziazione):**
- Accede al valore all'indirizzo puntato
- `*puntatore` → valore all'indirizzo contenuto nel puntatore
- Due usi diversi:
  1. **Dichiarazione**: `int *ptr;` (dichiara un puntatore)
  2. **Dereferenziazione**: `*ptr = 100;` (accede/modifica il valore puntato)

#### Esempio Pratico: Scambio di Valori

**Versione SBAGLIATA (senza puntatori):**
```c
#include <stdio.h>

void swap_sbagliato(int a, int b)
{
    int temp = a;
    a = b;
    b = temp;
    
    printf("Dentro funzione: a=%d, b=%d\n", a, b);
}

int main(void)
{
    int x = 5, y = 10;
    
    printf("Prima: x=%d, y=%d\n", x, y);
    swap_sbagliato(x, y);
    printf("Dopo: x=%d, y=%d\n", x, y);  // NON cambiati!
    
    return 0;
}
```

**Output:**
```
Prima: x=5, y=10
Dentro funzione: a=10, b=5
Dopo: x=5, y=10
```

**Perché non funziona?**
- `swap_sbagliato` riceve **copie** di x e y
- Modifica le copie, non gli originali
- Le copie vengono distrutte al return

**Diagramma dello stack:**
```
Stack durante swap_sbagliato:
┌─────────────┐
│ temp = 5    │  ← variabili locali di swap_sbagliato
│ a = 10      │  ← copia di x
│ b = 5       │  ← copia di y
├─────────────┤
│ x = 5       │  ← variabili di main (NON modificate!)
│ y = 10      │
└─────────────┘
```

**Versione CORRETTA (con puntatori):**
```c
#include <stdio.h>

void swap_corretto(int *a, int *b)
{
    int temp = *a;  // Leggi valore puntato da a
    *a = *b;        // Scrivi in a il valore puntato da b
    *b = temp;      // Scrivi in b il valore temporaneo
    
    printf("Dentro funzione: *a=%d, *b=%d\n", *a, *b);
}

int main(void)
{
    int x = 5, y = 10;
    
    printf("Prima: x=%d, y=%d\n", x, y);
    swap_corretto(&x, &y);  // Passa gli INDIRIZZI
    printf("Dopo: x=%d, y=%d\n", x, y);  // Cambiati!
    
    return 0;
}
```

**Output:**
```
Prima: x=5, y=10
Dentro funzione: *a=10, *b=5
Dopo: x=10, y=5
```

**Diagramma dello stack:**
```
Stack durante swap_corretto:
┌─────────────┐
│ temp = 5    │
│ a = 0x1000  │──┐
│ b = 0x1004  │──┼──┐
├─────────────┤  │  │
│ x = 5       │<─┘  │  ← modificato tramite *a
│ y = 10      │<────┘  ← modificato tramite *b
└─────────────┘
   0x1000  0x1004
```

**Passo per passo in swap_corretto:**
1. `int temp = *a;`
   - `*a` legge il valore dove punta `a` (x = 5)
   - `temp = 5`

2. `*a = *b;`
   - `*b` legge il valore dove punta `b` (y = 10)
   - `*a = 10` scrive 10 dove punta `a` (x diventa 10)

3. `*b = temp;`
   - `*b = 5` scrive 5 dove punta `b` (y diventa 5)

**Compilazione e test:**
```bash
gcc -Wall -Wextra puntatori_intro.c -o puntatori
./puntatori
```

### Aritmetica dei Puntatori

L'aritmetica dei puntatori permette di navigare attraverso la memoria in modo efficiente, specialmente con gli array.

```c
#include <stdio.h>

int main(void)
{
    int arr[] = {10, 20, 30, 40, 50};
    int *ptr = arr;  // arr è già un puntatore al primo elemento
    
    printf("Primo elemento: %d\n", *ptr);        // 10
    printf("Secondo elemento: %d\n", *(ptr+1));  // 20
    printf("Terzo elemento: %d\n", *(ptr+2));    // 30
    
    /* Iterazione con puntatori */
    printf("Array usando puntatori:\n");
    for (int *p = arr; p < arr + 5; p++) {
        printf("%d ", *p);
    }
    printf("\n");
    
    /* Dimensione degli incrementi */
    printf("ptr: %p\n", (void*)ptr);
    printf("ptr+1: %p\n", (void*)(ptr+1));
    printf("Differenza: %ld bytes\n", (char*)(ptr+1) - (char*)ptr);
    
    /* Sottrazione di puntatori */
    int *start = arr;
    int *end = arr + 4;
    printf("Distanza tra puntatori: %ld elementi\n", end - start);
    
    return 0;
}
```

#### Spiegazione Dettagliata dell'Aritmetica dei Puntatori

**Concetto fondamentale:**
Quando fai `ptr + 1`, NON aggiungi 1 byte, ma 1 elemento del tipo puntato!

**Array in memoria:**
```c
int arr[] = {10, 20, 30, 40, 50};
```

**Rappresentazione in memoria:**
```
Indirizzo    Valore    Espressione       Array notation
───────────────────────────────────────────────────────
0x1000       10        *arr o *(arr+0)   arr[0]
0x1004       20        *(arr+1)          arr[1]
0x1008       30        *(arr+2)          arr[2]
0x100C       40        *(arr+3)          arr[3]
0x1010       50        *(arr+4)          arr[4]
```

**Diagramma visivo:**
```
arr ────┐
        │
        ▼
┌────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │
└────┴────┴────┴────┴────┘
 arr  +1   +2   +3   +4
0x1000    0x1004    0x1008
```

**`int *ptr = arr;`**
- `arr` decade a puntatore al primo elemento
- Equivalente a: `int *ptr = &arr[0];`
- `ptr` ora contiene 0x1000 (indirizzo di arr[0])

**`*ptr` (dereferenzia ptr):**
- Legge il valore all'indirizzo contenuto in `ptr`
- `*ptr` → valore a 0x1000 → 10

**`*(ptr+1)` (aritmetica + dereferenziazione):**
- `ptr+1`: Calcola indirizzo dell'elemento successivo
- `ptr` è di tipo `int*`, quindi `ptr+1` = 0x1000 + sizeof(int) = 0x1000 + 4 = 0x1004
- `*(ptr+1)`: Legge il valore a 0x1004 → 20
- **Equivalente a**: `arr[1]`

**Regola generale:**
```c
ptr + n  →  (indirizzo_base + n * sizeof(tipo))
```

**Tabella di equivalenze:**
```c
Pointer Notation    Array Notation    Valore
────────────────────────────────────────────
*arr                arr[0]            10
*(arr+1)            arr[1]            20
*(arr+2)            arr[2]            30
*(arr+i)            arr[i]            arr[i]
```

**Iterazione con puntatori:**
```c
for (int *p = arr; p < arr + 5; p++) {
    printf("%d ", *p);
}
```

**Passo per passo:**
1. **Inizializzazione**: `p = arr` (p punta al primo elemento)
2. **Condizione**: `p < arr + 5` (p non ha superato l'ultimo elemento)
3. **Corpo**: `printf("%d ", *p)` (stampa valore puntato)
4. **Incremento**: `p++` (p avanza al prossimo elemento)

**Diagramma delle iterazioni:**
```
Iterazione 1: p = 0x1000  →  *p = 10  →  stampa 10
Iterazione 2: p = 0x1004  →  *p = 20  →  stampa 20
Iterazione 3: p = 0x1008  →  *p = 30  →  stampa 30
Iterazione 4: p = 0x100C  →  *p = 40  →  stampa 40
Iterazione 5: p = 0x1010  →  *p = 50  →  stampa 50
Iterazione 6: p = 0x1014  →  p >= arr+5  →  STOP
```

**Incremento automatico della dimensione:**
```c
printf("ptr: %p\n", (void*)ptr);        // 0x1000
printf("ptr+1: %p\n", (void*)(ptr+1));  // 0x1004
```

**Spiegazione:**
- `ptr` è di tipo `int*` (puntatore a int)
- `int` è 4 byte su sistemi comuni
- `ptr+1` avanza di 4 byte (1 elemento)
- Se fosse `char*`, avanzerebbe di 1 byte
- Se fosse `double*`, avanzerebbe di 8 byte

**Calcolo della differenza:**
```c
printf("Differenza: %ld bytes\n", (char*)(ptr+1) - (char*)ptr);
```

**Spiegazione:**
- Cast a `char*` per calcolare la differenza in byte
- Senza cast: `(ptr+1) - ptr` darebbe 1 (1 elemento)
- Con cast: `(char*)(ptr+1) - (char*)ptr` dà 4 (4 byte)

**Sottrazione di puntatori:**
```c
int *start = arr;      // 0x1000
int *end = arr + 4;    // 0x1010
printf("Distanza: %ld elementi\n", end - start);  // 4
```

**Spiegazione:**
- Sottrazione di puntatori restituisce il numero di elementi tra loro
- `end - start` = (0x1010 - 0x1000) / sizeof(int) = 16 / 4 = 4 elementi
- **NON** restituisce byte, ma numero di elementi!

#### Esempio Pratico: Ricerca in Array con Puntatori

```c
#include <stdio.h>
#include <stdbool.h>

/* Cerca un valore nell'array usando puntatori
 * Ritorna puntatore all'elemento se trovato, NULL altrimenti */
int* ricerca_valore(int *arr, int size, int target)
{
    /* Calcola puntatore alla fine dell'array */
    int *end = arr + size;
    
    /* Itera usando aritmetica dei puntatori */
    for (int *p = arr; p < end; p++) {
        if (*p == target) {
            return p;  /* Ritorna puntatore all'elemento trovato */
        }
    }
    
    return NULL;  /* Non trovato */
}

int main(void)
{
    int numeri[] = {15, 42, 7, 23, 56, 91, 12};
    int size = sizeof(numeri) / sizeof(numeri[0]);
    int target = 23;
    
    /* Cerca il target */
    int *risultato = ricerca_valore(numeri, size, target);
    
    if (risultato != NULL) {
        /* Calcola l'indice usando sottrazione di puntatori */
        int indice = risultato - numeri;
        
        printf("Valore %d trovato!\n", target);
        printf("Indirizzo: %p\n", (void*)risultato);
        printf("Indice: %d\n", indice);
        printf("Verifica: numeri[%d] = %d\n", indice, numeri[indice]);
    } else {
        printf("Valore %d non trovato\n", target);
    }
    
    return 0;
}
```

**Output:**
```
Valore 23 trovato!
Indirizzo: 0x7fff1234560c
Indice: 3
Verifica: numeri[3] = 23
```

**Analisi passo-passo:**

1. **`int *end = arr + size;`**
   - Calcola puntatore "oltre" l'ultimo elemento
   - `end` punta a `arr[7]` (fuori dai limiti, ma non dereferenziato)

2. **Loop `for (int *p = arr; p < end; p++)`:**
   - `p` parte da `arr[0]`
   - Continua finché `p` è prima di `end`
   - `p++` avanza al prossimo elemento

3. **`if (*p == target)`:**
   - Dereferenzia `p` e confronta con `target`
   - Se uguale, ritorna `p` (puntatore all'elemento)

4. **`int indice = risultato - numeri;`:**
   - Sottrazione di puntatori
   - Calcola la distanza in elementi
   - `risultato` punta a `numeri[3]`
   - `risultato - numeri` = 3

#### Operazioni Valide e Non Valide

**✅ Operazioni VALIDE:**
```c
int arr[5];
int *p = arr;

p + 3;        // Avanza di 3 elementi
p - 2;        // Indietro di 2 elementi
p++;          // Avanza di 1 elemento
p--;          // Indietro di 1 elemento
p += 2;       // Avanza di 2 elementi
p -= 1;       // Indietro di 1 elemento

int *q = arr + 2;
q - p;        // Differenza in elementi
p < q;        // Confronto di puntatori
p == q;       // Uguaglianza
```

**❌ Operazioni NON VALIDE:**
```c
p + q;        // ERRORE: non puoi sommare due puntatori
p * 2;        // ERRORE: non puoi moltiplicare puntatori
p / 2;        // ERRORE: non puoi dividere puntatori
```

#### Confronto: Notazione Array vs Puntatori

**Equivalenza completa:**
```c
int arr[5] = {1, 2, 3, 4, 5};

/* Queste sono tutte equivalenti: */
arr[i]          // Notazione array
*(arr + i)      // Notazione puntatore
*(i + arr)      // Anche questo funziona! (commutativo)
i[arr]          // Sorprendente ma valido! (non usare)
```

**Esempio di preferenza:**
```c
/* Quando usare array notation */
int x = arr[i];              // Chiaro e leggibile

/* Quando usare pointer notation */
for (int *p = arr; p < arr + size; p++) {
    process(*p);             // Efficiente per iterazione
}
```

**Compilazione e test:**
```bash
gcc -Wall -Wextra aritmetica_puntatori.c -o aritmetica
./aritmetica
```

### Puntatori e Array

```c
#include <stdio.h>

void stampa_array_ptr(int *arr, int size)
{
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);  // Equivalente a *(arr + i)
    }
    printf("\n");
}

void stampa_array_notation(int arr[], int size)
{
    for (int i = 0; i < size; i++) {
        printf("%d ", *(arr + i));  // Equivalente a arr[i]
    }
    printf("\n");
}

int main(void)
{
    int numeri[] = {1, 2, 3, 4, 5};
    int size = sizeof(numeri) / sizeof(numeri[0]);
    
    /* Array e puntatori sono intercambiabili come parametri */
    stampa_array_ptr(numeri, size);
    stampa_array_notation(numeri, size);
    
    /* ATTENZIONE: differenza tra array e puntatori */
    printf("sizeof(numeri): %lu\n", sizeof(numeri));      // 20 (5 * 4 bytes)
    
    int *ptr = numeri;
    printf("sizeof(ptr): %lu\n", sizeof(ptr));            // 8 (dimensione puntatore)
    
    return 0;
}
```

### Puntatori a Puntatori

```c
#include <stdio.h>

int main(void)
{
    int x = 42;
    int *ptr = &x;
    int **ptr_ptr = &ptr;  // Puntatore a puntatore
    
    printf("Valore di x: %d\n", x);
    printf("Tramite ptr: %d\n", *ptr);
    printf("Tramite ptr_ptr: %d\n", **ptr_ptr);
    
    /* Modificare tramite puntatore a puntatore */
    **ptr_ptr = 100;
    printf("Nuovo valore di x: %d\n", x);
    
    /* Uso pratico: array di stringhe */
    char *nomi[] = {"Alice", "Bob", "Charlie"};
    char **ptr_nomi = nomi;
    
    for (int i = 0; i < 3; i++) {
        printf("Nome %d: %s\n", i, ptr_nomi[i]);
    }
    
    return 0;
}
```

### Puntatori a Funzioni

```c
#include <stdio.h>

int somma(int a, int b) { return a + b; }
int sottrazione(int a, int b) { return a - b; }
int moltiplicazione(int a, int b) { return a * b; }

/* Funzione che accetta puntatore a funzione */
int esegui_operazione(int a, int b, int (*operazione)(int, int))
{
    return operazione(a, b);
}

/* Array di puntatori a funzione */
void usa_calcolatrice(void)
{
    int (*operazioni[])(int, int) = {somma, sottrazione, moltiplicazione};
    char *nomi[] = {"Somma", "Sottrazione", "Moltiplicazione"};
    
    int a = 10, b = 5;
    for (int i = 0; i < 3; i++) {
        int risultato = operazioni[i](a, b);
        printf("%s: %d\n", nomi[i], risultato);
    }
}

int main(void)
{
    /* Dichiarazione puntatore a funzione */
    int (*func_ptr)(int, int);
    
    func_ptr = somma;
    printf("10 + 5 = %d\n", func_ptr(10, 5));
    
    func_ptr = sottrazione;
    printf("10 - 5 = %d\n", func_ptr(10, 5));
    
    /* Passare funzione come parametro */
    printf("Esegui somma: %d\n", esegui_operazione(10, 5, somma));
    printf("Esegui moltiplicazione: %d\n", esegui_operazione(10, 5, moltiplicazione));
    
    printf("\nCalcolatrice:\n");
    usa_calcolatrice();
    
    return 0;
}
```

## Allocazione Dinamica della Memoria

L'allocazione dinamica permette di creare variabili e array la cui dimensione può essere decisa a runtime, non a compile-time.

### Stack vs Heap

**Stack (allocazione automatica):**
```c
int x = 10;              // Stack
int arr[100];            // Stack - dimensione fissa
```
- **Dimensione**: Fissa a compile-time
- **Durata**: Variabile distrutta quando esci dallo scope
- **Gestione**: Automatica
- **Velocità**: Molto veloce
- **Dimensione limitata**: Tipicamente 1-8 MB

**Heap (allocazione dinamica):**
```c
int *ptr = malloc(sizeof(int));        // Heap
int *arr = malloc(100 * sizeof(int));  // Heap - dimensione variabile
```
- **Dimensione**: Decisa a runtime
- **Durata**: Vive finché non chiami `free()`
- **Gestione**: Manuale (devi liberare la memoria!)
- **Velocità**: Più lenta dello stack
- **Dimensione**: Limitata solo dalla RAM disponibile

**Diagramma Stack vs Heap:**
```
Memoria del Processo:
┌─────────────────┐
│   Code          │  ← Codice del programma
├─────────────────┤
│   Static Data   │  ← Variabili globali/static
├─────────────────┤
│   Heap →        │  ← Cresce verso il basso
│   (malloc)      │     Allocazione dinamica
│                 │
│   ↓ (libero) ↑  │
│                 │
│   (Stack)       │  ← Cresce verso l'alto
│   ← Stack       │     Variabili locali
└─────────────────┘
```

### malloc, calloc, realloc, free

**Funzioni principali:**
- `malloc()`: Alloca memoria NON inizializzata
- `calloc()`: Alloca memoria inizializzata a ZERO
- `realloc()`: Ridimensiona memoria già allocata
- `free()`: Libera memoria allocata

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    /* malloc - alloca memoria NON inizializzata */
    int *arr1 = (int*)malloc(5 * sizeof(int));
    if (arr1 == NULL) {
        fprintf(stderr, "Errore: allocazione fallita\n");
        return 1;
    }
    
    for (int i = 0; i < 5; i++) {
        arr1[i] = i * 10;
    }
    
    /* calloc - alloca memoria INIZIALIZZATA a zero */
    int *arr2 = (int*)calloc(5, sizeof(int));
    if (arr2 == NULL) {
        free(arr1);
        fprintf(stderr, "Errore: allocazione fallita\n");
        return 1;
    }
    
    printf("arr2 (calloc):\n");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr2[i]);  // Tutti 0
    }
    printf("\n");
    
    /* realloc - ridimensiona memoria allocata */
    arr1 = (int*)realloc(arr1, 10 * sizeof(int));
    if (arr1 == NULL) {
        free(arr2);
        fprintf(stderr, "Errore: riallocazione fallita\n");
        return 1;
    }
    
    /* Inizializza nuovi elementi */
    for (int i = 5; i < 10; i++) {
        arr1[i] = i * 10;
    }
    
    printf("arr1 dopo realloc:\n");
    for (int i = 0; i < 10; i++) {
        printf("%d ", arr1[i]);
    }
    printf("\n");
    
    /* IMPORTANTE: liberare sempre la memoria! */
    free(arr1);
    free(arr2);
    
    return 0;
}
```

#### Spiegazione Dettagliata delle Funzioni

**`malloc()` - Memory Allocation**

**Sintassi:**
```c
void* malloc(size_t size);
```

**Parametri:**
- `size`: Numero di byte da allocare

**Ritorna:**
- Puntatore alla memoria allocata (tipo `void*`)
- `NULL` se l'allocazione fallisce

**Esempio annotato:**
```c
int *arr = (int*)malloc(5 * sizeof(int));
```

**Analisi passo-passo:**
1. `sizeof(int)`: Calcola dimensione di un int (tipicamente 4 byte)
2. `5 * sizeof(int)`: Calcola byte totali necessari (5 × 4 = 20 byte)
3. `malloc(20)`: Alloca 20 byte contigui nell'heap
4. Ritorna `void*` (puntatore generico)
5. `(int*)`: Cast a puntatore a int (obbligatorio in C++)
6. `arr`: Riceve l'indirizzo del primo byte allocato

**Memoria allocata:**
```
Heap:
┌────┬────┬────┬────┬────┐
│ ?? │ ?? │ ?? │ ?? │ ?? │  ← 5 interi NON inizializzati
└────┴────┴────┴────┴────┘
 arr  [0]  [1]  [2]  [3]  [4]
```

**⚠️ IMPORTANTE: Controllare sempre NULL!**
```c
int *arr = (int*)malloc(5 * sizeof(int));
if (arr == NULL) {
    fprintf(stderr, "Errore: memoria insufficiente\n");
    return 1;
}
```

**Quando malloc può fallire:**
- Memoria insufficiente
- Frammentazione della memoria
- Sistema sotto stress
- Limite del processo raggiunto

**`calloc()` - Contiguous Allocation**

**Sintassi:**
```c
void* calloc(size_t num, size_t size);
```

**Parametri:**
- `num`: Numero di elementi
- `size`: Dimensione di ogni elemento

**Ritorna:**
- Puntatore alla memoria allocata E INIZIALIZZATA A ZERO
- `NULL` se fallisce

**Esempio annotato:**
```c
int *arr = (int*)calloc(5, sizeof(int));
```

**Differenza con malloc:**
- `malloc(5 * sizeof(int))`: Memoria NON inizializzata (valori casuali)
- `calloc(5, sizeof(int))`: Memoria inizializzata a ZERO

**Memoria allocata:**
```
Heap:
┌────┬────┬────┬────┬────┐
│  0 │  0 │  0 │  0 │  0 │  ← Tutti inizializzati a 0!
└────┴────┴────┴────┴────┘
```

**Quando usare calloc:**
- Array di struct che devono essere inizializzate
- Array di puntatori (inizializzati a NULL)
- Quando zero è un valore valido di default
- Per sicurezza (evita valori garbage)

**Prestazioni:**
- `calloc` è leggermente più lento di `malloc` (deve azzerare)
- Differenza trascurabile per array piccoli
- Può essere più efficiente per array molto grandi (ottimizzazioni OS)

**`realloc()` - Reallocate Memory**

**Sintassi:**
```c
void* realloc(void *ptr, size_t new_size);
```

**Parametri:**
- `ptr`: Puntatore alla memoria da ridimensionare (o NULL)
- `new_size`: Nuova dimensione in byte

**Ritorna:**
- Puntatore alla memoria ridimensionata
- Può essere DIVERSO dal puntatore originale!
- `NULL` se fallisce (ma memoria originale rimane valida)

**Esempio annotato:**
```c
int *arr = (int*)malloc(5 * sizeof(int));
// ... uso arr ...
arr = (int*)realloc(arr, 10 * sizeof(int));
```

**Cosa succede internamente:**

**Caso 1: Espansione con spazio disponibile**
```
Prima:
┌────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │ [spazio libero...]
└────┴────┴────┴────┴────┘
 arr

Dopo realloc(arr, 10*sizeof(int)):
┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │ ?? │ ?? │ ?? │ ?? │ ?? │
└────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
 arr (stesso indirizzo)
```

**Caso 2: Espansione senza spazio (deve spostare)**
```
Prima:
┌────┬────┬────┬────┬────┐ [occupato...]
│ 10 │ 20 │ 30 │ 40 │ 50 │
└────┴────┴────┴────┴────┘
 arr @ 0x1000

Dopo realloc:
1. Alloca nuovo blocco da 10 elementi @ 0x2000
2. Copia i primi 5 elementi dal vecchio al nuovo
3. Libera il vecchio blocco @ 0x1000
4. Ritorna nuovo indirizzo @ 0x2000

┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │ ?? │ ?? │ ?? │ ?? │ ?? │
└────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
 arr @ 0x2000 (NUOVO indirizzo!)
```

**⚠️ PERICOLO: Gestione errori di realloc**

**SBAGLIATO:**
```c
arr = (int*)realloc(arr, new_size);
if (arr == NULL) {
    // PROBLEMA: hai perso il puntatore originale!
}
```

**CORRETTO:**
```c
int *temp = (int*)realloc(arr, new_size);
if (temp == NULL) {
    // arr è ancora valido, puoi usarlo o liberarlo
    fprintf(stderr, "Errore: realloc fallita\n");
    free(arr);  // Libera la memoria originale
    return 1;
} else {
    arr = temp;  // Aggiorna arr solo se successo
}
```

**Casi speciali di realloc:**
```c
// Se ptr è NULL, equivale a malloc
int *arr = (int*)realloc(NULL, 10 * sizeof(int));
// Equivalente a: malloc(10 * sizeof(int))

// Se new_size è 0, equivale a free (non portabile!)
realloc(arr, 0);
// Meglio usare: free(arr); arr = NULL;
```

**`free()` - Free Memory**

**Sintassi:**
```c
void free(void *ptr);
```

**Parametri:**
- `ptr`: Puntatore alla memoria da liberare

**Esempio annotato:**
```c
int *arr = (int*)malloc(5 * sizeof(int));
// ... uso arr ...
free(arr);        // Libera la memoria
arr = NULL;       // Buona pratica: previene dangling pointer
```

**Cosa fa free:**
1. Marca il blocco di memoria come disponibile
2. **NON** azzera la memoria
3. **NON** cambia il valore del puntatore

**Diagramma:**
```
Prima di free(arr):
Heap:
┌────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │  ← Memoria allocata
└────┴────┴────┴────┴────┘
 ↑
 arr punta qui

Dopo free(arr):
Heap:
┌────┬────┬────┬────┬────┐
│ ?? │ ?? │ ?? │ ?? │ ?? │  ← Memoria libera (ma dati ancora presenti!)
└────┴────┴────┴────┴────┘
 ↑
 arr ANCORA punta qui (DANGLING POINTER!)

Dopo arr = NULL:
Heap:
┌────┬────┬────┬────┬────┐
│ ?? │ ?? │ ?? │ ?? │ ?? │  ← Memoria libera
└────┴────┴────┴────┴────┘

arr = NULL  ← Sicuro: non punta più a memoria invalida
```

**Regole di free:**
1. **Mai** fare free di memoria non allocata con malloc/calloc/realloc
2. **Mai** fare free della stessa memoria due volte
3. **Mai** usare memoria dopo averla liberata
4. **Sempre** settare il puntatore a NULL dopo free

#### Esempio Completo: Array Dinamico che Cresce

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int capacita = 5;
    int dimensione = 0;
    int *arr = (int*)malloc(capacita * sizeof(int));
    
    if (arr == NULL) {
        fprintf(stderr, "Errore: allocazione fallita\n");
        return 1;
    }
    
    printf("Capacità iniziale: %d\n", capacita);
    
    /* Aggiungi 10 numeri all'array */
    for (int i = 0; i < 10; i++) {
        /* Se array pieno, raddoppia la capacità */
        if (dimensione >= capacita) {
            capacita *= 2;
            printf("Espando array a capacità %d\n", capacita);
            
            int *temp = (int*)realloc(arr, capacita * sizeof(int));
            if (temp == NULL) {
                fprintf(stderr, "Errore: realloc fallita\n");
                free(arr);
                return 1;
            }
            arr = temp;
        }
        
        /* Aggiungi elemento */
        arr[dimensione] = i * 10;
        dimensione++;
        printf("Aggiunto %d (dimensione: %d/%d)\n", 
               arr[dimensione-1], dimensione, capacita);
    }
    
    /* Stampa array finale */
    printf("\nArray finale:\n");
    for (int i = 0; i < dimensione; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    
    /* Libera memoria */
    free(arr);
    arr = NULL;
    
    return 0;
}
```

**Output:**
```
Capacità iniziale: 5
Aggiunto 0 (dimensione: 1/5)
Aggiunto 10 (dimensione: 2/5)
Aggiunto 20 (dimensione: 3/5)
Aggiunto 30 (dimensione: 4/5)
Aggiunto 40 (dimensione: 5/5)
Espando array a capacità 10
Aggiunto 50 (dimensione: 6/10)
Aggiunto 60 (dimensione: 7/10)
Aggiunto 70 (dimensione: 8/10)
Aggiunto 80 (dimensione: 9/10)
Aggiunto 90 (dimensione: 10/10)

Array finale:
0 10 20 30 40 50 60 70 80 90
```

**Compilazione e test:**
```bash
gcc -Wall -Wextra allocazione_dinamica.c -o allocazione
./allocazione
```

### Memory Leaks e Dangling Pointers

```c
#include <stdio.h>
#include <stdlib.h>

/* ERRORE: Memory leak */
void memory_leak_bad(void)
{
    int *ptr = (int*)malloc(100 * sizeof(int));
    // Uso di ptr...
    // MANCA free(ptr)!  <-- MEMORY LEAK
}

/* CORRETTO */
void memory_leak_good(void)
{
    int *ptr = (int*)malloc(100 * sizeof(int));
    if (ptr == NULL) return;
    // Uso di ptr...
    free(ptr);  // Memoria liberata correttamente
}

/* ERRORE: Dangling pointer */
void dangling_pointer_bad(void)
{
    int *ptr = (int*)malloc(sizeof(int));
    *ptr = 42;
    free(ptr);
    // *ptr = 100;  // ERRORE: uso dopo free! <-- DANGLING POINTER
}

/* CORRETTO */
void dangling_pointer_good(void)
{
    int *ptr = (int*)malloc(sizeof(int));
    *ptr = 42;
    free(ptr);
    ptr = NULL;  // Previene uso accidentale
    
    if (ptr != NULL) {
        *ptr = 100;  // Questo non verrà eseguito
    }
}

/* ERRORE: Double free */
void double_free_bad(void)
{
    int *ptr = (int*)malloc(sizeof(int));
    free(ptr);
    // free(ptr);  // ERRORE: doppia free! <-- UNDEFINED BEHAVIOR
}

int main(void)
{
    memory_leak_good();
    dangling_pointer_good();
    
    printf("Esempi eseguiti correttamente\n");
    return 0;
}
```

### Allocazione di Strutture Dinamiche

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char *nome;
    int eta;
    double altezza;
} Persona;

Persona* crea_persona(const char *nome, int eta, double altezza)
{
    Persona *p = (Persona*)malloc(sizeof(Persona));
    if (p == NULL) return NULL;
    
    /* Alloca memoria per la stringa */
    p->nome = (char*)malloc(strlen(nome) + 1);
    if (p->nome == NULL) {
        free(p);
        return NULL;
    }
    
    strcpy(p->nome, nome);
    p->eta = eta;
    p->altezza = altezza;
    
    return p;
}

void distruggi_persona(Persona *p)
{
    if (p != NULL) {
        free(p->nome);  // Prima libera la stringa
        free(p);        // Poi libera la struttura
    }
}

void stampa_persona(const Persona *p)
{
    if (p != NULL) {
        printf("Nome: %s, Età: %d, Altezza: %.2f\n", 
               p->nome, p->eta, p->altezza);
    }
}

int main(void)
{
    Persona *p1 = crea_persona("Mario Rossi", 30, 1.75);
    Persona *p2 = crea_persona("Luigi Verdi", 25, 1.80);
    
    if (p1 && p2) {
        stampa_persona(p1);
        stampa_persona(p2);
        
        distruggi_persona(p1);
        distruggi_persona(p2);
    }
    
    return 0;
}
```

## Strutture (struct)

### Definizione e Uso

```c
#include <stdio.h>
#include <string.h>

/* Definizione di struttura */
struct Punto {
    int x;
    int y;
};

/* Con typedef per evitare "struct" ogni volta */
typedef struct {
    char nome[50];
    int eta;
    float voto;
} Studente;

int main(void)
{
    /* Dichiarazione e inizializzazione */
    struct Punto p1 = {10, 20};
    struct Punto p2 = {.y = 30, .x = 40};  // Designated initializers (C99)
    
    printf("Punto 1: (%d, %d)\n", p1.x, p1.y);
    printf("Punto 2: (%d, %d)\n", p2.x, p2.y);
    
    /* Con typedef */
    Studente s1 = {"Mario", 20, 27.5};
    Studente s2;
    strcpy(s2.nome, "Luigi");
    s2.eta = 22;
    s2.voto = 28.0;
    
    printf("Studente 1: %s, %d anni, voto %.1f\n", 
           s1.nome, s1.eta, s1.voto);
    printf("Studente 2: %s, %d anni, voto %.1f\n", 
           s2.nome, s2.eta, s2.voto);
    
    return 0;
}
```

### Strutture Annidate e Array di Strutture

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    int giorno;
    int mese;
    int anno;
} Data;

typedef struct {
    char nome[50];
    Data nascita;
    float stipendio;
} Impiegato;

void stampa_impiegato(const Impiegato *imp)
{
    printf("Nome: %s\n", imp->nome);
    printf("Nascita: %02d/%02d/%04d\n", 
           imp->nascita.giorno, imp->nascita.mese, imp->nascita.anno);
    printf("Stipendio: €%.2f\n\n", imp->stipendio);
}

int main(void)
{
    /* Array di strutture */
    Impiegato impiegati[3] = {
        {"Mario Rossi", {15, 3, 1990}, 2500.00},
        {"Luigi Verdi", {22, 7, 1985}, 3000.00},
        {"Anna Bianchi", {8, 12, 1992}, 2800.00}
    };
    
    printf("Elenco impiegati:\n");
    for (int i = 0; i < 3; i++) {
        printf("--- Impiegato %d ---\n", i + 1);
        stampa_impiegato(&impiegati[i]);
    }
    
    return 0;
}
```

### Puntatori a Strutture

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char titolo[100];
    char autore[50];
    int anno;
    float prezzo;
} Libro;

void stampa_libro_ptr(const Libro *libro)
{
    /* Sintassi con -> per accedere a membri tramite puntatore */
    printf("Titolo: %s\n", libro->titolo);
    printf("Autore: %s\n", libro->autore);
    printf("Anno: %d\n", libro->anno);
    printf("Prezzo: €%.2f\n\n", libro->prezzo);
}

void applica_sconto(Libro *libro, float percentuale)
{
    libro->prezzo *= (1.0 - percentuale / 100.0);
}

int main(void)
{
    Libro *libro = (Libro*)malloc(sizeof(Libro));
    if (libro == NULL) return 1;
    
    strcpy(libro->titolo, "Il linguaggio C");
    strcpy(libro->autore, "Kernighan & Ritchie");
    libro->anno = 1978;
    libro->prezzo = 45.00;
    
    printf("Prima dello sconto:\n");
    stampa_libro_ptr(libro);
    
    applica_sconto(libro, 20);  // 20% di sconto
    
    printf("Dopo lo sconto del 20%%:\n");
    stampa_libro_ptr(libro);
    
    free(libro);
    return 0;
}
```

## Union

### Definizione e Uso

Le union permettono di memorizzare diversi tipi di dati nello stesso spazio di memoria.

```c
#include <stdio.h>

typedef union {
    int i;
    float f;
    char c;
} Dato;

typedef struct {
    enum { TIPO_INT, TIPO_FLOAT, TIPO_CHAR } tipo;
    union {
        int i;
        float f;
        char c;
    } valore;
} DatoTagged;

int main(void)
{
    Dato d;
    
    d.i = 42;
    printf("Come int: %d\n", d.i);
    
    d.f = 3.14f;
    printf("Come float: %.2f\n", d.f);
    printf("Come int (dopo aver settato float): %d\n", d.i);  // GARBAGE!
    
    /* Union tagged - approccio sicuro */
    DatoTagged dt1 = {TIPO_INT, {.i = 100}};
    DatoTagged dt2 = {TIPO_FLOAT, {.f = 2.718f}};
    DatoTagged dt3 = {TIPO_CHAR, {.c = 'A'}};
    
    /* Funzione per stampare in modo sicuro */
    DatoTagged dati[] = {dt1, dt2, dt3};
    for (int i = 0; i < 3; i++) {
        switch (dati[i].tipo) {
            case TIPO_INT:
                printf("Int: %d\n", dati[i].valore.i);
                break;
            case TIPO_FLOAT:
                printf("Float: %.3f\n", dati[i].valore.f);
                break;
            case TIPO_CHAR:
                printf("Char: %c\n", dati[i].valore.c);
                break;
        }
    }
    
    printf("\nDimensioni:\n");
    printf("sizeof(Dato): %lu\n", sizeof(Dato));
    printf("sizeof(int): %lu\n", sizeof(int));
    printf("sizeof(float): %lu\n", sizeof(float));
    printf("sizeof(char): %lu\n", sizeof(char));
    
    return 0;
}
```

## Typedef ed Enum

### Typedef

```c
#include <stdio.h>

/* Typedef per tipi base */
typedef unsigned long ulong;
typedef unsigned char byte;

/* Typedef per puntatori */
typedef int* IntPtr;

/* Typedef per array */
typedef int Matrix[3][3];

/* Typedef per funzioni */
typedef int (*ComparaFn)(int, int);

int confronta_crescente(int a, int b) { return a - b; }
int confronta_decrescente(int a, int b) { return b - a; }

int main(void)
{
    ulong big = 123456789UL;
    byte b = 255;
    
    IntPtr ptr = &b;  // int*
    
    Matrix m = {{1,2,3}, {4,5,6}, {7,8,9}};
    printf("m[1][1] = %d\n", m[1][1]);
    
    ComparaFn cmp = confronta_crescente;
    printf("Confronto 5 e 3: %d\n", cmp(5, 3));
    
    return 0;
}
```

### Enum

```c
#include <stdio.h>

/* Enum semplice */
enum Giorno {
    LUNEDI,      // 0
    MARTEDI,     // 1
    MERCOLEDI,   // 2
    GIOVEDI,     // 3
    VENERDI,     // 4
    SABATO,      // 5
    DOMENICA     // 6
};

/* Enum con valori specifici */
enum Mese {
    GENNAIO = 1,
    FEBBRAIO,
    MARZO,
    APRILE,
    MAGGIO,
    GIUGNO,
    LUGLIO,
    AGOSTO,
    SETTEMBRE,
    OTTOBRE,
    NOVEMBRE,
    DICEMBRE
};

/* Enum per flag (bit mask) */
enum Permessi {
    LETTURA = 1 << 0,    // 0001 = 1
    SCRITTURA = 1 << 1,  // 0010 = 2
    ESECUZIONE = 1 << 2  // 0100 = 4
};

typedef enum {
    ROSSO,
    VERDE,
    BLU
} Colore;

const char* nome_giorno(enum Giorno g)
{
    switch (g) {
        case LUNEDI: return "Lunedì";
        case MARTEDI: return "Martedì";
        case MERCOLEDI: return "Mercoledì";
        case GIOVEDI: return "Giovedì";
        case VENERDI: return "Venerdì";
        case SABATO: return "Sabato";
        case DOMENICA: return "Domenica";
        default: return "Sconosciuto";
    }
}

int main(void)
{
    enum Giorno oggi = MERCOLEDI;
    printf("Oggi è %s\n", nome_giorno(oggi));
    
    enum Mese corrente = NOVEMBRE;
    printf("Mese corrente: %d\n", corrente);
    
    /* Uso di flag */
    int permessi = LETTURA | SCRITTURA;  // 0011 = 3
    
    if (permessi & LETTURA) {
        printf("Ha permesso di lettura\n");
    }
    if (permessi & SCRITTURA) {
        printf("Ha permesso di scrittura\n");
    }
    if (!(permessi & ESECUZIONE)) {
        printf("NON ha permesso di esecuzione\n");
    }
    
    return 0;
}
```

## Preprocessore

### Direttive Base

```c
#include <stdio.h>

/* Define semplici */
#define PI 3.14159
#define MAX_SIZE 100
#define SQUARE(x) ((x) * (x))  // Macro con parametri

/* Macro multi-linea */
#define SWAP(a, b, type) do { \
    type temp = a; \
    a = b; \
    b = temp; \
} while(0)

/* Stringification */
#define TO_STRING(x) #x
#define PRINT_VAR(var) printf(#var " = %d\n", var)

/* Token pasting */
#define CONCAT(a, b) a##b

/* Compilazione condizionale */
#define DEBUG 1

#if DEBUG
    #define LOG(msg) printf("LOG: %s\n", msg)
#else
    #define LOG(msg)
#endif

int main(void)
{
    printf("PI = %f\n", PI);
    printf("Area cerchio (r=5): %f\n", PI * SQUARE(5));
    
    int x = 10, y = 20;
    printf("Prima: x=%d, y=%d\n", x, y);
    SWAP(x, y, int);
    printf("Dopo: x=%d, y=%d\n", x, y);
    
    printf("Stringification: %s\n", TO_STRING(Hello World));
    
    int value = 42;
    PRINT_VAR(value);
    
    int CONCAT(var, 123) = 999;
    printf("var123 = %d\n", var123);
    
    LOG("Programma avviato");
    
    /* Macro predefinite */
    printf("File: %s\n", __FILE__);
    printf("Linea: %d\n", __LINE__);
    printf("Data compilazione: %s\n", __DATE__);
    printf("Ora compilazione: %s\n", __TIME__);
    
    return 0;
}
```

### Include Guards

```c
/* file: myheader.h */

#ifndef MYHEADER_H
#define MYHEADER_H

/* Dichiarazioni... */
void my_function(void);
int my_variable;

#endif /* MYHEADER_H */

/* Oppure con pragma (non standard ma molto supportato) */
#pragma once

/* Dichiarazioni... */
```

## File I/O

### Operazioni Base

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    FILE *file;
    
    /* Scrittura */
    file = fopen("output.txt", "w");
    if (file == NULL) {
        perror("Errore apertura file");
        return 1;
    }
    
    fprintf(file, "Prima riga\n");
    fprintf(file, "Numero: %d\n", 42);
    fprintf(file, "Float: %.2f\n", 3.14);
    
    fclose(file);
    
    /* Lettura */
    file = fopen("output.txt", "r");
    if (file == NULL) {
        perror("Errore apertura file");
        return 1;
    }
    
    char buffer[256];
    printf("Contenuto del file:\n");
    while (fgets(buffer, sizeof(buffer), file) != NULL) {
        printf("%s", buffer);
    }
    
    fclose(file);
    
    /* Append */
    file = fopen("output.txt", "a");
    if (file != NULL) {
        fprintf(file, "Riga aggiunta\n");
        fclose(file);
    }
    
    return 0;
}
```

### Lettura e Scrittura Binaria

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int id;
    char nome[50];
    float stipendio;
} Record;

int main(void)
{
    /* Scrittura binaria */
    FILE *file = fopen("data.bin", "wb");
    if (file == NULL) return 1;
    
    Record records[] = {
        {1, "Mario", 2500.00},
        {2, "Luigi", 3000.00},
        {3, "Anna", 2800.00}
    };
    
    fwrite(records, sizeof(Record), 3, file);
    fclose(file);
    
    /* Lettura binaria */
    file = fopen("data.bin", "rb");
    if (file == NULL) return 1;
    
    Record read_records[3];
    fread(read_records, sizeof(Record), 3, file);
    fclose(file);
    
    printf("Record letti:\n");
    for (int i = 0; i < 3; i++) {
        printf("ID: %d, Nome: %s, Stipendio: %.2f\n",
               read_records[i].id,
               read_records[i].nome,
               read_records[i].stipendio);
    }
    
    return 0;
}
```

## Esercizi

1. Implementa una linked list con allocazione dinamica
2. Crea un programma che legge un file CSV e lo analizza
3. Scrivi funzioni per manipolare una matrice dinamica
4. Implementa un sistema di gestione studenti con struct
5. Crea un parser per un semplice formato di configurazione

---

[← Fondamenti](../02_Fondamenti/README.md) | [Torna al Sommario](../README.md) | [Prossimo: Avanzato →](../04_Avanzato/README.md)
