# Modulo 03.5 - Undefined Behavior: Il Nemico Invisibile

## 🎯 Obiettivo del Modulo

**Undefined Behavior (UB)** è uno dei concetti più pericolosi e misconosciuti del C. Un programma con UB può:
- Funzionare perfettamente sul tuo computer
- Crashare sul computer di qualcun altro
- Essere sfruttato da un attaccante
- Comportarsi diversamente con ottimizzazioni diverse

**Questo modulo ti insegnerà:**
- Cos'è l'Undefined Behavior e perché esiste
- I 15+ tipi più comuni di UB in C
- Come riconoscerli e prevenirli
- Tools per detectare UB (sanitizers, static analysis)
- Come pensare "difensivamente" per evitare UB

---

## ⚠️ Perché l'Undefined Behavior è Pericoloso

```c
// ESEMPIO FAMOSO:

int main() {
    int x = 0;
    x = x++ + ++x;  // UB!
    printf("%d\n", x);
    return 0;
}

/* POSSIBILI OUTPUT:
 * GCC -O0: 3
 * GCC -O2: 5
 * Clang: 4
 * MSVC: 2
 * 
 * TUTTI VALIDI secondo lo standard C!
 * Il compilatore può fare QUALSIASI COSA con UB!
 */
```

**Lo standard C dice:**
> "Undefined behavior: behavior, upon use of a nonportable or erroneous program construct, for which this International Standard imposes no requirements."

**Traduzione:** Il compilatore può fare LETTERALMENTE qualsiasi cosa:
- ✓ Funzionare come ti aspetti
- ✓ Crashare
- ✓ Formattare il tuo hard disk (teoricamente!)
- ✓ Far partire i missili nucleari (esempio classico)

**In pratica:**
- Il compilatore ASSUME che non ci sia mai UB
- Ottimizza basandosi su questo assunto
- Se c'è UB, le ottimizzazioni causano comportamenti imprevedibili

---

## 📚 Catalogo Completo degli UB

### 1. [Buffer Overflow](01_buffer_overflow.md)
Scrivere oltre i limiti di un array.

```c
int arr[5];
arr[10] = 42;  // UB! Scrittura fuori bounds
```

### 2. [Use-After-Free](02_use_after_free.md)
Usare memoria dopo averla liberata.

```c
int *p = malloc(sizeof(int));
free(p);
*p = 10;  // UB! p è dangling pointer
```

### 3. [NULL Pointer Dereference](03_null_dereference.md)
Dereferenziare un puntatore NULL.

```c
int *p = NULL;
*p = 10;  // UB! Segmentation fault
```

### 4. [Signed Integer Overflow](04_signed_overflow.md)
Overflow di interi con segno.

```c
int x = INT_MAX;
x++;  // UB! Overflow signed
```

### 5. [Uninitialized Variables](05_uninitialized.md)
Leggere variabili non inizializzate.

```c
int x;
printf("%d\n", x);  // UB! x non inizializzata
```

### 6. [Sequence Point Violations](06_sequence_points.md)
Modificare stessa variabile più volte tra sequence point.

```c
int x = 5;
x = x++ + ++x;  // UB!
```

### 7. [Division by Zero](07_division_zero.md)
Divisione per zero.

```c
int x = 10 / 0;  // UB!
```

### 8. [Invalid Pointer Arithmetic](08_pointer_arithmetic.md)
Aritmetica puntatori fuori bounds.

```c
int arr[5];
int *p = arr + 10;  // UB! Oltre array
```

### 9. [Type Punning](09_type_punning.md)
Accesso a memoria con tipo sbagliato (strict aliasing).

```c
int x = 42;
float *fp = (float*)&x;
*fp = 3.14;  // UB! Strict aliasing violation
```

### 10. [Data Race](10_data_race.md)
Accesso concorrente a stessa variabile senza sincronizzazione.

```c
// Thread 1: x = 10;
// Thread 2: y = x;  // UB se concorrente!
```

---

## 🛡️ Come Difendersi dall'UB

### 1. Compilatore Warnings

```bash
# Abilita TUTTI i warning!
gcc -Wall -Wextra -Werror -pedantic program.c

# Warning aggiuntivi utili:
gcc -Wconversion -Wshadow -Wformat=2 -Wunused program.c
```

### 2. Sanitizers (FONDAMENTALI!)

```bash
# AddressSanitizer: detect buffer overflow, use-after-free
gcc -fsanitize=address -g program.c
./a.out

# UndefinedBehaviorSanitizer: detect molti UB
gcc -fsanitize=undefined -g program.c
./a.out

# ThreadSanitizer: detect data race
gcc -fsanitize=thread -g program.c
./a.out

# MemorySanitizer: detect uninitialized reads
clang -fsanitize=memory -g program.c
./a.out
```

### 3. Static Analysis

```bash
# Clang Static Analyzer
scan-build gcc program.c

# Cppcheck
cppcheck --enable=all program.c

# Splint
splint +strict program.c
```

### 4. Valgrind

```bash
# Memory error detector
valgrind --leak-check=full --show-leak-kinds=all ./program

# Uninitialized value detector
valgrind --track-origins=yes ./program
```

---

## 📊 Tabella Riepilogativa UB

```
┌────────────────────────┬──────────────┬─────────────┬──────────────┐
│ TIPO UB                │ FREQUENZA    │ PERICOLOSITÀ│ DETECTION    │
├────────────────────────┼──────────────┼─────────────┼──────────────┤
│ Buffer Overflow        │ ★★★★★        │ ★★★★★       │ ASan         │
│ Use-After-Free         │ ★★★★☆        │ ★★★★★       │ ASan         │
│ NULL Dereference       │ ★★★★★        │ ★★★☆☆       │ ASan/Valgrind│
│ Signed Overflow        │ ★★★☆☆        │ ★★★★☆       │ UBSan        │
│ Uninitialized Vars     │ ★★★★☆        │ ★★★☆☆       │ MSan/Valgrind│
│ Sequence Point Viol.   │ ★★☆☆☆        │ ★★★★☆       │ Compiler warn│
│ Division by Zero       │ ★★★☆☆        │ ★★☆☆☆       │ UBSan        │
│ Invalid Ptr Arith      │ ★★★☆☆        │ ★★★★☆       │ ASan         │
│ Type Punning           │ ★★☆☆☆        │ ★★★☆☆       │ UBSan        │
│ Data Race              │ ★★★☆☆        │ ★★★★★       │ TSan         │
└────────────────────────┴──────────────┴─────────────┴──────────────┘

LEGGENDA:
★★★★★ = Molto comune/pericoloso
☆ = Raro/poco pericoloso

ASan = AddressSanitizer
UBSan = UndefinedBehaviorSanitizer
MSan = MemorySanitizer
TSan = ThreadSanitizer
```

---

## 💡 Principi Difensivi

### PRINCIPIO 1: Zero Trust
> "Non fidarti mai dell'input, nemmeno del tuo"

```c
// ❌ FIDUCIA CIECA
void process(int *arr, int size) {
    for (int i = 0; i < size; i++) {
        arr[i] = 0;  // Cosa se arr è NULL? size negativo?
    }
}

// ✅ DIFENSIVO
void process_safe(int *arr, int size) {
    if (arr == NULL) {
        fprintf(stderr, "Error: NULL array\n");
        return;
    }
    if (size < 0) {
        fprintf(stderr, "Error: negative size\n");
        return;
    }
    for (int i = 0; i < size; i++) {
        arr[i] = 0;
    }
}
```

### PRINCIPIO 2: Fail Fast
> "Crash subito è meglio di corruzione silente"

```c
// ❌ CONTINUA CON ERRORE
int *allocate(size_t size) {
    int *p = malloc(size);
    return p;  // Potrebbe essere NULL!
}

// ✅ FAIL FAST
int *allocate_safe(size_t size) {
    int *p = malloc(size);
    if (p == NULL) {
        fprintf(stderr, "Out of memory!\n");
        exit(EXIT_FAILURE);  // Termina invece di continuare
    }
    return p;
}
```

### PRINCIPIO 3: Const Correctness
> "Se non deve cambiare, marcalo const"

```c
// ✅ USO CONST
void print_array(const int *arr, int size) {
    // arr non può essere modificato
    // Previene errori accidentali
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
}
```

---

## 🎯 Workflow per Evitare UB

### Durante Development:

1. **Scrivi codice con warning abilitati**
   ```bash
   gcc -Wall -Wextra -Werror ...
   ```

2. **Testa con sanitizers**
   ```bash
   gcc -fsanitize=address,undefined -g ...
   ```

3. **Valgrind periodico**
   ```bash
   valgrind --leak-check=full ./program
   ```

4. **Code review**: Cerca pattern UB comuni

### Prima del Release:

1. **Static analysis completa**
   ```bash
   scan-build make
   cppcheck --enable=all src/
   ```

2. **Test coverage elevato** (idealmente >80%)

3. **Fuzzing** (per input esterni)

4. **Tutti i sanitizer** su test suite

---

## 📖 Come Usare Questo Modulo

1. **Leggi ogni tipo di UB** in ordine
2. **Compila gli esempi** con sanitizers
3. **Vedi gli errori** in azione
4. **Impara a riconoscerli** nel tuo codice
5. **Adotta pratiche difensive**

---

## 🚨 Mantra Anti-UB

Ripeti ogni giorno:

```
✓ Controllo SEMPRE i bounds
✓ Inizializzo SEMPRE le variabili
✓ Verifico SEMPRE malloc != NULL
✓ NON uso DOPO free()
✓ Compilo con -Wall -Wextra
✓ Testo con sanitizers
✓ Valgrind è mio amico
```

---

[Inizia: Buffer Overflow →](01_buffer_overflow.md) | [Torna al Sommario](../README.md)
