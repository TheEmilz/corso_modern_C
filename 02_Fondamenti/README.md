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

I tipi interi in C rappresentano numeri senza parte decimale. La dimensione e il range dipendono dal tipo.

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

#### Spiegazione Dettagliata dei Tipi Interi

**`char c = 'A';`**
- `char`: Il tipo più piccolo, 1 byte (8 bit)
- Può contenere valori da -128 a 127 (se signed)
- Usato principalmente per caratteri ASCII
- `'A'`: Carattere letterale - internamente è il valore ASCII 65
- In memoria: `01000001` (rappresentazione binaria di 65)

**`unsigned char uc = 255;`**
- `unsigned`: Modificatore che rende il tipo sempre positivo
- Range: 0 a 255 (tutti i 256 valori possibili con 8 bit)
- Usa tutti i bit per il valore (nessun bit per il segno)
- Utile per: byte grezzi, dati binari, pixel

**`short s = -32768;`**
- `short`: Tipo intero a 2 byte (16 bit)
- Range: -32,768 a 32,767
- Utile quando serve risparmiare memoria rispetto a `int`
- Calcolo del range: -(2^15) a (2^15 - 1)

**`int i = -2147483648;`**
- `int`: Tipo intero standard, tipicamente 4 byte (32 bit)
- Range: -2,147,483,648 a 2,147,483,647
- Il tipo intero di default in C
- **Attenzione**: La dimensione può variare tra sistemi!
  - Su sistemi a 16 bit: potrebbe essere 2 byte
  - Su sistemi a 32/64 bit: tipicamente 4 byte

**`unsigned int ui = 4294967295U;`**
- Range: 0 a 4,294,967,295
- `U` o `u`: Suffisso che indica unsigned literal
- Doppio range positivo rispetto a `int` signed
- Utile per: contatori, dimensioni, bit mask

**`long l = -2147483648L;`**
- `long`: Intero "lungo", almeno 4 byte
- `L` o `l`: Suffisso per long literal
- Su Linux a 64 bit: tipicamente 8 byte
- Su Windows a 64 bit: tipicamente 4 byte
- **Problema di portabilità!**

**`long long ll = 9223372036854775807LL;`**
- `long long`: Sempre 8 byte (garantito da C99)
- Range: -(2^63) a (2^63 - 1)
- `LL` o `ll`: Suffisso per long long literal
- Utile per: grandi numeri, timestamp, calcoli astronomici

**Tipi a Dimensione Fissa (Raccomandati!):**

**`int8_t i8 = -128;`**
- **Esattamente** 8 bit signed (-128 a 127)
- Garantito da `<stdint.h>` (C99+)
- Portabile su tutti i sistemi

**`uint32_t u32 = 4294967295U;`**
- **Esattamente** 32 bit unsigned (0 a 4,294,967,295)
- `uint`: unsigned integer
- `32`: numero di bit
- `_t`: tipo

**Perché usare tipi a dimensione fissa?**
1. **Portabilità**: Stesso comportamento su tutti i sistemi
2. **Chiarezza**: Il nome dice esattamente la dimensione
3. **Networking**: Protocolli richiedono dimensioni precise
4. **File I/O**: Formato binario consistente
5. **Embedded**: Controllo preciso della memoria

**Costanti da `<limits.h>`:**
```c
CHAR_MIN, CHAR_MAX    // Limiti di char
SHRT_MIN, SHRT_MAX    // Limiti di short
INT_MIN, INT_MAX      // Limiti di int
LONG_MIN, LONG_MAX    // Limiti di long
LLONG_MIN, LLONG_MAX  // Limiti di long long
```

#### Esempio Pratico: Overflow degli Interi

```c
#include <stdio.h>
#include <stdint.h>

int main(void)
{
    /* Overflow con char signed */
    int8_t piccolo = 127;
    printf("Valore iniziale: %d\n", (int)piccolo);  // Cast a int per portabilità
    
    piccolo = piccolo + 1;  // Overflow!
    printf("Dopo +1: %d\n", (int)piccolo);  // -128 (wrap around!)
    
    /* Overflow con unsigned */
    uint8_t senza_segno = 255;
    printf("\nValore iniziale: %u\n", senza_segno);
    
    senza_segno = senza_segno + 1;  // Overflow!
    printf("Dopo +1: %u\n", senza_segno);  // 0 (wrap around!)
    
    /* Quando usare quale tipo */
    int32_t contatore = 0;           // Contatori generali
    uint32_t dimensione = 1024;      // Dimensioni, non possono essere negative
    int64_t timestamp = 1234567890;  // Grandi valori temporali
    uint8_t byte_dato = 0xFF;        // Dati grezzi
    
    return 0;
}
```

**Spiegazione dell'Overflow:**
- Gli interi hanno un range limitato
- Superare il massimo causa "wrap around" al minimo
- Per signed: 127 + 1 = -128
- Per unsigned: 255 + 1 = 0
- **Non è un errore per il compilatore!** È comportamento definito per unsigned, indefinito per signed
- In produzione: controlla sempre i range prima di operazioni critiche

**Compilazione ed esecuzione:**
```bash
gcc -Wall -Wextra tipi_interi.c -o tipi_interi
./tipi_interi
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

Gli operatori aritmetici permettono di eseguire calcoli matematici su numeri.

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

#### Spiegazione Dettagliata degli Operatori Aritmetici

**Addizione: `a + b`**
```c
int a = 10, b = 3;
int somma = a + b;  // somma = 13
```
- Operatore binario (richiede due operandi)
- Risultato ha il tipo del più grande tra i due operandi
- Può causare overflow se il risultato supera il range del tipo
- Esempio overflow: `INT_MAX + 1` → comportamento indefinito

**Sottrazione: `a - b`**
```c
int differenza = a - b;  // differenza = 7
```
- Sottrae il secondo operando dal primo
- Può dare risultati negativi anche con unsigned (con wrap around)
- Esempio: `(unsigned int)5 - 10` → valore molto grande!

**Moltiplicazione: `a * b`**
```c
int prodotto = a * b;  // prodotto = 30
```
- Moltiplica due valori
- Attenzione all'overflow! `INT_MAX * 2` causa problemi
- Per grandi moltiplicazioni, usa tipi più grandi (`long long`)

**Divisione Intera: `a / b` (con interi)**
```c
int divisione = 10 / 3;  // divisione = 3 (NON 3.333...)
```
- **CRITICO**: La divisione tra interi **tronca** il risultato (non arrotonda!)
- Elimina la parte decimale
- Esempi:
  - `7 / 2` → `3` (non 3.5)
  - `9 / 5` → `1` (non 1.8)
  - `-7 / 2` → `-3` (non -3.5)
- **Divisione per zero**: Comportamento indefinito (spesso crash)

**Modulo: `a % b`**
```c
int resto = 10 % 3;  // resto = 1
```
- Restituisce il resto della divisione intera
- Solo per tipi interi (non per float/double!)
- Utile per:
  - Verificare se un numero è pari: `n % 2 == 0`
  - Operazioni cicliche: `(indice + 1) % size`
  - Verificare divisibilità: `n % 5 == 0`
- Esempi:
  - `10 % 3` → `1` (10 = 3*3 + 1)
  - `15 % 4` → `3` (15 = 4*3 + 3)
  - `20 % 5` → `0` (20 = 5*4 + 0, divisibile)

**Divisione Floating Point:**
```c
double x = 10.0, y = 3.0;
double risultato = x / y;  // risultato = 3.333...
```
- Se **almeno uno** degli operandi è float/double, il risultato è float/double
- Mantiene la parte decimale
- Precisione limitata (circa 15 cifre decimali per `double`)

**Cast Esplicito:**
```c
int a = 10, b = 3;
double risultato = (double)a / b;  // risultato = 3.333...
```
- `(double)a`: Converte `a` in double
- Ora la divisione è tra double e int → risultato double
- **Ordine importante**:
  - `(double)(a / b)` → `(double)3` → `3.0` (SBAGLIATO!)
  - `(double)a / b` → `10.0 / 3` → `3.333...` (CORRETTO!)

#### Esempio Pratico: Calcolo della Media

```c
#include <stdio.h>

int main(void)
{
    int voti[5] = {85, 90, 78, 92, 88};
    int somma = 0;
    int numero_voti = 5;
    
    /* Calcola la somma */
    for (int i = 0; i < numero_voti; i++) {
        somma += voti[i];  // somma = somma + voti[i]
        printf("Voto %d: %d (somma parziale: %d)\n", i+1, voti[i], somma);
    }
    
    /* Media - SBAGLIATA (divisione intera!) */
    int media_sbagliata = somma / numero_voti;
    printf("\nMedia (sbagliata): %d\n", media_sbagliata);  // 86, non 86.6!
    
    /* Media - CORRETTA (con cast) */
    double media_corretta = (double)somma / numero_voti;
    printf("Media (corretta): %.2f\n", media_corretta);  // 86.60
    
    /* Alternativa: usa double da subito */
    double somma_double = 0.0;
    for (int i = 0; i < numero_voti; i++) {
        somma_double += voti[i];
    }
    double media_alt = somma_double / numero_voti;
    printf("Media (alternativa): %.2f\n", media_alt);  // 86.60
    
    return 0;
}
```

**Output del programma:**
```
Voto 1: 85 (somma parziale: 85)
Voto 2: 90 (somma parziale: 175)
Voto 3: 78 (somma parziale: 253)
Voto 4: 92 (somma parziale: 345)
Voto 5: 88 (somma parziale: 433)

Media (sbagliata): 86
Media (corretta): 86.60
Media (alternativa): 86.60
```

**Spiegazione passo-passo:**
1. **Inizializzazione**: `somma = 0`, array con 5 voti
2. **Loop di somma**: Accumula ogni voto in `somma`
   - Iterazione 1: `somma = 0 + 85 = 85`
   - Iterazione 2: `somma = 85 + 90 = 175`
   - ... continua fino a `somma = 433`
3. **Divisione sbagliata**: `433 / 5 = 86` (tronca .6)
4. **Divisione corretta**: `433.0 / 5 = 86.6` (mantiene decimali)

#### Operatori di Assegnamento Composti

```c
#include <stdio.h>

int main(void)
{
    int x = 10;
    
    x += 5;   // Equivalente a: x = x + 5;    (x = 15)
    x -= 3;   // Equivalente a: x = x - 3;    (x = 12)
    x *= 2;   // Equivalente a: x = x * 2;    (x = 24)
    x /= 4;   // Equivalente a: x = x / 4;    (x = 6)
    x %= 5;   // Equivalente a: x = x % 5;    (x = 1)
    
    printf("Valore finale: %d\n", x);
    
    /* Vantaggi degli operatori composti */
    int array[100];
    int indice = 0;
    
    // Più leggibile e meno ripetitivo
    indice += 10;  // invece di: indice = indice + 10;
    
    // Più efficiente (il compilatore può ottimizzare meglio)
    array[indice] += 5;  // invece di: array[indice] = array[indice] + 5;
    
    return 0;
}
```

**Quando usare operatori composti:**
- Codice più conciso e leggibile
- Meno errori (scrivi la variabile una sola volta)
- Potenziali ottimizzazioni del compilatore
- Convenzione standard in C

**Compilazione e test:**
```bash
gcc -Wall -Wextra operatori_aritmetici.c -o operatori
./operatori
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

Il loop `for` è perfetto quando sai in anticipo quante iterazioni fare.

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

#### Spiegazione Dettagliata del Loop For

**Sintassi del For:**
```c
for (inizializzazione; condizione; incremento) {
    // corpo del loop
}
```

**Flusso di esecuzione:**
1. **Inizializzazione**: Eseguita **una sola volta** all'inizio
2. **Condizione**: Controllata **prima** di ogni iterazione
3. **Corpo**: Eseguito se la condizione è vera
4. **Incremento**: Eseguito **dopo** ogni iterazione
5. Torna al passo 2

**Esempio classico analizzato:**
```c
for (int i = 0; i < 5; i++) {
    printf("%d ", i);
}
```

**Esecuzione passo-passo:**
```
Passo 1: int i = 0         (inizializzazione)
Passo 2: i < 5?            (0 < 5 = vero)
Passo 3: printf("%d ", i)  (stampa 0)
Passo 4: i++               (i diventa 1)
Passo 5: i < 5?            (1 < 5 = vero)
Passo 6: printf("%d ", i)  (stampa 1)
Passo 7: i++               (i diventa 2)
...continua...
Passo N: i < 5?            (5 < 5 = falso)
Passo N+1: Esce dal loop
```

**Output:** `0 1 2 3 4`

**Equivalente con while:**
```c
int i = 0;                 // Inizializzazione
while (i < 5) {            // Condizione
    printf("%d ", i);      // Corpo
    i++;                   // Incremento
}
```

**For con step diverso:**
```c
for (int i = 0; i < 10; i += 2) {
    printf("%d ", i);
}
```
- `i += 2`: Incrementa di 2 invece di 1
- Stampa solo numeri pari: `0 2 4 6 8`
- Utile per: saltare elementi, processare a blocchi

**For decrescente:**
```c
for (int i = 5; i > 0; i--) {
    printf("%d ", i);
}
```
- Parte da 5 e scende fino a 1
- `i--`: Decrementa invece di incrementare
- Output: `5 4 3 2 1`
- Utile per: countdown, processare array al contrario

**For con variabili multiple:**
```c
for (int i = 0, j = 10; i < j; i++, j--) {
    printf("i=%d, j=%d\n", i, j);
}
```
- **Inizializzazione**: `int i = 0, j = 10` (virgola separa dichiarazioni)
- **Condizione**: `i < j` (continua finché i è minore di j)
- **Incremento**: `i++, j--` (virgola separa espressioni)
- Output:
  ```
  i=0, j=10
  i=1, j=9
  i=2, j=8
  i=3, j=7
  i=4, j=6
  ```
- Si ferma quando i e j si incrociano

**Infinite loop:**
```c
for (;;) {
    // Codice infinito
    // Serve break per uscire
}
```
- Nessuna inizializzazione, condizione, incremento
- Condizione vuota = sempre vera
- Equivalente a `while (1)` o `while (true)`
- **Attenzione**: Assicurati di avere un modo per uscire!

#### Esempi Pratici Completi

**Esempio 1: Tabellina**
```c
#include <stdio.h>

int main(void)
{
    int numero = 7;
    
    printf("Tabellina del %d:\n", numero);
    printf("-------------------\n");
    
    for (int i = 1; i <= 10; i++) {
        int risultato = numero * i;
        printf("%d x %d = %d\n", numero, i, risultato);
    }
    
    return 0;
}
```

**Output:**
```
Tabellina del 7:
-------------------
7 x 1 = 7
7 x 2 = 14
7 x 3 = 21
...
7 x 10 = 70
```

**Spiegazione:**
- Loop da 1 a 10 (inclusi)
- Ogni iterazione: moltiplica numero per i
- Stampa l'equazione completa

**Esempio 2: Somma di Numeri**
```c
#include <stdio.h>

int main(void)
{
    int somma = 0;
    int n = 100;
    
    /* Somma dei primi 100 numeri */
    for (int i = 1; i <= n; i++) {
        somma += i;
        
        /* Mostra progresso ogni 20 numeri */
        if (i % 20 == 0) {
            printf("Somma fino a %d: %d\n", i, somma);
        }
    }
    
    printf("\nSomma finale (1 + 2 + ... + %d) = %d\n", n, somma);
    
    /* Formula di Gauss per verifica */
    int gauss = n * (n + 1) / 2;
    printf("Verifica con formula di Gauss: %d\n", gauss);
    
    return 0;
}
```

**Output:**
```
Somma fino a 20: 210
Somma fino a 40: 820
Somma fino a 60: 1830
Somma fino a 80: 3240
Somma fino a 100: 5050

Somma finale (1 + 2 + ... + 100) = 5050
Verifica con formula di Gauss: 5050
```

**Spiegazione:**
- `somma += i`: Accumula ogni numero nella somma
- `if (i % 20 == 0)`: Stampa ogni 20 iterazioni per vedere il progresso
- Formula di Gauss: n * (n + 1) / 2 per la somma da 1 a n
- Verifica che il risultato sia corretto

**Esempio 3: Array e For**
```c
#include <stdio.h>

int main(void)
{
    int numeri[] = {10, 20, 30, 40, 50};
    int size = sizeof(numeri) / sizeof(numeri[0]);
    
    /* Stampare array */
    printf("Array originale: ");
    for (int i = 0; i < size; i++) {
        printf("%d ", numeri[i]);
    }
    printf("\n");
    
    /* Raddoppiare ogni elemento */
    for (int i = 0; i < size; i++) {
        numeri[i] = numeri[i] * 2;
    }
    
    /* Stampare array modificato */
    printf("Array raddoppiato: ");
    for (int i = 0; i < size; i++) {
        printf("%d ", numeri[i]);
    }
    printf("\n");
    
    /* Trovare il massimo */
    int massimo = numeri[0];
    for (int i = 1; i < size; i++) {
        if (numeri[i] > massimo) {
            massimo = numeri[i];
        }
    }
    printf("Valore massimo: %d\n", massimo);
    
    return 0;
}
```

**Output:**
```
Array originale: 10 20 30 40 50
Array raddoppiato: 20 40 60 80 100
Valore massimo: 100
```

**Spiegazione:**
- `sizeof(numeri) / sizeof(numeri[0])`: Calcola il numero di elementi
- Primo loop: Legge e stampa ogni elemento
- Secondo loop: Modifica ogni elemento in place
- Terzo loop: Trova il massimo confrontando elemento per elemento

**Esempio 4: Loop Annidati (Matrice)**
```c
#include <stdio.h>

int main(void)
{
    int righe = 3;
    int colonne = 4;
    
    /* Matrice con loop annidati */
    printf("Matrice %dx%d:\n", righe, colonne);
    
    for (int i = 0; i < righe; i++) {           // Loop esterno: righe
        printf("Riga %d: ", i);
        
        for (int j = 0; j < colonne; j++) {     // Loop interno: colonne
            int valore = i * colonne + j;
            printf("%2d ", valore);
        }
        
        printf("\n");
    }
    
    return 0;
}
```

**Output:**
```
Matrice 3x4:
Riga 0:  0  1  2  3
Riga 1:  4  5  6  7
Riga 2:  8  9 10 11
```

**Spiegazione dell'annidamento:**
```
Iterazione 1 (i=0):
  j=0: stampa 0
  j=1: stampa 1
  j=2: stampa 2
  j=3: stampa 3
  newline

Iterazione 2 (i=1):
  j=0: stampa 4
  j=1: stampa 5
  j=2: stampa 6
  j=3: stampa 7
  newline

Iterazione 3 (i=2):
  j=0: stampa 8
  j=1: stampa 9
  j=2: stampa 10
  j=3: stampa 11
  newline
```

**Regola dei loop annidati:**
- Il loop **interno** completa tutte le sue iterazioni
- Per **ogni singola** iterazione del loop esterno
- Complessità temporale: O(righe × colonne)

**Compilazione e test:**
```bash
gcc -Wall -Wextra loop_for.c -o loop_for
./loop_for
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

## Diagrammi di Memoria - Concetti Visivi

### Memoria degli Array

```c
int numeri[5] = {10, 20, 30, 40, 50};
```

**Rappresentazione in memoria:**
```
Indirizzo    Contenuto    Variabile
0x1000   →   10          numeri[0]
0x1004   →   20          numeri[1]
0x1008   →   30          numeri[2]
0x100C   →   40          numeri[3]
0x1010   →   50          numeri[4]
```

**Spiegazione:**
- Ogni `int` occupa 4 byte (su sistemi a 32/64 bit)
- Gli elementi sono contigui in memoria
- L'indirizzo aumenta di 4 byte per ogni elemento
- `numeri` è un puntatore all'inizio (0x1000)
- `numeri[i]` equivale a `*(numeri + i)`

### Stack e Variabili Locali

```c
void funzione(int x) {
    int y = 20;
    int z = x + y;
}

int main(void) {
    int a = 10;
    funzione(a);
    return 0;
}
```

**Stack durante l'esecuzione:**
```
        Stack
    ┌───────────┐
    │ z = 30    │  ← funzione() - variabili locali
    │ y = 20    │
    │ x = 10    │  ← parametro copiato
    ├───────────┤
    │ a = 10    │  ← main() - variabile locale
    ├───────────┤
    │ ...       │
    └───────────┘
```

**Cosa succede:**
1. `main` crea `a` nello stack
2. Chiama `funzione(a)`: copia il valore di `a` in `x`
3. `funzione` crea `y` e `z` nello stack
4. Al return: `z`, `y`, `x` vengono rimossi dallo stack
5. Torna a `main`: `a` ancora presente

## Esempi Pratici Completi Annotati

### Esempio 1: Gestione Voti Studenti

```c
#include <stdio.h>
#include <string.h>

/* Struttura per rappresentare uno studente */
typedef struct {
    char nome[50];
    int voti[5];
    double media;
} Studente;

/* Calcola la media dei voti */
double calcola_media(int voti[], int n)
{
    int somma = 0;
    
    /* Somma tutti i voti */
    for (int i = 0; i < n; i++) {
        somma += voti[i];
    }
    
    /* Ritorna la media (con cast per evitare divisione intera) */
    return (double)somma / n;
}

/* Stampa i dati di uno studente */
void stampa_studente(Studente s)
{
    printf("\nStudente: %s\n", s.nome);
    printf("Voti: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", s.voti[i]);
    }
    printf("\nMedia: %.2f\n", s.media);
}

int main(void)
{
    /* Crea uno studente */
    Studente studente;
    
    /* Inizializza il nome */
    strcpy(studente.nome, "Mario Rossi");
    
    /* Inizializza i voti */
    studente.voti[0] = 85;
    studente.voti[1] = 90;
    studente.voti[2] = 78;
    studente.voti[3] = 92;
    studente.voti[4] = 88;
    
    /* Calcola la media */
    studente.media = calcola_media(studente.voti, 5);
    
    /* Stampa i dati */
    stampa_studente(studente);
    
    /* Verifica se promosso */
    if (studente.media >= 60) {
        printf("Risultato: PROMOSSO\n");
    } else {
        printf("Risultato: BOCCIATO\n");
    }
    
    return 0;
}
```

**Spiegazione passo-passo:**

1. **Definizione struct** (linea 4-8):
   - Definisce un nuovo tipo `Studente`
   - Contiene: nome (array di char), voti (array di int), media (double)
   - `typedef`: Permette di scrivere `Studente s` invece di `struct Studente s`

2. **Funzione `calcola_media`** (linea 11-22):
   - Parametri: array di voti e numero di voti
   - Accumula la somma in un loop
   - Usa cast `(double)` per ottenere media con decimali
   - Ritorna il risultato

3. **Funzione `stampa_studente`** (linea 25-34):
   - Riceve uno studente per valore (copia)
   - Stampa nome usando `%s`
   - Loop per stampare tutti i voti
   - Stampa media con 2 decimali (`%.2f`)

4. **Main** (linea 36-61):
   - Crea una variabile di tipo `Studente`
   - `strcpy`: Copia la stringa nel campo nome
   - Inizializza ogni voto singolarmente
   - Chiama `calcola_media` e salva il risultato
   - Chiama `stampa_studente` per visualizzare
   - Verifica con `if` se la media è sufficiente

**Output:**
```
Studente: Mario Rossi
Voti: 85 90 78 92 88
Media: 86.60
Risultato: PROMOSSO
```

**Compilazione:**
```bash
gcc -Wall -Wextra -std=c11 gestione_voti.c -o gestione_voti
./gestione_voti
```

### Esempio 2: Mini Calcolatrice con Menu

```c
#include <stdio.h>
#include <stdlib.h>

/* Funzioni per le operazioni */
double somma(double a, double b) { return a + b; }
double sottrazione(double a, double b) { return a - b; }
double moltiplicazione(double a, double b) { return a * b; }
double divisione(double a, double b) {
    if (b == 0) {
        printf("ERRORE: Divisione per zero!\n");
        return 0;
    }
    return a / b;
}

/* Stampa il menu */
void stampa_menu(void)
{
    printf("\n=== CALCOLATRICE ===\n");
    printf("1. Addizione\n");
    printf("2. Sottrazione\n");
    printf("3. Moltiplicazione\n");
    printf("4. Divisione\n");
    printf("5. Esci\n");
    printf("Scelta: ");
}

/* Legge un numero double dall'utente */
double leggi_numero(const char *messaggio)
{
    double numero;
    printf("%s", messaggio);
    scanf("%lf", &numero);
    return numero;
}

int main(void)
{
    int scelta;
    double num1, num2, risultato;
    
    /* Loop principale del programma */
    while (1) {
        stampa_menu();
        scanf("%d", &scelta);
        
        /* Esci se scelta == 5 */
        if (scelta == 5) {
            printf("Arrivederci!\n");
            break;
        }
        
        /* Verifica scelta valida */
        if (scelta < 1 || scelta > 5) {
            printf("Scelta non valida!\n");
            continue;
        }
        
        /* Leggi i due numeri */
        num1 = leggi_numero("Primo numero: ");
        num2 = leggi_numero("Secondo numero: ");
        
        /* Esegui l'operazione scelta */
        switch (scelta) {
            case 1:
                risultato = somma(num1, num2);
                printf("%.2f + %.2f = %.2f\n", num1, num2, risultato);
                break;
            case 2:
                risultato = sottrazione(num1, num2);
                printf("%.2f - %.2f = %.2f\n", num1, num2, risultato);
                break;
            case 3:
                risultato = moltiplicazione(num1, num2);
                printf("%.2f * %.2f = %.2f\n", num1, num2, risultato);
                break;
            case 4:
                risultato = divisione(num1, num2);
                if (num2 != 0) {  /* Stampa solo se divisione valida */
                    printf("%.2f / %.2f = %.2f\n", num1, num2, risultato);
                }
                break;
        }
    }
    
    return 0;
}
```

**Spiegazione architettura:**

1. **Funzioni operazione** (linea 4-14):
   - Ogni operazione è una funzione separata
   - `divisione`: Controlla divisione per zero prima di eseguire
   - Ritorna il risultato dell'operazione

2. **Funzione `stampa_menu`** (linea 17-25):
   - Stampa le opzioni disponibili
   - Separata per evitare ripetizioni
   - `void`: Non ha parametri né ritorna valori

3. **Funzione `leggi_numero`** (linea 28-34):
   - Stampa un messaggio personalizzato
   - Legge un `double` dall'utente
   - Ritorna il numero letto
   - Riutilizzabile per entrambi i numeri

4. **Loop principale** (linea 42-85):
   - `while (1)`: Loop infinito
   - Stampa menu e legge scelta
   - `break`: Esce dal loop se scelta == 5
   - `continue`: Salta al prossimo ciclo se scelta non valida
   - `switch`: Esegue l'operazione corrispondente alla scelta
   - Ogni `case` chiama la funzione appropriata e stampa il risultato

**Flusso di esecuzione:**
```
1. Entra nel loop infinito
2. Stampa menu
3. Legge scelta utente
4. Se 5 → esce
5. Se non valida → torna a 2
6. Legge num1 e num2
7. Esegue operazione con switch
8. Stampa risultato
9. Torna a 2
```

**Esempio di sessione:**
```
=== CALCOLATRICE ===
1. Addizione
2. Sottrazione
3. Moltiplicazione
4. Divisione
5. Esci
Scelta: 1
Primo numero: 15.5
Secondo numero: 7.3
15.50 + 7.30 = 22.80

=== CALCOLATRICE ===
1. Addizione
2. Sottrazione
3. Moltiplicazione
4. Divisione
5. Esci
Scelta: 4
Primo numero: 10
Secondo numero: 0
ERRORE: Divisione per zero!

=== CALCOLATRICE ===
1. Addizione
2. Sottrazione
3. Moltiplicazione
4. Divisione
5. Esci
Scelta: 5
Arrivederci!
```

## Esercizi di Riepilogo con Soluzioni

### Esercizio 1: Fattoriale Iterativo

**Problema:** Scrivi un programma che calcola il fattoriale usando un loop.

**Soluzione:**
```c
#include <stdio.h>

unsigned long long fattoriale(int n)
{
    /* Caso base: 0! = 1 e 1! = 1 */
    if (n <= 1) {
        return 1;
    }
    
    unsigned long long risultato = 1;
    
    /* Moltiplica tutti i numeri da 2 a n */
    for (int i = 2; i <= n; i++) {
        risultato *= i;
    }
    
    return risultato;
}

int main(void)
{
    int n;
    
    printf("Inserisci un numero: ");
    scanf("%d", &n);
    
    if (n < 0) {
        printf("Errore: il fattoriale di numeri negativi non esiste\n");
        return 1;
    }
    
    unsigned long long result = fattoriale(n);
    printf("%d! = %llu\n", n, result);
    
    return 0;
}
```

**Compilazione e test:**
```bash
gcc -Wall -Wextra fattoriale.c -o fattoriale
./fattoriale
# Input: 5
# Output: 5! = 120
```

### Esercizio 2: Inversione Array

**Problema:** Crea una funzione che inverte un array.

**Soluzione:**
```c
#include <stdio.h>

/* Inverte un array sul posto */
void inverti_array(int arr[], int size)
{
    /* Usa due indici: uno all'inizio, uno alla fine */
    int start = 0;
    int end = size - 1;
    
    /* Scambia elementi finché i due indici si incontrano */
    while (start < end) {
        /* Swap usando variabile temporanea */
        int temp = arr[start];
        arr[start] = arr[end];
        arr[end] = temp;
        
        /* Muovi gli indici verso il centro */
        start++;
        end--;
    }
}

/* Stampa un array */
void stampa_array(int arr[], int size)
{
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main(void)
{
    int numeri[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int size = sizeof(numeri) / sizeof(numeri[0]);
    
    printf("Array originale: ");
    stampa_array(numeri, size);
    
    inverti_array(numeri, size);
    
    printf("Array invertito: ");
    stampa_array(numeri, size);
    
    return 0;
}
```

**Output:**
```
Array originale: 1 2 3 4 5 6 7 8 9 10
Array invertito: 10 9 8 7 6 5 4 3 2 1
```

### Esercizio 3: Ricerca Lineare

**Problema:** Implementa una funzione di ricerca lineare.

**Soluzione:**
```c
#include <stdio.h>

/* Cerca un elemento nell'array e ritorna l'indice
 * Ritorna -1 se non trovato */
int ricerca_lineare(int arr[], int size, int target)
{
    /* Controlla ogni elemento */
    for (int i = 0; i < size; i++) {
        if (arr[i] == target) {
            return i;  /* Trovato! Ritorna l'indice */
        }
    }
    
    return -1;  /* Non trovato */
}

int main(void)
{
    int numeri[] = {64, 34, 25, 12, 22, 11, 90};
    int size = sizeof(numeri) / sizeof(numeri[0]);
    int target;
    
    printf("Array: ");
    for (int i = 0; i < size; i++) {
        printf("%d ", numeri[i]);
    }
    printf("\n");
    
    printf("Inserisci numero da cercare: ");
    scanf("%d", &target);
    
    int indice = ricerca_lineare(numeri, size, target);
    
    if (indice != -1) {
        printf("Elemento %d trovato all'indice %d\n", target, indice);
    } else {
        printf("Elemento %d non trovato\n", target);
    }
    
    return 0;
}
```

## Punti Chiave da Ricordare

### Tipi di Dati
✅ Usa `int` per interi generali
✅ Usa `double` per numeri con decimali
✅ Preferisci `int32_t`, `uint32_t` per portabilità
✅ Ricorda che `int / int` dà un `int` (no decimali)

### Operatori
✅ Divisione intera tronca (10/3 = 3)
✅ Usa cast per divisione con decimali: `(double)a / b`
✅ Modulo `%` solo per interi
✅ Controlla divisione per zero!

### Array
✅ Gli indici partono da 0
✅ `size = sizeof(arr) / sizeof(arr[0])`
✅ Nessun controllo automatico dei bounds
✅ Array passati a funzioni sono puntatori

### Stringhe
✅ Servono sempre `n+1` caratteri (per `\0`)
✅ Usa `fgets` invece di `scanf` per sicurezza
✅ `strlen` ritorna lunghezza senza `\0`
✅ Compara stringhe con `strcmp`, non `==`

### Funzioni
✅ Passaggio per valore (copia)
✅ Usa puntatori per modificare l'originale
✅ Dichiarazioni (prototipi) prima di `main`
✅ `return` per funzioni non-void

### Loop
✅ `for`: quando sai quante iterazioni
✅ `while`: quando la condizione è al controllo
✅ `do-while`: esegue almeno una volta
✅ `break`: esce dal loop
✅ `continue`: salta al prossimo ciclo

## Risorse Aggiuntive

- **Glossario Comandi**: [GLOSSARIO_COMANDI.md](../GLOSSARIO_COMANDI.md) - Guida completa alla compilazione
- **Quick Reference**: [QUICK_REFERENCE.md](../QUICK_REFERENCE.md) - Riferimento rapido della sintassi

---

[← Introduzione](../01_Introduzione/README.md) | [Torna al Sommario](../README.md) | [Prossimo: Intermedio →](../03_Intermedio/README.md)
