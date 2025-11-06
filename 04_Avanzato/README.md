# Modulo 04 - Livello Avanzato

## Puntatori Avanzati

### Array di Puntatori vs Puntatori ad Array

Questa è una delle distinzioni più confuse in C. Vediamo la differenza con diagrammi dettagliati.

```c
#include <stdio.h>

int main(void)
{
    /* Array di puntatori */
    int a = 1, b = 2, c = 3;
    int *array_of_ptrs[3] = {&a, &b, &c};
    
    printf("Array di puntatori:\n");
    for (int i = 0; i < 3; i++) {
        printf("*array_of_ptrs[%d] = %d\n", i, *array_of_ptrs[i]);
    }
    
    /* Puntatore ad array */
    int arr[3] = {10, 20, 30};
    int (*ptr_to_array)[3] = &arr;  // Puntatore all'intero array
    
    printf("\nPuntatore ad array:\n");
    for (int i = 0; i < 3; i++) {
        printf("(*ptr_to_array)[%d] = %d\n", i, (*ptr_to_array)[i]);
    }
    
    /* Array multidimensionale e puntatori */
    int matrix[2][3] = {{1, 2, 3}, {4, 5, 6}};
    int (*ptr_to_matrix)[3] = matrix;  // Puntatore alla prima riga
    
    printf("\nMatrix con puntatore:\n");
    for (int i = 0; i < 2; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d ", ptr_to_matrix[i][j]);
        }
        printf("\n");
    }
    
    return 0;
}
```

#### Spiegazione Dettagliata

**1. Array di Puntatori: `int *array_of_ptrs[3]`**

**Dichiarazione analizzata:**
```c
int *array_of_ptrs[3];
```

**Come leggere la dichiarazione (da destra a sinistra):**
1. `array_of_ptrs[3]`: È un array di 3 elementi
2. `*`: Ogni elemento è un puntatore
3. `int`: Ogni puntatore punta a un int

**Quindi:** "array_of_ptrs è un array di 3 puntatori a int"

**Diagramma di memoria:**
```
a @ 0x1000:  [1]
b @ 0x1004:  [2]
c @ 0x1008:  [3]

array_of_ptrs:
┌─────────┬─────────┬─────────┐
│ 0x1000  │ 0x1004  │ 0x1008  │  ← Array di 3 indirizzi
└────┬────┴────┬────┴────┬────┘
     │         │         │
     ▼         ▼         ▼
    [1]       [2]       [3]
```

**Memoria effettiva:**
```
0x2000: array_of_ptrs[0] = 0x1000  →  punta a 'a' (valore 1)
0x2008: array_of_ptrs[1] = 0x1004  →  punta a 'b' (valore 2)
0x2010: array_of_ptrs[2] = 0x1008  →  punta a 'c' (valore 3)
```

**Accesso:**
```c
array_of_ptrs[0]     // Valore: 0x1000 (indirizzo di a)
*array_of_ptrs[0]    // Valore: 1 (valore di a)
array_of_ptrs[1]     // Valore: 0x1004 (indirizzo di b)
*array_of_ptrs[1]    // Valore: 2 (valore di b)
```

**Dimensione in memoria:**
- Su sistema a 64 bit: 3 puntatori × 8 byte = 24 byte
- Su sistema a 32 bit: 3 puntatori × 4 byte = 12 byte

**Uso tipico:**
```c
// Array di stringhe
char *nomi[3] = {"Alice", "Bob", "Charlie"};

// Array di funzioni (callback)
void (*funzioni[3])(void) = {func1, func2, func3};
```

**2. Puntatore ad Array: `int (*ptr_to_array)[3]`**

**Dichiarazione analizzata:**
```c
int (*ptr_to_array)[3];
```

**Come leggere (le parentesi sono cruciali!):**
1. `(*ptr_to_array)`: ptr_to_array è un puntatore
2. `[3]`: Punta a un array di 3 elementi
3. `int`: Gli elementi sono int

**Quindi:** "ptr_to_array è un puntatore a un array di 3 int"

**Diagramma di memoria:**
```
arr @ 0x3000:
┌────┬────┬────┐
│ 10 │ 20 │ 30 │  ← Array di 3 int contigui
└────┴────┴────┘

ptr_to_array @ 0x4000:
┌──────────┐
│ 0x3000   │─────────────┐
└──────────┘             │
                         ▼
                    ┌────┬────┬────┐
                    │ 10 │ 20 │ 30 │
                    └────┴────┴────┘
```

**Differenza chiave:**
```c
int *p = arr;              // p punta al PRIMO ELEMENTO (arr[0])
int (*pa)[3] = &arr;       // pa punta all'INTERO ARRAY
```

**Aritmetica dei puntatori:**
```c
int arr1[3] = {10, 20, 30};
int arr2[3] = {40, 50, 60};

int *p = arr1;             // p punta a arr1[0]
p++;                       // p avanza di 1 int (4 byte) → punta a arr1[1]

int (*pa)[3] = &arr1;      // pa punta all'intero array arr1
pa++;                      // pa avanza di 1 array (12 byte) → punta a arr2
```

**Diagramma aritmetica:**
```
arr1:  [10][20][30]    arr2:  [40][50][60]
       ▲                      ▲
       │                      │
       pa                     pa++
    (0x3000)               (0x300C = 0x3000 + 12)
```

**3. Array Multidimensionale e Puntatori**

**Dichiarazione:**
```c
int matrix[2][3] = {{1, 2, 3}, {4, 5, 6}};
int (*ptr_to_matrix)[3] = matrix;
```

**Memoria della matrice:**
```
matrix @ 0x5000:
Riga 0: [1][2][3]
Riga 1: [4][5][6]

Layout lineare in memoria:
0x5000: 1
0x5004: 2
0x5008: 3
0x500C: 4
0x5010: 5
0x5014: 6
```

**Rappresentazione:**
```
matrix        ← Decade a puntatore alla prima riga
│
▼
┌───┬───┬───┐
│ 1 │ 2 │ 3 │  ← matrix[0] o *matrix
├───┼───┼───┤
│ 4 │ 5 │ 6 │  ← matrix[1] o *(matrix+1)
└───┴───┴───┘
```

**Tipo di `matrix`:**
- `matrix`: Tipo `int (*)[3]` (puntatore ad array di 3 int)
- `matrix[0]`: Tipo `int *` (puntatore a int)
- `matrix[0][0]`: Tipo `int` (int singolo)

**Equivalenze:**
```c
matrix[i][j]           // Notazione array standard
(*(matrix + i))[j]     // Parzialmente espansa
*(*(matrix + i) + j)   // Completamente espansa
```

**Passo per passo per `matrix[1][2]`:**
```c
matrix[1][2]
= (*(matrix + 1))[2]     // Sostituisci matrix[1]
= *(*(matrix + 1) + 2)   // Sostituisci [2]
```

**Calcolo indirizzi:**
1. `matrix` → 0x5000 (indirizzo riga 0)
2. `matrix + 1` → 0x500C (0x5000 + 1×12 byte) (indirizzo riga 1)
3. `*(matrix + 1)` → Dereferenzia, ottieni puntatore al primo elemento della riga 1
4. `*(matrix + 1) + 2` → 0x5014 (0x500C + 2×4 byte)
5. `*(*(matrix + 1) + 2)` → Valore a 0x5014 → 6

#### Confronto Visivo Completo

**Sintassi:**
```c
int *arr[3];        // Array di puntatori
int (*ptr)[3];      // Puntatore ad array
```

**Regola mnemonica:**
- `[]` ha precedenza su `*`
- Usa `()` per cambiare la precedenza

**Esempi di precedenza:**
```c
int *p[5];          // [] prima → array DI puntatori
int (*p)[5];        // () prima → puntatore A array
int *(*p)[5];       // Puntatore a (array di puntatori)
int (*p[5])[3];     // Array di (puntatori ad array)
```

#### Esempio Pratico: Argomenti della Linea di Comando

```c
#include <stdio.h>

/* main riceve un array di puntatori a char (stringhe) */
int main(int argc, char *argv[])  // o: char **argv
{
    printf("Numero di argomenti: %d\n", argc);
    
    /* argv è un array di puntatori */
    for (int i = 0; i < argc; i++) {
        printf("argv[%d] = %s\n", i, argv[i]);
    }
    
    /* Dimostra che argv è array di puntatori */
    printf("\nIndirizzi:\n");
    for (int i = 0; i < argc; i++) {
        printf("argv[%d] si trova a: %p\n", i, (void*)argv[i]);
    }
    
    return 0;
}
```

**Esecuzione:**
```bash
$ ./program hello world
Numero di argomenti: 3
argv[0] = ./program
argv[1] = hello
argv[2] = world

Indirizzi:
argv[0] si trova a: 0x7ffc1234abcd
argv[1] si trova a: 0x7ffc1234abd8
argv[2] si trova a: 0x7ffc1234abe0
```

**Memoria di argv:**
```
argv:
┌──────┬──────┬──────┬──────┐
│ ptr0 │ ptr1 │ ptr2 │ NULL │  ← Array di puntatori
└──┬───┴──┬───┴──┬───┴──────┘
   │      │      │
   ▼      ▼      ▼
"./program" "hello" "world"
```

**Compilazione e test:**
```bash
gcc -Wall -Wextra puntatori_avanzati.c -o puntatori_avanzati
./puntatori_avanzati arg1 arg2
```

### Puntatori Const

I puntatori const sono fondamentali per scrivere codice sicuro e prevenire modifiche accidentali.

```c
#include <stdio.h>

int main(void)
{
    int x = 10, y = 20;
    
    /* Puntatore a costante - NON puoi modificare il valore puntato */
    const int *ptr1 = &x;
    // *ptr1 = 100;  // ERRORE!
    ptr1 = &y;  // OK - puoi cambiare a cosa punta
    
    /* Puntatore costante - NON puoi cambiare a cosa punta */
    int *const ptr2 = &x;
    *ptr2 = 100;  // OK - puoi modificare il valore
    // ptr2 = &y;  // ERRORE!
    
    /* Puntatore costante a costante */
    const int *const ptr3 = &x;
    // *ptr3 = 100;  // ERRORE!
    // ptr3 = &y;     // ERRORE!
    
    printf("x = %d, y = %d\n", x, y);
    
    /* Regola mnemonica:
     * Leggi da destra a sinistra:
     * const int *ptr    -> "ptr è un puntatore a int costante"
     * int *const ptr    -> "ptr è un puntatore costante a int"
     * const int *const ptr -> "ptr è un puntatore costante a int costante"
     */
    
    return 0;
}
```

#### Spiegazione Dettagliata dei 3 Tipi di Const

**Tipo 1: Puntatore a Const (Pointer to Const)**

**Sintassi:**
```c
const int *ptr1 = &x;
// Equivalente a:
int const *ptr1 = &x;
```

**Diagramma:**
```
     ptr1                     x
   ┌──────┐               ┌──────┐
   │ addr │──────────────>│  10  │  ← NON modificabile tramite ptr1
   └──────┘               └──────┘
   
   Modificabile           Valore protetto
```

**Cosa puoi fare:**
```c
ptr1 = &y;     // ✅ OK: cambia dove punta ptr1
printf("%d", *ptr1);  // ✅ OK: leggi il valore
```

**Cosa NON puoi fare:**
```c
*ptr1 = 100;   // ❌ ERRORE: non puoi modificare il valore tramite ptr1
```

**Nota importante:**
```c
const int *ptr = &x;
*ptr = 100;    // ERRORE di compilazione
x = 100;       // OK! x non è const, solo l'accesso tramite ptr è const
```

**Uso tipico: Parametri funzione read-only**
```c
void stampa_array(const int *arr, int size)
{
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);    // ✅ Lettura OK
        // arr[i] = 0;            // ❌ ERRORE: non puoi modificare
    }
}
```

**Vantaggi:**
- Previene modifiche accidentali
- Documenta l'intento (funzione non modifica i dati)
- Permette al compilatore di ottimizzare
- Permette di passare array const

**Tipo 2: Puntatore Const (Const Pointer)**

**Sintassi:**
```c
int *const ptr2 = &x;
```

**Diagramma:**
```
     ptr2 (FISSO)            x
   ┌──────┐               ┌──────┐
   │ addr │──────────────>│  10  │  ← Modificabile tramite ptr2
   └──────┘               └──────┘
   
   NON modificabile       Valore modificabile
   (come riferimento C++)
```

**Cosa puoi fare:**
```c
*ptr2 = 100;   // ✅ OK: modifica il valore puntato
printf("%d", *ptr2);  // ✅ OK: leggi il valore
```

**Cosa NON puoi fare:**
```c
ptr2 = &y;     // ❌ ERRORE: non puoi cambiare dove punta ptr2
ptr2++;        // ❌ ERRORE: non puoi muovere il puntatore
```

**Uso tipico: Puntatore che non deve mai cambiare target**
```c
struct Config {
    int value;
};

void process(struct Config *const config)
{
    config->value = 42;    // ✅ OK: modifica contenuto
    // config = NULL;      // ❌ ERRORE: non puoi cambiare puntatore
}
```

**Analogia:**
- Come un riferimento C++ o `final` in Java
- Il puntatore è "bloccato" a una variabile specifica

**Tipo 3: Puntatore Const a Const (Const Pointer to Const)**

**Sintassi:**
```c
const int *const ptr3 = &x;
```

**Diagramma:**
```
     ptr3 (FISSO)            x
   ┌──────┐               ┌──────┐
   │ addr │──────────────>│  10  │  ← Protetto
   └──────┘               └──────┘
   
   NON modificabile       NON modificabile tramite ptr3
   (massima protezione)
```

**Cosa puoi fare:**
```c
printf("%d", *ptr3);  // ✅ OK: solo lettura
```

**Cosa NON puoi fare:**
```c
*ptr3 = 100;   // ❌ ERRORE: non puoi modificare il valore
ptr3 = &y;     // ❌ ERRORE: non puoi cambiare dove punta
ptr3++;        // ❌ ERRORE: non puoi muovere il puntatore
```

**Uso tipico: Costanti immutabili**
```c
const int *const DAYS_IN_WEEK = &(int){7};

void func(const char *const message)
{
    // message è completamente read-only
    printf("%s", message);
}
```

#### Tabella di Confronto Completa

| Dichiarazione | Puntatore Modificabile? | Valore Modificabile? | Nome |
|---------------|------------------------|---------------------|------|
| `int *ptr` | ✅ Sì | ✅ Sì | Puntatore normale |
| `const int *ptr` | ✅ Sì | ❌ No | Puntatore a const |
| `int *const ptr` | ❌ No | ✅ Sì | Puntatore const |
| `const int *const ptr` | ❌ No | ❌ No | Puntatore const a const |

#### Regola di Lettura: Spirale Clockwise/Right-Left

**Tecnica: Leggi da destra a sinistra, saltando quando trovi `*`**

**Esempio 1:**
```c
const int *ptr;
```
Lettura: `ptr` → `*` (è un puntatore a) → `int` → `const` (int costante)
**Risultato:** "ptr è un puntatore a int const"

**Esempio 2:**
```c
int *const ptr;
```
Lettura: `ptr` → `const` (è const) → `*` (puntatore a) → `int`
**Risultato:** "ptr è un puntatore const a int"

**Esempio 3:**
```c
const int *const ptr;
```
Lettura: `ptr` → `const` (è const) → `*` (puntatore a) → `int const`
**Risultato:** "ptr è un puntatore const a int const"

#### Posizione di `const` - Due Sintassi Equivalenti

```c
/* Queste due sono IDENTICHE: */
const int *ptr;    // Più comune
int const *ptr;    // Meno comune ma valida

/* Entrambe significano: "ptr è un puntatore a int const" */
```

**Regola pratica:**
- `const` prima del tipo o dopo il tipo: protegge il VALORE
- `const` dopo `*`: protegge il PUNTATORE

#### Esempio Pratico: Funzioni con Const Correttezza

```c
#include <stdio.h>
#include <string.h>

/* Buona pratica: parametri read-only con const */
size_t mia_strlen(const char *str)
{
    const char *ptr = str;  // ptr è puntatore a char const
    
    while (*ptr != '\0') {
        ptr++;    // OK: possiamo muovere il puntatore
        // *ptr = 'X';  // ERRORE: non possiamo modificare
    }
    
    return ptr - str;
}

/* Funzione che modifica il contenuto */
void to_uppercase(char *str)  // NON const: modifica permessa
{
    while (*str) {
        if (*str >= 'a' && *str <= 'z') {
            *str = *str - 32;  // OK: str non è const
        }
        str++;
    }
}

/* Versione const-corretta di strcpy */
char *mia_strcpy(char *dest, const char *src)
{
    char *original_dest = dest;  // Salva l'indirizzo iniziale
    
    while (*src != '\0') {
        *dest = *src;  // OK: dest non è const, src è const
        dest++;
        src++;
    }
    *dest = '\0';
    
    return original_dest;
}

int main(void)
{
    const char *message = "Hello";  // Stringa letterale (const)
    char buffer[50] = "test";
    
    /* OK: mia_strlen non modifica */
    printf("Lunghezza: %zu\n", mia_strlen(message));
    
    /* OK: buffer non è const */
    to_uppercase(buffer);
    printf("Maiuscolo: %s\n", buffer);
    
    /* OK: dest non const, src const */
    char dest[50];
    mia_strcpy(dest, message);
    printf("Copiato: %s\n", dest);
    
    return 0;
}
```

#### Conversioni di Const: Cosa è Permesso

**✅ Conversioni VALIDE (aggiungere const):**
```c
int x = 10;
int *p = &x;

const int *cp = p;        // OK: aggiungi const
const int *cp2 = &x;      // OK: aggiungi const
```

**❌ Conversioni NON VALIDE (rimuovere const):**
```c
const int cx = 10;
const int *cp = &cx;

int *p = cp;              // ERRORE: rimuovi const (warning)
int *p2 = &cx;            // ERRORE: rimuovi const (warning)
```

**⚠️ Cast esplicito (pericoloso!):**
```c
const int cx = 10;
int *p = (int *)&cx;      // Compila ma COMPORTAMENTO INDEFINITO!
*p = 20;                  // Potrebbe crashare o dare risultati strani
```

#### Best Practices con Const

**1. Usa const per parametri read-only:**
```c
// ✅ Buono
void process(const Data *data);

// ❌ Cattivo (manca const)
void process(Data *data);  // Implica che data può essere modificato
```

**2. Restituisci const per prevenire modifiche:**
```c
// ✅ Buono
const char *get_name(void);

// L'utente non può fare:
char *name = get_name();
name[0] = 'X';  // Warning: assegnamento da const a non-const
```

**3. Const in struct:**
```c
typedef struct {
    const int id;           // id immutabile dopo inizializzazione
    char name[50];
} Person;

Person p = {1, "Alice"};
// p.id = 2;                // ERRORE: id è const
strcpy(p.name, "Bob");     // OK: name non è const
```

**Compilazione e test:**
```bash
gcc -Wall -Wextra const_pointers.c -o const_pointers
./const_pointers
```

### Puntatori Void e Casting

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* Funzione generica di swap usando void* */
void swap_generic(void *a, void *b, size_t size)
{
    void *temp = malloc(size);
    if (temp == NULL) return;
    
    memcpy(temp, a, size);
    memcpy(a, b, size);
    memcpy(b, temp, size);
    
    free(temp);
}

/* Funzione di confronto generica per qsort */
int compare_int(const void *a, const void *b)
{
    return (*(int*)a - *(int*)b);
}

int compare_float(const void *a, const void *b)
{
    float diff = *(float*)a - *(float*)b;
    return (diff > 0) - (diff < 0);
}

int main(void)
{
    /* Swap con void* */
    int x = 10, y = 20;
    printf("Prima: x=%d, y=%d\n", x, y);
    swap_generic(&x, &y, sizeof(int));
    printf("Dopo: x=%d, y=%d\n\n", x, y);
    
    double d1 = 3.14, d2 = 2.71;
    printf("Prima: d1=%.2f, d2=%.2f\n", d1, d2);
    swap_generic(&d1, &d2, sizeof(double));
    printf("Dopo: d1=%.2f, d2=%.2f\n\n", d1, d2);
    
    /* Uso di qsort con void* */
    int arr_int[] = {5, 2, 8, 1, 9, 3};
    int size_int = sizeof(arr_int) / sizeof(arr_int[0]);
    
    printf("Array prima di qsort: ");
    for (int i = 0; i < size_int; i++) printf("%d ", arr_int[i]);
    printf("\n");
    
    qsort(arr_int, size_int, sizeof(int), compare_int);
    
    printf("Array dopo qsort: ");
    for (int i = 0; i < size_int; i++) printf("%d ", arr_int[i]);
    printf("\n");
    
    return 0;
}
```

## Manipolazione Bit

### Operazioni Bit Comuni

```c
#include <stdio.h>
#include <stdint.h>

void print_bits(uint32_t num)
{
    for (int i = 31; i >= 0; i--) {
        printf("%d", (num >> i) & 1);
        if (i % 8 == 0) printf(" ");
    }
    printf("\n");
}

/* Imposta bit n a 1 */
uint32_t set_bit(uint32_t num, int n)
{
    return num | (1U << n);
}

/* Imposta bit n a 0 */
uint32_t clear_bit(uint32_t num, int n)
{
    return num & ~(1U << n);
}

/* Toggle bit n */
uint32_t toggle_bit(uint32_t num, int n)
{
    return num ^ (1U << n);
}

/* Controlla se bit n è 1 */
int check_bit(uint32_t num, int n)
{
    return (num >> n) & 1;
}

/* Conta bit a 1 (population count) */
int popcount(uint32_t num)
{
    int count = 0;
    while (num) {
        count += num & 1;
        num >>= 1;
    }
    return count;
}

/* Algoritmo di Brian Kernighan per popcount */
int popcount_fast(uint32_t num)
{
    int count = 0;
    while (num) {
        num &= num - 1;  // Rimuove il bit 1 più a destra
        count++;
    }
    return count;
}

/* Inverti bit */
uint32_t reverse_bits(uint32_t num)
{
    uint32_t result = 0;
    for (int i = 0; i < 32; i++) {
        result <<= 1;
        result |= num & 1;
        num >>= 1;
    }
    return result;
}

int main(void)
{
    uint32_t num = 0b10110010;  // 178 in decimale
    
    printf("Numero iniziale (%u): ", num);
    print_bits(num);
    
    num = set_bit(num, 0);
    printf("Dopo set bit 0:    ");
    print_bits(num);
    
    num = clear_bit(num, 1);
    printf("Dopo clear bit 1:  ");
    print_bits(num);
    
    num = toggle_bit(num, 5);
    printf("Dopo toggle bit 5: ");
    print_bits(num);
    
    printf("\nBit 4 è %s\n", check_bit(num, 4) ? "set" : "clear");
    
    printf("Numero di bit a 1: %d\n", popcount(num));
    printf("Numero di bit a 1 (fast): %d\n", popcount_fast(num));
    
    uint32_t reversed = reverse_bits(num);
    printf("\nOriginale: ");
    print_bits(num);
    printf("Invertito: ");
    print_bits(reversed);
    
    return 0;
}
```

### Bit Fields

```c
#include <stdio.h>
#include <stdint.h>

/* Bit fields nelle strutture */
struct Flags {
    unsigned int flag1 : 1;  // 1 bit
    unsigned int flag2 : 1;  // 1 bit
    unsigned int flag3 : 1;  // 1 bit
    unsigned int unused : 5; // 5 bit non usati
};

/* Esempio pratico: RGB color */
typedef struct {
    uint32_t red   : 8;  // 8 bit per rosso
    uint32_t green : 8;  // 8 bit per verde
    uint32_t blue  : 8;  // 8 bit per blu
    uint32_t alpha : 8;  // 8 bit per alpha
} Color;

/* Esempio: registro hardware simulato */
typedef struct {
    uint8_t enable    : 1;  // bit 0
    uint8_t mode      : 2;  // bit 1-2
    uint8_t priority  : 3;  // bit 3-5
    uint8_t reserved  : 2;  // bit 6-7
} RegisterControl;

int main(void)
{
    struct Flags f = {0};
    printf("Dimensione struct Flags: %lu byte\n", sizeof(f));
    
    f.flag1 = 1;
    f.flag2 = 0;
    f.flag3 = 1;
    
    printf("flag1=%u, flag2=%u, flag3=%u\n", f.flag1, f.flag2, f.flag3);
    
    /* RGB Color */
    Color red = {255, 0, 0, 255};
    Color green = {0, 255, 0, 255};
    Color blue = {0, 0, 255, 255};
    
    printf("\nColor size: %lu bytes\n", sizeof(Color));
    printf("Red: R=%u G=%u B=%u A=%u\n", red.red, red.green, red.blue, red.alpha);
    
    /* Registro hardware */
    RegisterControl reg = {
        .enable = 1,
        .mode = 2,    // 0-3
        .priority = 5, // 0-7
        .reserved = 0
    };
    
    printf("\nRegistro: enable=%u, mode=%u, priority=%u\n", 
           reg.enable, reg.mode, reg.priority);
    
    return 0;
}
```

### Maschere di Bit

```c
#include <stdio.h>
#include <stdint.h>

/* Definizione di flag */
#define FLAG_READ    0x01  // 0000 0001
#define FLAG_WRITE   0x02  // 0000 0010
#define FLAG_EXECUTE 0x04  // 0000 0100
#define FLAG_HIDDEN  0x08  // 0000 1000

/* Macro per operazioni su flag */
#define HAS_FLAG(val, flag)    ((val) & (flag))
#define SET_FLAG(val, flag)    ((val) |= (flag))
#define CLEAR_FLAG(val, flag)  ((val) &= ~(flag))
#define TOGGLE_FLAG(val, flag) ((val) ^= (flag))

/* Estrai byte da word */
#define BYTE_0(x) ((x) & 0xFF)
#define BYTE_1(x) (((x) >> 8) & 0xFF)
#define BYTE_2(x) (((x) >> 16) & 0xFF)
#define BYTE_3(x) (((x) >> 24) & 0xFF)

int main(void)
{
    uint8_t permissions = 0;
    
    /* Set flags */
    SET_FLAG(permissions, FLAG_READ);
    SET_FLAG(permissions, FLAG_WRITE);
    
    printf("Permissions: 0x%02X\n", permissions);
    
    /* Check flags */
    if (HAS_FLAG(permissions, FLAG_READ)) {
        printf("Has READ permission\n");
    }
    if (HAS_FLAG(permissions, FLAG_WRITE)) {
        printf("Has WRITE permission\n");
    }
    if (!HAS_FLAG(permissions, FLAG_EXECUTE)) {
        printf("Does NOT have EXECUTE permission\n");
    }
    
    /* Toggle flag */
    TOGGLE_FLAG(permissions, FLAG_HIDDEN);
    printf("After toggle HIDDEN: 0x%02X\n", permissions);
    
    /* Clear flag */
    CLEAR_FLAG(permissions, FLAG_WRITE);
    printf("After clearing WRITE: 0x%02X\n", permissions);
    
    /* Estrai byte da intero */
    uint32_t value = 0x12345678;
    printf("\nValue: 0x%08X\n", value);
    printf("Byte 0: 0x%02X\n", BYTE_0(value));
    printf("Byte 1: 0x%02X\n", BYTE_1(value));
    printf("Byte 2: 0x%02X\n", BYTE_2(value));
    printf("Byte 3: 0x%02X\n", BYTE_3(value));
    
    return 0;
}
```

## Ottimizzazione del Codice

### Ottimizzazioni del Compilatore

```c
#include <stdio.h>
#include <time.h>

/* Funzione NON ottimizzata */
int somma_array_slow(int arr[], int size)
{
    int sum = 0;
    for (int i = 0; i < size; i++) {
        sum = sum + arr[i];
    }
    return sum;
}

/* Funzione ottimizzata - loop unrolling manuale */
int somma_array_fast(int arr[], int size)
{
    int sum = 0;
    int i = 0;
    
    /* Processa 4 elementi alla volta */
    for (; i < size - 3; i += 4) {
        sum += arr[i];
        sum += arr[i+1];
        sum += arr[i+2];
        sum += arr[i+3];
    }
    
    /* Processa elementi rimanenti */
    for (; i < size; i++) {
        sum += arr[i];
    }
    
    return sum;
}

/* Inline function per calcoli semplici */
static inline int square(int x)
{
    return x * x;
}

/* Restrict keyword per ottimizzazione puntatori */
void copy_array(int *restrict dest, const int *restrict src, int size)
{
    for (int i = 0; i < size; i++) {
        dest[i] = src[i];
    }
}

int main(void)
{
    const int SIZE = 10000000;
    int *arr = malloc(SIZE * sizeof(int));
    
    for (int i = 0; i < SIZE; i++) {
        arr[i] = i;
    }
    
    /* Benchmark */
    clock_t start, end;
    
    start = clock();
    int sum1 = somma_array_slow(arr, SIZE);
    end = clock();
    double time_slow = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    start = clock();
    int sum2 = somma_array_fast(arr, SIZE);
    end = clock();
    double time_fast = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("Somma (slow): %d, Tempo: %.6f s\n", sum1, time_slow);
    printf("Somma (fast): %d, Tempo: %.6f s\n", sum2, time_fast);
    printf("Speedup: %.2fx\n", time_slow / time_fast);
    
    free(arr);
    
    /* Test inline */
    int result = square(5);
    printf("\nSquare of 5: %d\n", result);
    
    return 0;
}
```

### Cache-Friendly Code

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define SIZE 1000

/* NON cache-friendly - accesso per colonne */
void sum_columns_bad(int matrix[SIZE][SIZE])
{
    for (int j = 0; j < SIZE; j++) {
        int sum = 0;
        for (int i = 0; i < SIZE; i++) {
            sum += matrix[i][j];  // Cache miss frequenti!
        }
    }
}

/* Cache-friendly - accesso per righe */
void sum_rows_good(int matrix[SIZE][SIZE])
{
    for (int i = 0; i < SIZE; i++) {
        int sum = 0;
        for (int j = 0; j < SIZE; j++) {
            sum += matrix[i][j];  // Cache hit più frequenti
        }
    }
}

/* Structure of Arrays (SoA) - cache friendly */
typedef struct {
    float *x;
    float *y;
    float *z;
} Points_SoA;

/* Array of Structures (AoS) - meno cache friendly */
typedef struct {
    float x;
    float y;
    float z;
} Point;

typedef struct {
    Point *points;
} Points_AoS;

int main(void)
{
    int (*matrix)[SIZE] = malloc(sizeof(int[SIZE][SIZE]));
    
    /* Inizializza */
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            matrix[i][j] = i * j;
        }
    }
    
    clock_t start, end;
    
    start = clock();
    sum_columns_bad(matrix);
    end = clock();
    double time_bad = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    start = clock();
    sum_rows_good(matrix);
    end = clock();
    double time_good = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("Column access: %.6f s\n", time_bad);
    printf("Row access: %.6f s\n", time_good);
    printf("Speedup: %.2fx\n", time_bad / time_good);
    
    free(matrix);
    
    return 0;
}
```

## Gestione Errori Avanzata

### Error Codes e errno

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>

/* Custom error codes */
typedef enum {
    SUCCESS = 0,
    ERROR_NULL_POINTER = -1,
    ERROR_OUT_OF_MEMORY = -2,
    ERROR_INVALID_INPUT = -3,
    ERROR_FILE_NOT_FOUND = -4
} ErrorCode;

const char* error_string(ErrorCode code)
{
    switch (code) {
        case SUCCESS: return "Success";
        case ERROR_NULL_POINTER: return "Null pointer error";
        case ERROR_OUT_OF_MEMORY: return "Out of memory";
        case ERROR_INVALID_INPUT: return "Invalid input";
        case ERROR_FILE_NOT_FOUND: return "File not found";
        default: return "Unknown error";
    }
}

/* Funzione che usa errno */
void read_file_example(const char *filename)
{
    FILE *file = fopen(filename, "r");
    if (file == NULL) {
        fprintf(stderr, "Error opening file: %s\n", strerror(errno));
        switch (errno) {
            case ENOENT:
                fprintf(stderr, "File does not exist\n");
                break;
            case EACCES:
                fprintf(stderr, "Permission denied\n");
                break;
            default:
                fprintf(stderr, "Other error (errno=%d)\n", errno);
        }
        return;
    }
    
    fclose(file);
}

/* Funzione con error code */
ErrorCode allocate_and_initialize(int **arr, int size)
{
    if (size <= 0) {
        return ERROR_INVALID_INPUT;
    }
    
    *arr = (int*)malloc(size * sizeof(int));
    if (*arr == NULL) {
        return ERROR_OUT_OF_MEMORY;
    }
    
    for (int i = 0; i < size; i++) {
        (*arr)[i] = 0;
    }
    
    return SUCCESS;
}

int main(void)
{
    int *array;
    ErrorCode result = allocate_and_initialize(&array, 100);
    
    if (result != SUCCESS) {
        fprintf(stderr, "Error: %s\n", error_string(result));
        return 1;
    }
    
    printf("Array allocated successfully\n");
    free(array);
    
    /* Test errno */
    read_file_example("nonexistent.txt");
    
    return 0;
}
```

### Assertions e Defensive Programming

```c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <stdint.h>

/* Contract programming con assert */
int divide(int a, int b)
{
    assert(b != 0 && "Divisor cannot be zero");
    return a / b;
}

/* Pre-condizioni, post-condizioni, invarianti */
void sort_array(int *arr, int size)
{
    /* Pre-condizione */
    assert(arr != NULL && "Array cannot be NULL");
    assert(size >= 0 && "Size must be non-negative");
    
    /* Algoritmo di sorting... */
    for (int i = 0; i < size - 1; i++) {
        for (int j = 0; j < size - i - 1; j++) {
            if (arr[j] > arr[j+1]) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }
    
    /* Post-condizione: array ordinato */
    for (int i = 0; i < size - 1; i++) {
        assert(arr[i] <= arr[i+1] && "Array must be sorted");
    }
}

/* Static assertions (compile-time) */
_Static_assert(sizeof(int) >= 4, "int must be at least 4 bytes");
_Static_assert(sizeof(void*) == 8, "This code requires 64-bit pointers");

/* Defensive programming */
int* safe_allocate(size_t size)
{
    if (size == 0 || size > SIZE_MAX / sizeof(int)) {
        return NULL;
    }
    
    int *ptr = (int*)malloc(size * sizeof(int));
    if (ptr == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return NULL;
    }
    
    /* Inizializza a valore sicuro */
    for (size_t i = 0; i < size; i++) {
        ptr[i] = 0;
    }
    
    return ptr;
}

int main(void)
{
    /* Test assertions */
    printf("10 / 2 = %d\n", divide(10, 2));
    // printf("10 / 0 = %d\n", divide(10, 0));  // Assertion failure!
    
    /* Test sort */
    int arr[] = {5, 2, 8, 1, 9};
    int size = sizeof(arr) / sizeof(arr[0]);
    
    printf("Before sort: ");
    for (int i = 0; i < size; i++) printf("%d ", arr[i]);
    printf("\n");
    
    sort_array(arr, size);
    
    printf("After sort: ");
    for (int i = 0; i < size; i++) printf("%d ", arr[i]);
    printf("\n");
    
    /* Test safe allocation */
    int *safe_arr = safe_allocate(100);
    if (safe_arr != NULL) {
        printf("Safe allocation successful\n");
        free(safe_arr);
    }
    
    return 0;
}
```

## Multi-threading con pthreads

### Thread Base

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

/* Funzione eseguita dal thread */
void* thread_function(void *arg)
{
    int thread_id = *(int*)arg;
    printf("Thread %d: started\n", thread_id);
    
    /* Simula lavoro */
    sleep(2);
    
    printf("Thread %d: finished\n", thread_id);
    return NULL;
}

/* Thread con valore di ritorno */
void* calculate_sum(void *arg)
{
    int n = *(int*)arg;
    int *result = malloc(sizeof(int));
    
    *result = 0;
    for (int i = 1; i <= n; i++) {
        *result += i;
    }
    
    return result;
}

int main(void)
{
    pthread_t threads[5];
    int thread_ids[5];
    
    /* Crea threads */
    printf("Creating threads...\n");
    for (int i = 0; i < 5; i++) {
        thread_ids[i] = i + 1;
        if (pthread_create(&threads[i], NULL, thread_function, &thread_ids[i]) != 0) {
            fprintf(stderr, "Error creating thread %d\n", i);
            return 1;
        }
    }
    
    /* Attendi completamento */
    printf("Waiting for threads...\n");
    for (int i = 0; i < 5; i++) {
        pthread_join(threads[i], NULL);
    }
    
    printf("\nAll threads completed\n\n");
    
    /* Thread con ritorno */
    pthread_t calc_thread;
    int n = 100;
    pthread_create(&calc_thread, NULL, calculate_sum, &n);
    
    int *sum;
    pthread_join(calc_thread, (void**)&sum);
    printf("Sum 1 to %d = %d\n", n, *sum);
    free(sum);
    
    return 0;
}
```

### Sincronizzazione - Mutex

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define NUM_THREADS 10
#define INCREMENTS 100000

/* Variabile condivisa */
int counter = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

/* Thread senza sincronizzazione - RACE CONDITION */
void* increment_unsafe(void *arg)
{
    for (int i = 0; i < INCREMENTS; i++) {
        counter++;  // Non atomico!
    }
    return NULL;
}

/* Thread con mutex - SICURO */
void* increment_safe(void *arg)
{
    for (int i = 0; i < INCREMENTS; i++) {
        pthread_mutex_lock(&mutex);
        counter++;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void test_unsafe(void)
{
    pthread_t threads[NUM_THREADS];
    counter = 0;
    
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, increment_unsafe, NULL);
    }
    
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }
    
    printf("Unsafe counter (expected %d): %d\n", NUM_THREADS * INCREMENTS, counter);
}

void test_safe(void)
{
    pthread_t threads[NUM_THREADS];
    counter = 0;
    
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, increment_safe, NULL);
    }
    
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }
    
    printf("Safe counter (expected %d): %d\n", NUM_THREADS * INCREMENTS, counter);
}

int main(void)
{
    printf("Testing without mutex (race condition):\n");
    test_unsafe();
    
    printf("\nTesting with mutex (safe):\n");
    test_safe();
    
    pthread_mutex_destroy(&mutex);
    return 0;
}
```

### Condition Variables

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define BUFFER_SIZE 5

typedef struct {
    int buffer[BUFFER_SIZE];
    int count;
    int in;
    int out;
    pthread_mutex_t mutex;
    pthread_cond_t not_empty;
    pthread_cond_t not_full;
} BoundedBuffer;

void buffer_init(BoundedBuffer *bb)
{
    bb->count = 0;
    bb->in = 0;
    bb->out = 0;
    pthread_mutex_init(&bb->mutex, NULL);
    pthread_cond_init(&bb->not_empty, NULL);
    pthread_cond_init(&bb->not_full, NULL);
}

void buffer_put(BoundedBuffer *bb, int item)
{
    pthread_mutex_lock(&bb->mutex);
    
    /* Attendi se buffer pieno */
    while (bb->count == BUFFER_SIZE) {
        pthread_cond_wait(&bb->not_full, &bb->mutex);
    }
    
    bb->buffer[bb->in] = item;
    bb->in = (bb->in + 1) % BUFFER_SIZE;
    bb->count++;
    
    printf("Produced: %d (count=%d)\n", item, bb->count);
    
    /* Segnala che buffer non è vuoto */
    pthread_cond_signal(&bb->not_empty);
    pthread_mutex_unlock(&bb->mutex);
}

int buffer_get(BoundedBuffer *bb)
{
    pthread_mutex_lock(&bb->mutex);
    
    /* Attendi se buffer vuoto */
    while (bb->count == 0) {
        pthread_cond_wait(&bb->not_empty, &bb->mutex);
    }
    
    int item = bb->buffer[bb->out];
    bb->out = (bb->out + 1) % BUFFER_SIZE;
    bb->count--;
    
    printf("Consumed: %d (count=%d)\n", item, bb->count);
    
    /* Segnala che buffer non è pieno */
    pthread_cond_signal(&bb->not_full);
    pthread_mutex_unlock(&bb->mutex);
    
    return item;
}

void* producer(void *arg)
{
    BoundedBuffer *bb = (BoundedBuffer*)arg;
    for (int i = 0; i < 10; i++) {
        buffer_put(bb, i);
        usleep(100000);  // 100ms
    }
    return NULL;
}

void* consumer(void *arg)
{
    BoundedBuffer *bb = (BoundedBuffer*)arg;
    for (int i = 0; i < 10; i++) {
        buffer_get(bb);
        usleep(150000);  // 150ms
    }
    return NULL;
}

int main(void)
{
    BoundedBuffer bb;
    buffer_init(&bb);
    
    pthread_t prod_thread, cons_thread;
    
    pthread_create(&prod_thread, NULL, producer, &bb);
    pthread_create(&cons_thread, NULL, consumer, &bb);
    
    pthread_join(prod_thread, NULL);
    pthread_join(cons_thread, NULL);
    
    pthread_mutex_destroy(&bb.mutex);
    pthread_cond_destroy(&bb.not_empty);
    pthread_cond_destroy(&bb.not_full);
    
    return 0;
}
```

## Gestione Memoria Avanzata

### Memory Pool

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct Block {
    struct Block *next;
} Block;

typedef struct {
    void *memory;
    Block *free_list;
    size_t block_size;
    size_t num_blocks;
} MemoryPool;

MemoryPool* pool_create(size_t block_size, size_t num_blocks)
{
    MemoryPool *pool = malloc(sizeof(MemoryPool));
    if (!pool) return NULL;
    
    pool->block_size = block_size;
    pool->num_blocks = num_blocks;
    pool->memory = malloc(block_size * num_blocks);
    
    if (!pool->memory) {
        free(pool);
        return NULL;
    }
    
    /* Inizializza free list */
    pool->free_list = NULL;
    char *mem = (char*)pool->memory;
    for (size_t i = 0; i < num_blocks; i++) {
        Block *block = (Block*)(mem + i * block_size);
        block->next = pool->free_list;
        pool->free_list = block;
    }
    
    return pool;
}

void* pool_alloc(MemoryPool *pool)
{
    if (!pool->free_list) {
        return NULL;  // Pool esaurito
    }
    
    Block *block = pool->free_list;
    pool->free_list = block->next;
    
    return (void*)block;
}

void pool_free(MemoryPool *pool, void *ptr)
{
    if (!ptr) return;
    
    Block *block = (Block*)ptr;
    block->next = pool->free_list;
    pool->free_list = block;
}

void pool_destroy(MemoryPool *pool)
{
    if (pool) {
        free(pool->memory);
        free(pool);
    }
}

int main(void)
{
    /* Crea pool di 64-byte blocks */
    MemoryPool *pool = pool_create(64, 10);
    
    /* Alloca alcuni blocchi */
    void *blocks[5];
    for (int i = 0; i < 5; i++) {
        blocks[i] = pool_alloc(pool);
        printf("Allocated block %d: %p\n", i, blocks[i]);
    }
    
    /* Libera alcuni blocchi */
    pool_free(pool, blocks[1]);
    pool_free(pool, blocks[3]);
    printf("\nFreed blocks 1 and 3\n\n");
    
    /* Rialloca */
    void *new_block = pool_alloc(pool);
    printf("Reallocated block: %p\n", new_block);
    
    /* Cleanup */
    for (int i = 0; i < 5; i++) {
        if (i != 1 && i != 3) {
            pool_free(pool, blocks[i]);
        }
    }
    pool_free(pool, new_block);
    
    pool_destroy(pool);
    
    return 0;
}
```

## Esercizi

1. Implementa un hash table thread-safe
2. Crea un sistema di logging con diversi livelli di errore
3. Scrivi un memory allocator personalizzato
4. Implementa un producer-consumer con multiple threads
5. Crea un sistema di bit manipulation per operazioni su grandi dataset

---

[← Intermedio](../03_Intermedio/README.md) | [Torna al Sommario](../README.md) | [Prossimo: Assembly →](../05_Assembly_LowLevel/README.md)
