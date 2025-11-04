# Modulo 02 - Fondamenti del Linguaggio C

## Struttura di un Programma C

### Anatomia Basilare

```c
/* 1. Direttive del preprocessore */
#include <stdio.h>
#include <stdlib.h>

/* 2. Definizioni di costanti */
#define PI 3.14159
#define MAX_SIZE 100

/* 3. Dichiarazioni di tipo */
typedef struct {
    int x;
    int y;
} Point;

/* 4. Dichiarazioni di funzioni (prototipi) */
int somma(int a, int b);
void stampa_messaggio(void);

/* 5. Funzione main - punto di ingresso */
int main(void)
{
    printf("Programma avviato\n");
    return 0;  // Ritorna al sistema operativo
}

/* 6. Definizioni di funzioni */
int somma(int a, int b)
{
    return a + b;
}

void stampa_messaggio(void)
{
    printf("Messaggio di test\n");
}
```

## Tipi di Dati

### Tipi Interi

```c
#include <stdio.h>
#include <stdint.h>
#include <limits.h>

int main(void)
{
    /* Tipi base */
    char c = 'A';                    // 1 byte, -128 a 127
    unsigned char uc = 255;          // 1 byte, 0 a 255
    short s = -32768;                // 2 byte
    unsigned short us = 65535;       // 2 byte
    int i = -2147483648;             // 4 byte (tipicamente)
    unsigned int ui = 4294967295U;   // 4 byte
    long l = -2147483648L;           // 4 o 8 byte
    long long ll = 9223372036854775807LL;  // 8 byte
    
    /* Tipi a dimensione fissa (C99+) - RACCOMANDATI */
    int8_t   i8 = -128;
    uint8_t  u8 = 255;
    int16_t  i16 = -32768;
    uint16_t u16 = 65535;
    int32_t  i32 = -2147483648;
    uint32_t u32 = 4294967295U;
    int64_t  i64 = -9223372036854775807LL;
    uint64_t u64 = 18446744073709551615ULL;
    
    /* Stampare i limiti */
    printf("char: %d a %d\n", CHAR_MIN, CHAR_MAX);
    printf("int: %d a %d\n", INT_MIN, INT_MAX);
    printf("long long: %lld a %lld\n", LLONG_MIN, LLONG_MAX);
    
    return 0;
}
```

### Tipi Floating Point

```c
#include <stdio.h>
#include <float.h>
#include <math.h>

int main(void)
{
    float f = 3.14159f;           // 4 byte, ~7 cifre decimali
    double d = 3.14159265358979;  // 8 byte, ~15 cifre decimali
    long double ld = 3.14159265358979323846L;  // 10-16 byte
    
    /* Notazione scientifica */
    double grande = 1.5e10;   // 1.5 × 10^10
    double piccolo = 1.5e-10; // 1.5 × 10^-10
    
    /* Limiti */
    printf("float range: %e to %e\n", FLT_MIN, FLT_MAX);
    printf("double range: %e to %e\n", DBL_MIN, DBL_MAX);
    
    /* ATTENZIONE: confronti floating point */
    double a = 0.1 + 0.2;
    double b = 0.3;
    if (fabs(a - b) < 1e-9) {  // Usa epsilon per confronto!
        printf("Uguali (con tolleranza)\n");
    }
    
    return 0;
}
```

### Tipo Boolean (C99+)

```c
#include <stdio.h>
#include <stdbool.h>

int main(void)
{
    bool flag = true;
    bool is_valid = false;
    
    if (flag && !is_valid) {
        printf("flag è true, is_valid è false\n");
    }
    
    /* Senza stdbool.h (old school) */
    int old_style_bool = 1;  // 0 = false, non-zero = true
    
    return 0;
}
```

## Operatori

### Operatori Aritmetici

```c
#include <stdio.h>

int main(void)
{
    int a = 10, b = 3;
    
    printf("Addizione: %d + %d = %d\n", a, b, a + b);      // 13
    printf("Sottrazione: %d - %d = %d\n", a, b, a - b);    // 7
    printf("Moltiplicazione: %d * %d = %d\n", a, b, a * b);// 30
    printf("Divisione intera: %d / %d = %d\n", a, b, a / b);// 3 (NON 3.33!)
    printf("Modulo: %d %% %d = %d\n", a, b, a % b);        // 1
    
    /* Divisione floating point */
    double x = 10.0, y = 3.0;
    printf("Divisione float: %.2f / %.2f = %.2f\n", x, y, x / y);  // 3.33
    
    /* Cast per ottenere divisione float */
    double result = (double)a / b;  // 3.33
    printf("Cast: %d / %d = %.2f\n", a, b, result);
    
    return 0;
}
```

### Operatori di Incremento e Decremento

```c
#include <stdio.h>

int main(void)
{
    int x = 5;
    
    printf("x = %d\n", x);       // 5
    printf("x++ = %d\n", x++);   // 5 (post-incremento: usa poi incrementa)
    printf("x = %d\n", x);       // 6
    printf("++x = %d\n", ++x);   // 7 (pre-incremento: incrementa poi usa)
    printf("x = %d\n", x);       // 7
    
    /* Equivalenti */
    int y = 10;
    y = y + 1;  // Equivalente a y++
    y += 1;     // Equivalente a y++
    
    return 0;
}
```

### Operatori Relazionali e Logici

```c
#include <stdio.h>
#include <stdbool.h>

int main(void)
{
    int a = 5, b = 10;
    
    /* Operatori relazionali */
    printf("%d == %d: %d\n", a, b, a == b);  // 0 (false)
    printf("%d != %d: %d\n", a, b, a != b);  // 1 (true)
    printf("%d < %d: %d\n", a, b, a < b);    // 1
    printf("%d > %d: %d\n", a, b, a > b);    // 0
    printf("%d <= %d: %d\n", a, b, a <= b);  // 1
    printf("%d >= %d: %d\n", a, b, a >= b);  // 0
    
    /* Operatori logici */
    bool x = true, y = false;
    printf("AND: %d && %d = %d\n", x, y, x && y);  // 0
    printf("OR: %d || %d = %d\n", x, y, x || y);   // 1
    printf("NOT: !%d = %d\n", x, !x);              // 0
    
    /* Short-circuit evaluation */
    int n = 0;
    if (n != 0 && 10/n > 1) {  // Secondo check NON eseguito se n==0
        printf("Questo non viene stampato\n");
    }
    
    return 0;
}
```

### Operatori Bitwise

```c
#include <stdio.h>

void print_binary(unsigned int n)
{
    for (int i = 31; i >= 0; i--) {
        printf("%d", (n >> i) & 1);
        if (i % 4 == 0) printf(" ");
    }
    printf("\n");
}

int main(void)
{
    unsigned int a = 60;    // 0011 1100
    unsigned int b = 13;    // 0000 1101
    
    printf("a = %u: ", a); print_binary(a);
    printf("b = %u: ", b); print_binary(b);
    printf("\n");
    
    printf("a & b = %u: ", a & b);  print_binary(a & b);   // AND
    printf("a | b = %u: ", a | b);  print_binary(a | b);   // OR
    printf("a ^ b = %u: ", a ^ b);  print_binary(a ^ b);   // XOR
    printf("~a = %u: ", ~a);        print_binary(~a);      // NOT
    printf("a << 2 = %u: ", a << 2); print_binary(a << 2); // Shift left
    printf("a >> 2 = %u: ", a >> 2); print_binary(a >> 2); // Shift right
    
    return 0;
}
```

## Strutture di Controllo

### If-Else

```c
#include <stdio.h>

int main(void)
{
    int voto = 85;
    
    /* If semplice */
    if (voto >= 90) {
        printf("Eccellente!\n");
    }
    
    /* If-else */
    if (voto >= 60) {
        printf("Promosso\n");
    } else {
        printf("Bocciato\n");
    }
    
    /* If-else if-else */
    if (voto >= 90) {
        printf("Voto: A\n");
    } else if (voto >= 80) {
        printf("Voto: B\n");
    } else if (voto >= 70) {
        printf("Voto: C\n");
    } else if (voto >= 60) {
        printf("Voto: D\n");
    } else {
        printf("Voto: F\n");
    }
    
    /* Operatore ternario */
    char *stato = (voto >= 60) ? "Promosso" : "Bocciato";
    printf("Stato: %s\n", stato);
    
    return 0;
}
```

### Switch

```c
#include <stdio.h>

int main(void)
{
    int giorno = 3;
    
    switch (giorno) {
        case 1:
            printf("Lunedì\n");
            break;
        case 2:
            printf("Martedì\n");
            break;
        case 3:
            printf("Mercoledì\n");
            break;
        case 4:
            printf("Giovedì\n");
            break;
        case 5:
            printf("Venerdì\n");
            break;
        case 6:
        case 7:
            printf("Weekend!\n");
            break;
        default:
            printf("Giorno non valido\n");
    }
    
    /* ATTENZIONE: fall-through senza break */
    int mese = 2;
    int giorni;
    switch (mese) {
        case 1: case 3: case 5: case 7: case 8: case 10: case 12:
            giorni = 31;
            break;
        case 4: case 6: case 9: case 11:
            giorni = 30;
            break;
        case 2:
            giorni = 28;  // Semplificato
            break;
        default:
            giorni = 0;
    }
    printf("Giorni nel mese %d: %d\n", mese, giorni);
    
    return 0;
}
```

### Loop - While

```c
#include <stdio.h>

int main(void)
{
    /* While loop */
    int i = 1;
    printf("Conteggio con while:\n");
    while (i <= 5) {
        printf("%d ", i);
        i++;
    }
    printf("\n");
    
    /* Do-while loop */
    int j = 1;
    printf("Conteggio con do-while:\n");
    do {
        printf("%d ", j);
        j++;
    } while (j <= 5);
    printf("\n");
    
    /* Differenza: do-while esegue almeno una volta */
    int k = 10;
    do {
        printf("Eseguito anche se k=%d > 5\n", k);
    } while (k < 5);
    
    return 0;
}
```

### Loop - For

```c
#include <stdio.h>

int main(void)
{
    /* For classico */
    printf("For classico:\n");
    for (int i = 0; i < 5; i++) {
        printf("%d ", i);
    }
    printf("\n");
    
    /* For con step diverso */
    printf("For con step 2:\n");
    for (int i = 0; i < 10; i += 2) {
        printf("%d ", i);
    }
    printf("\n");
    
    /* For decrescente */
    printf("For decrescente:\n");
    for (int i = 5; i > 0; i--) {
        printf("%d ", i);
    }
    printf("\n");
    
    /* For con condizioni complesse */
    printf("For con condizioni multiple:\n");
    for (int i = 0, j = 10; i < j; i++, j--) {
        printf("i=%d, j=%d\n", i, j);
    }
    
    /* Infinite loop */
    // for (;;) {  // Equivalente a while(1)
    //     // Codice infinito
    // }
    
    return 0;
}
```

### Break e Continue

```c
#include <stdio.h>

int main(void)
{
    /* Break: esce dal loop */
    printf("Esempio break:\n");
    for (int i = 0; i < 10; i++) {
        if (i == 5) {
            break;  // Esce quando i == 5
        }
        printf("%d ", i);
    }
    printf("\n");
    
    /* Continue: salta all'iterazione successiva */
    printf("Esempio continue:\n");
    for (int i = 0; i < 10; i++) {
        if (i % 2 == 0) {
            continue;  // Salta numeri pari
        }
        printf("%d ", i);
    }
    printf("\n");
    
    /* Trova primo numero divisibile per 7 e 13 */
    printf("Primo numero divisibile per 7 e 13: ");
    for (int i = 1; i < 1000; i++) {
        if (i % 7 == 0 && i % 13 == 0) {
            printf("%d\n", i);
            break;
        }
    }
    
    return 0;
}
```

## Funzioni

### Definizione e Chiamata

```c
#include <stdio.h>

/* Prototipo (dichiarazione) */
int somma(int a, int b);
void stampa_tabellina(int n);
double calcola_media(int arr[], int size);

int main(void)
{
    int risultato = somma(5, 3);
    printf("5 + 3 = %d\n", risultato);
    
    stampa_tabellina(7);
    
    int numeri[] = {10, 20, 30, 40, 50};
    double media = calcola_media(numeri, 5);
    printf("Media: %.2f\n", media);
    
    return 0;
}

/* Definizione */
int somma(int a, int b)
{
    return a + b;
}

void stampa_tabellina(int n)
{
    printf("Tabellina del %d:\n", n);
    for (int i = 1; i <= 10; i++) {
        printf("%d x %d = %d\n", n, i, n * i);
    }
}

double calcola_media(int arr[], int size)
{
    int somma = 0;
    for (int i = 0; i < size; i++) {
        somma += arr[i];
    }
    return (double)somma / size;
}
```

### Passaggio per Valore vs Riferimento

```c
#include <stdio.h>

/* Passaggio per valore - NON modifica l'originale */
void incrementa_valore(int x)
{
    x++;
    printf("Dentro funzione: %d\n", x);
}

/* Passaggio per riferimento - MODIFICA l'originale */
void incrementa_riferimento(int *x)
{
    (*x)++;
    printf("Dentro funzione: %d\n", *x);
}

/* Scambio di valori */
void swap(int *a, int *b)
{
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main(void)
{
    int num = 10;
    
    printf("Prima: %d\n", num);
    incrementa_valore(num);
    printf("Dopo (per valore): %d\n\n", num);  // Ancora 10!
    
    printf("Prima: %d\n", num);
    incrementa_riferimento(&num);
    printf("Dopo (per riferimento): %d\n\n", num);  // Ora è 11!
    
    int x = 5, y = 10;
    printf("Prima swap: x=%d, y=%d\n", x, y);
    swap(&x, &y);
    printf("Dopo swap: x=%d, y=%d\n", x, y);
    
    return 0;
}
```

### Funzioni Ricorsive

```c
#include <stdio.h>

/* Fattoriale */
unsigned long long fattoriale(int n)
{
    if (n <= 1) {
        return 1;  // Caso base
    }
    return n * fattoriale(n - 1);  // Chiamata ricorsiva
}

/* Fibonacci */
int fibonacci(int n)
{
    if (n <= 1) {
        return n;
    }
    return fibonacci(n - 1) + fibonacci(n - 2);
}

/* Ricerca binaria ricorsiva */
int ricerca_binaria(int arr[], int left, int right, int target)
{
    if (right >= left) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            return mid;
        }
        
        if (arr[mid] > target) {
            return ricerca_binaria(arr, left, mid - 1, target);
        }
        
        return ricerca_binaria(arr, mid + 1, right, target);
    }
    
    return -1;  // Non trovato
}

int main(void)
{
    printf("Fattoriale di 10: %llu\n", fattoriale(10));
    
    printf("Primi 10 numeri di Fibonacci:\n");
    for (int i = 0; i < 10; i++) {
        printf("%d ", fibonacci(i));
    }
    printf("\n");
    
    int arr[] = {2, 3, 4, 10, 40, 50, 60, 70};
    int size = sizeof(arr) / sizeof(arr[0]);
    int target = 10;
    int result = ricerca_binaria(arr, 0, size - 1, target);
    printf("Elemento %d trovato all'indice: %d\n", target, result);
    
    return 0;
}
```

## Array

### Array Monodimensionali

```c
#include <stdio.h>

int main(void)
{
    /* Dichiarazione e inizializzazione */
    int numeri[5];  // Array non inizializzato
    int valori[5] = {10, 20, 30, 40, 50};  // Inizializzato
    int parziale[5] = {1, 2};  // Resto inizializzato a 0
    int automatico[] = {1, 2, 3, 4, 5};  // Dimensione automatica
    
    /* Accesso agli elementi */
    printf("Primo elemento: %d\n", valori[0]);
    printf("Ultimo elemento: %d\n", valori[4]);
    
    /* Iterazione */
    printf("Tutti gli elementi: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", valori[i]);
    }
    printf("\n");
    
    /* Dimensione dell'array */
    int size = sizeof(valori) / sizeof(valori[0]);
    printf("Dimensione array: %d\n", size);
    
    /* Modificare elementi */
    valori[2] = 100;
    printf("Elemento modificato: %d\n", valori[2]);
    
    /* ATTENZIONE: nessun controllo sui bounds! */
    // valori[10] = 999;  // ERRORE: accesso fuori dai limiti!
    
    return 0;
}
```

### Array Multidimensionali

```c
#include <stdio.h>

int main(void)
{
    /* Matrice 3x3 */
    int matrice[3][3] = {
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9}
    };
    
    /* Accesso agli elementi */
    printf("Elemento [1][2]: %d\n", matrice[1][2]);  // 6
    
    /* Stampa matrice */
    printf("Matrice:\n");
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d ", matrice[i][j]);
        }
        printf("\n");
    }
    
    /* Array 3D */
    int cubo[2][3][4] = {
        {
            {1, 2, 3, 4},
            {5, 6, 7, 8},
            {9, 10, 11, 12}
        },
        {
            {13, 14, 15, 16},
            {17, 18, 19, 20},
            {21, 22, 23, 24}
        }
    };
    
    printf("Elemento cubo[1][2][3]: %d\n", cubo[1][2][3]);  // 24
    
    return 0;
}
```

## Stringhe

### Stringhe come Array di Caratteri

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    /* Dichiarazione */
    char str1[20] = "Hello";           // Con terminatore \0
    char str2[] = "World";             // Dimensione automatica
    char str3[20] = {'C', 'i', 'a', 'o', '\0'};  // Manuale
    
    /* ATTENZIONE: serve sempre spazio per \0 */
    char troppo_corta[5] = "Hello";    // ERRORE: manca spazio per \0
    
    /* Stampare stringhe */
    printf("%s\n", str1);
    printf("Carattere per carattere: ");
    for (int i = 0; str1[i] != '\0'; i++) {
        printf("%c", str1[i]);
    }
    printf("\n");
    
    /* Funzioni di stringa (string.h) */
    printf("Lunghezza di '%s': %lu\n", str1, strlen(str1));
    
    /* Concatenazione */
    char dest[50] = "Hello ";
    strcat(dest, "World!");
    printf("Concatenato: %s\n", dest);
    
    /* Copia */
    char copia[50];
    strcpy(copia, dest);
    printf("Copiato: %s\n", copia);
    
    /* Confronto */
    if (strcmp(str1, "Hello") == 0) {
        printf("Stringhe uguali!\n");
    }
    
    /* Input */
    char input[100];
    printf("Inserisci una stringa: ");
    fgets(input, sizeof(input), stdin);
    input[strcspn(input, "\n")] = '\0';  // Rimuove newline
    printf("Hai inserito: %s\n", input);
    
    return 0;
}
```

### Funzioni Comuni per Stringhe

```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>

/* Lunghezza stringa personalizzata */
int mia_strlen(const char *str)
{
    int len = 0;
    while (str[len] != '\0') {
        len++;
    }
    return len;
}

/* Copia stringa personalizzata */
void mia_strcpy(char *dest, const char *src)
{
    int i = 0;
    while (src[i] != '\0') {
        dest[i] = src[i];
        i++;
    }
    dest[i] = '\0';
}

/* Conversione a maiuscolo */
void to_uppercase(char *str)
{
    for (int i = 0; str[i] != '\0'; i++) {
        str[i] = toupper(str[i]);
    }
}

int main(void)
{
    char str[] = "Hello World";
    
    printf("Lunghezza: %d\n", mia_strlen(str));
    
    char dest[50];
    mia_strcpy(dest, str);
    printf("Copia: %s\n", dest);
    
    to_uppercase(dest);
    printf("Maiuscolo: %s\n", dest);
    
    return 0;
}
```

## Esercizi di Riepilogo

1. Scrivi un programma che calcola il fattoriale usando un loop
2. Crea una funzione che inverte un array
3. Implementa una funzione di ricerca lineare
4. Scrivi una funzione che conta le vocali in una stringa
5. Crea un programma che stampa il triangolo di Pascal

---

[← Introduzione](../01_Introduzione/README.md) | [Torna al Sommario](../README.md) | [Prossimo: Intermedio →](../03_Intermedio/README.md)
