# Modulo 03 - Livello Intermedio

## Puntatori

### Introduzione ai Puntatori

I puntatori sono uno degli aspetti più potenti e più pericolosi del linguaggio C. Un puntatore è una variabile che contiene l'indirizzo di memoria di un'altra variabile.

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

### Aritmetica dei Puntatori

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

### malloc, calloc, realloc, free

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
