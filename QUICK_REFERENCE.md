# Guida Rapida di Riferimento C

## Compilazione

```bash
# Compilazione base
gcc programma.c -o programma

# Con warnings
gcc -Wall -Wextra programma.c -o programma

# Con debugging
gcc -g programma.c -o programma

# Con ottimizzazione
gcc -O2 programma.c -o programma

# Specifica standard C
gcc -std=c11 programma.c -o programma

# Link con librerie matematiche
gcc programma.c -o programma -lm

# Link con pthread
gcc programma.c -o programma -lpthread
```

## Tipi di Dati

```c
/* Interi */
char            // 1 byte
short           // 2 byte
int             // 4 byte (tipicamente)
long            // 4 o 8 byte
long long       // 8 byte

/* Interi unsigned */
unsigned char   // 0 a 255
unsigned int    // 0 a 4,294,967,295

/* Interi a dimensione fissa (stdint.h) */
int8_t, uint8_t
int16_t, uint16_t
int32_t, uint32_t
int64_t, uint64_t

/* Float */
float           // 4 byte
double          // 8 byte
long double     // 10-16 byte

/* Altri */
bool            // C99+, richiede stdbool.h
void            // Tipo vuoto
```

## Operatori

```c
/* Aritmetici */
+ - * / %

/* Confronto */
== != < > <= >=

/* Logici */
&& || !

/* Bitwise */
& | ^ ~ << >>

/* Assegnamento */
= += -= *= /= %= &= |= ^= <<= >>=

/* Incremento/Decremento */
++ --

/* Puntatore */
& (indirizzo di)
* (dereferenzia)

/* Altro */
? : (ternario)
, (virgola)
sizeof
```

## Strutture di Controllo

```c
/* If-else */
if (condition) {
    // codice
} else if (other_condition) {
    // codice
} else {
    // codice
}

/* Switch */
switch (variable) {
    case VALUE1:
        // codice
        break;
    case VALUE2:
        // codice
        break;
    default:
        // codice
}

/* While */
while (condition) {
    // codice
}

/* Do-while */
do {
    // codice
} while (condition);

/* For */
for (init; condition; increment) {
    // codice
}

/* Break e Continue */
break;      // Esce dal loop
continue;   // Salta all'iterazione successiva
```

## Funzioni

```c
/* Dichiarazione */
tipo_ritorno nome_funzione(tipo1 param1, tipo2 param2);

/* Definizione */
int somma(int a, int b) {
    return a + b;
}

/* Funzione void */
void stampa(int n) {
    printf("%d\n", n);
}

/* Passaggio per riferimento */
void modifica(int *ptr) {
    *ptr = 100;
}

/* Array come parametro */
void processa(int arr[], int size) {
    // oppure: int *arr
}
```

## Array

```c
/* Dichiarazione */
int arr[10];
int arr[] = {1, 2, 3, 4, 5};
int matrix[3][4];

/* Accesso */
arr[0] = 10;
int val = arr[5];

/* Dimensione */
int size = sizeof(arr) / sizeof(arr[0]);

/* Iterazione */
for (int i = 0; i < size; i++) {
    printf("%d ", arr[i]);
}
```

## Stringhe

```c
/* Dichiarazione */
char str[20];
char str[] = "Hello";
char *str = "Hello";  // String literal

/* Funzioni comuni (string.h) */
strlen(str)              // Lunghezza
strcpy(dest, src)        // Copia
strcat(dest, src)        // Concatena
strcmp(str1, str2)       // Confronta
strstr(haystack, needle) // Cerca substring

/* Input/Output */
scanf("%s", str);        // Leggi (pericoloso!)
fgets(str, size, stdin); // Leggi sicuro
printf("%s", str);       // Stampa
```

## Puntatori

```c
/* Dichiarazione */
int *ptr;
int **ptr_ptr;  // Puntatore a puntatore

/* Operazioni */
ptr = &variable;  // Assegna indirizzo
*ptr = 10;        // Dereferenzia e assegna
int val = *ptr;   // Dereferenzia e leggi

/* Aritmetica */
ptr++;            // Avanza di un elemento
ptr--;            // Arretra di un elemento
ptr + n           // Avanza di n elementi
ptr1 - ptr2       // Distanza tra puntatori

/* NULL */
ptr = NULL;       // Puntatore nullo
if (ptr != NULL)  // Controlla validità
```

## Allocazione Dinamica

```c
#include <stdlib.h>

/* malloc - alloca NON inizializzata */
int *arr = (int*)malloc(10 * sizeof(int));

/* calloc - alloca INIZIALIZZATA a zero */
int *arr = (int*)calloc(10, sizeof(int));

/* realloc - ridimensiona */
arr = (int*)realloc(arr, 20 * sizeof(int));

/* free - libera memoria */
free(arr);
arr = NULL;  // Buona pratica

/* Controlla sempre NULL */
if (arr == NULL) {
    fprintf(stderr, "Allocazione fallita\n");
    return 1;
}
```

## Strutture

```c
/* Definizione */
struct Point {
    int x;
    int y;
};

/* Con typedef */
typedef struct {
    int x;
    int y;
} Point;

/* Uso */
Point p1 = {10, 20};
p1.x = 30;

/* Con puntatori */
Point *ptr = &p1;
ptr->x = 40;  // Equivale a (*ptr).x = 40;
```

## File I/O

```c
#include <stdio.h>

/* Apertura */
FILE *file = fopen("file.txt", "r");  // r, w, a, r+, w+, a+
if (file == NULL) {
    perror("Errore apertura file");
    return 1;
}

/* Lettura */
char buffer[256];
fgets(buffer, sizeof(buffer), file);
fscanf(file, "%d", &num);

/* Scrittura */
fprintf(file, "Hello %s\n", name);
fputs("Hello\n", file);

/* Chiusura */
fclose(file);

/* Binario */
fread(buffer, size, count, file);
fwrite(buffer, size, count, file);
```

## Preprocessore

```c
/* Include */
#include <stdio.h>   // System header
#include "myfile.h"  // Local header

/* Define */
#define PI 3.14159
#define MAX(a,b) ((a) > (b) ? (a) : (b))

/* Conditional */
#ifdef DEBUG
    printf("Debug mode\n");
#endif

#ifndef HEADER_H
#define HEADER_H
// codice header
#endif

/* Predefinite */
__FILE__    // Nome file corrente
__LINE__    // Numero linea corrente
__DATE__    // Data compilazione
__TIME__    // Ora compilazione
```

## Printf/Scanf Format

```c
/* Integer */
%d, %i      // int
%u          // unsigned int
%ld         // long
%lld        // long long
%x, %X      // hex

/* Float */
%f          // float/double
%lf         // double (solo scanf)
%e, %E      // scientific notation
%g, %G      // shortest representation

/* String/Char */
%s          // string
%c          // char

/* Pointer */
%p          // pointer address

/* Modifiers */
%5d         // width 5
%-5d        // left align
%05d        // pad with zeros
%.2f        // 2 decimal places
%*d         // width da variabile
```

## Debugging

```bash
# GDB
gdb ./programma
(gdb) break main      # Breakpoint
(gdb) run             # Esegui
(gdb) next            # Prossima linea
(gdb) step            # Step into
(gdb) print variable  # Stampa variabile
(gdb) backtrace       # Stack trace
(gdb) quit            # Esci

# Valgrind
valgrind --leak-check=full ./programma
```

## Librerie Standard Comuni

```c
#include <stdio.h>      // I/O standard
#include <stdlib.h>     // malloc, rand, exit
#include <string.h>     // Funzioni stringhe
#include <math.h>       // Funzioni matematiche
#include <time.h>       // Time functions
#include <stdbool.h>    // bool type
#include <stdint.h>     // Fixed-width integers
#include <assert.h>     // Assertions
#include <errno.h>      // Error numbers
#include <limits.h>     // Type limits
#include <ctype.h>      // Character handling
```

## Funzioni Matematiche (math.h)

```c
/* Compilare con -lm */

sqrt(x)         // Radice quadrata
pow(x, y)       // x^y
abs(x)          // Valore assoluto (int)
fabs(x)         // Valore assoluto (float)

sin(x), cos(x), tan(x)
asin(x), acos(x), atan(x)

log(x)          // Logaritmo naturale
log10(x)        // Logaritmo base 10
exp(x)          // e^x

ceil(x)         // Arrotonda su
floor(x)        // Arrotonda giù
round(x)        // Arrotonda a intero più vicino
```

## Errori Comuni da Evitare

```c
/* 1. Array out of bounds */
int arr[10];
arr[10] = 5;  // ERRORE: index 0-9 validi

/* 2. Dimenticare \0 nelle stringhe */
char str[5] = "Hello";  // ERRORE: serve 6 byte!

/* 3. Memory leak */
int *ptr = malloc(sizeof(int));
// ... uso ptr ...
// ERRORE: dimenticato free(ptr)

/* 4. Use after free */
free(ptr);
*ptr = 10;  // ERRORE: dangling pointer

/* 5. Double free */
free(ptr);
free(ptr);  // ERRORE: undefined behavior

/* 6. Comparazione float */
if (f == 0.3)  // ERRORE: usa fabs(f - 0.3) < EPSILON

/* 7. Off-by-one */
for (int i = 0; i <= 10; i++)  // Se array è size 10, è errore

/* 8. Integer division */
int x = 5 / 2;  // x = 2, non 2.5!
double y = 5.0 / 2.0;  // y = 2.5

/* 9. Scanf con stringhe */
char str[10];
scanf("%s", str);  // Pericoloso! Usa fgets

/* 10. Modificare string literal */
char *str = "Hello";
str[0] = 'h';  // ERRORE: undefined behavior
```

## Best Practices

1. ✅ Usa `const` per parametri che non modifichi
2. ✅ Controlla sempre valore di ritorno di `malloc`
3. ✅ Inizializza sempre le variabili
4. ✅ Setta puntatori a `NULL` dopo `free`
5. ✅ Usa `sizeof` invece di numeri magici
6. ✅ Compila con `-Wall -Wextra`
7. ✅ Verifica memory leaks con valgrind
8. ✅ Commenta codice complesso
9. ✅ Usa nomi variabili descrittivi
10. ✅ Mantieni funzioni piccole e focalizzate

---

Per dettagli completi, consulta i moduli del corso!
