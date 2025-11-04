# Modulo 06 - Comparazioni con Altri Linguaggi

## C vs Python

### Filosofia e Design

**C:**
- Low-level, close to the hardware
- Compilato
- Tipizzazione statica
- Gestione manuale della memoria
- Performance-oriented

**Python:**
- High-level, abstractions
- Interpretato (con bytecode)
- Tipizzazione dinamica
- Garbage collection automatica
- Productivity-oriented

### Hello World

**C:**
```c
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    return 0;
}
```

**Python:**
```python
print("Hello, World!")
```

### Gestione Memoria

**C - Manuale:**
```c
#include <stdlib.h>

int* create_array(int size) {
    int *arr = malloc(size * sizeof(int));
    if (arr == NULL) {
        return NULL;
    }
    
    for (int i = 0; i < size; i++) {
        arr[i] = i;
    }
    
    return arr;  // Chiamante deve fare free()!
}

int main(void) {
    int *arr = create_array(10);
    // Uso array...
    free(arr);  // IMPORTANTE: liberare memoria!
    return 0;
}
```

**Python - Automatica:**
```python
def create_array(size):
    return [i for i in range(size)]

arr = create_array(10)
# Uso array...
# Garbage collector libera automaticamente!
```

### Performance Comparison

**C - Fibonacci iterativo:**
```c
#include <stdio.h>
#include <time.h>

long fibonacci(int n) {
    if (n <= 1) return n;
    
    long prev = 0, curr = 1;
    for (int i = 2; i <= n; i++) {
        long next = prev + curr;
        prev = curr;
        curr = next;
    }
    return curr;
}

int main(void) {
    clock_t start = clock();
    
    for (int i = 0; i < 1000000; i++) {
        fibonacci(30);
    }
    
    clock_t end = clock();
    double time = ((double)(end - start)) / CLOCKS_PER_SEC;
    printf("Time: %.6f seconds\n", time);
    
    return 0;
}
```

**Python - Fibonacci iterativo:**
```python
import time

def fibonacci(n):
    if n <= 1:
        return n
    
    prev, curr = 0, 1
    for _ in range(2, n + 1):
        prev, curr = curr, prev + curr
    return curr

start = time.time()

for _ in range(1000000):
    fibonacci(30)

end = time.time()
print(f"Time: {end - start:.6f} seconds")
```

**Risultato tipico:**
- C: ~0.5 secondi
- Python: ~15 secondi
- **C è ~30x più veloce**

### Quando Usare C vs Python

**Usa C quando:**
- Hai bisogno di massime prestazioni
- Controllo preciso della memoria è critico
- Stai scrivendo sistemi operativi, driver, embedded
- Interfacciamento con hardware
- Latenza predittibile è necessaria

**Usa Python quando:**
- Sviluppo rapido è prioritario
- Prototipazione
- Scripting e automazione
- Data science e ML (con librerie C/C++ sottostanti)
- Web development (backend)

**Approccio Ibrido:**
Python può chiamare codice C per parti performance-critical!

```python
# Python con estensione C
import ctypes

# Carica libreria C
lib = ctypes.CDLL('./mylib.so')

# Chiama funzione C da Python
result = lib.my_fast_function(42)
```

## C vs Java

### Differenze Fondamentali

| Caratteristica | C | Java |
|----------------|---|------|
| Paradigma | Procedurale | Object-Oriented |
| Gestione Memoria | Manuale | Garbage Collection |
| Compilazione | Nativo | Bytecode (JVM) |
| Portabilità | Dipende da platform | Write Once, Run Anywhere |
| Performance | Massima | Buona (JIT) |
| Sicurezza Memoria | No bounds checking | Array bounds checked |

### Sintassi a Confronto

**C:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char name[50];
    int age;
} Person;

Person* create_person(const char *name, int age) {
    Person *p = malloc(sizeof(Person));
    strcpy(p->name, name);
    p->age = age;
    return p;
}

void print_person(const Person *p) {
    printf("Name: %s, Age: %d\n", p->name, p->age);
}

int main(void) {
    Person *p = create_person("Mario", 30);
    print_person(p);
    free(p);
    return 0;
}
```

**Java:**
```java
class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public void print() {
        System.out.println("Name: " + name + ", Age: " + age);
    }
}

public class Main {
    public static void main(String[] args) {
        Person p = new Person("Mario", 30);
        p.print();
        // Garbage collector libera automaticamente
    }
}
```

### Puntatori vs Riferimenti

**C - Puntatori espliciti:**
```c
#include <stdio.h>

void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main(void) {
    int x = 10, y = 20;
    printf("Before: x=%d, y=%d\n", x, y);
    swap(&x, &y);  // Esplicito: passa indirizzo
    printf("After: x=%d, y=%d\n", x, y);
    return 0;
}
```

**Java - Sempre per riferimento (oggetti):**
```java
class Value {
    int val;
    Value(int v) { val = v; }
}

class Main {
    static void modify(Value v) {
        v.val = 100;  // Modifica oggetto originale
    }
    
    public static void main(String[] args) {
        Value v = new Value(42);
        System.out.println("Before: " + v.val);
        modify(v);  // Passa riferimento
        System.out.println("After: " + v.val);
    }
}
```

### Gestione Eccezioni

**C - Error codes:**
```c
#include <stdio.h>
#include <errno.h>

int divide(int a, int b, int *result) {
    if (b == 0) {
        return -1;  // Error code
    }
    *result = a / b;
    return 0;  // Success
}

int main(void) {
    int result;
    if (divide(10, 0, &result) != 0) {
        fprintf(stderr, "Error: division by zero\n");
        return 1;
    }
    printf("Result: %d\n", result);
    return 0;
}
```

**Java - Try-catch:**
```java
class Main {
    static int divide(int a, int b) throws ArithmeticException {
        if (b == 0) {
            throw new ArithmeticException("Division by zero");
        }
        return a / b;
    }
    
    public static void main(String[] args) {
        try {
            int result = divide(10, 0);
            System.out.println("Result: " + result);
        } catch (ArithmeticException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }
}
```

### Quando Usare C vs Java

**Usa C quando:**
- Sistemi embedded con risorse limitate
- Real-time systems
- Necessità di controllo preciso dell'hardware
- Performance critiche e prevedibili
- Footprint minimo di memoria

**Usa Java quando:**
- Applicazioni enterprise
- Android development
- Necessità di portabilità garantita
- Sviluppo rapido con librerie ricche
- Sicurezza e robustezza sono prioritarie

## C vs Rust

### Sicurezza della Memoria

**C - Unsafe by default:**
```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int *ptr = malloc(sizeof(int));
    *ptr = 42;
    
    free(ptr);
    
    // ERRORE: use-after-free (undefined behavior!)
    *ptr = 100;
    printf("%d\n", *ptr);  // Potrebbe crashare o comportarsi stranamente
    
    // ERRORE: double free
    free(ptr);  // Undefined behavior!
    
    return 0;
}
```

**Rust - Safe by default:**
```rust
fn main() {
    let mut x = Box::new(42);
    println!("{}", x);
    
    // Dopo questo, x non è più valido
    drop(x);
    
    // ERRORE DI COMPILAZIONE: borrow of moved value
    // println!("{}", x);  // Non compila!
    
    // Rust previene use-after-free e double-free a compile time!
}
```

### Ownership e Borrowing

**C - Manuale:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

char* create_string(void) {
    char *s = malloc(20);
    strcpy(s, "Hello");
    return s;  // Chi è responsabile di free()?
}

void use_string(char *s) {
    printf("%s\n", s);
    // Dovrei fare free() qui? Chi è il owner?
}

int main(void) {
    char *s = create_string();
    use_string(s);
    free(s);  // Speriamo di non dimenticarlo!
    return 0;
}
```

**Rust - Ownership automatico:**
```rust
fn create_string() -> String {
    String::from("Hello")
    // Ownership trasferito al chiamante
}

fn use_string(s: &String) {
    // Borrow, non ownership
    println!("{}", s);
}  // Borrow finisce, ma s non viene distrutto

fn main() {
    let s = create_string();  // s owns the string
    use_string(&s);            // Borrow temporaneo
    // s automaticamente distrutto qui
}
```

### Null Safety

**C - Null pointers pericolosi:**
```c
#include <stdio.h>
#include <stdlib.h>

void process(int *ptr) {
    // Dovrei controllare NULL? Il chiamante ha già controllato?
    *ptr = 42;  // Se ptr è NULL -> CRASH!
}

int main(void) {
    int *ptr = NULL;
    // Dimenticato di allocare!
    process(ptr);  // CRASH!
    return 0;
}
```

**Rust - Option type:**
```rust
fn process(value: Option<i32>) -> i32 {
    match value {
        Some(v) => v * 2,
        None => 0,
    }
    // Impossibile dimenticare di gestire None!
}

fn main() {
    let x = Some(21);
    let y = None;
    
    println!("{}", process(x));  // 42
    println!("{}", process(y));  // 0
}
```

### Concorrenza

**C - Unsafe data races:**
```c
#include <stdio.h>
#include <pthread.h>

int counter = 0;  // Shared mutable state

void* increment(void *arg) {
    for (int i = 0; i < 100000; i++) {
        counter++;  // DATA RACE! Non thread-safe
    }
    return NULL;
}

int main(void) {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, increment, NULL);
    pthread_create(&t2, NULL, increment, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    printf("Counter: %d\n", counter);  // Risultato imprevedibile!
    return 0;
}
```

**Rust - Fearless concurrency:**
```rust
use std::thread;
use std::sync::{Arc, Mutex};

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..2 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..100000 {
                let mut num = counter.lock().unwrap();
                *num += 1;
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Counter: {}", *counter.lock().unwrap());
    // Risultato sempre 200000!
}
```

### Performance

**C:**
```c
#include <stdio.h>
#include <time.h>

long sum_array(int *arr, int size) {
    long sum = 0;
    for (int i = 0; i < size; i++) {
        sum += arr[i];
    }
    return sum;
}
```

**Rust:**
```rust
fn sum_array(arr: &[i32]) -> i64 {
    arr.iter().map(|&x| x as i64).sum()
}
```

**Performance: Praticamente identica!**
Rust ha "zero-cost abstractions" - le abstrazioni ad alto livello compilano a codice efficiente come C.

### Quando Usare C vs Rust

**Usa C quando:**
- Codice legacy esistente
- Interoperabilità con API C esistenti
- Supporto universale richiesto (C è ovunque)
- Progetti molto piccoli e semplici
- Team già esperto in C

**Usa Rust quando:**
- Nuovi progetti dove sicurezza è critica
- Concorrenza complessa
- Vuoi performance di C con sicurezza
- Disposto ad investire nella curva di apprendimento
- Progetti a lungo termine che necessitano manutenzione

## Tabella Riassuntiva

| Feature | C | Python | Java | Rust |
|---------|---|--------|------|------|
| **Performance** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Memory Safety** | ❌ | ✅ (GC) | ✅ (GC) | ✅ (Ownership) |
| **Learning Curve** | ⭐⭐⭐ | ⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Productivity** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Portability** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Low-Level Control** | ⭐⭐⭐⭐⭐ | ❌ | ❌ | ⭐⭐⭐⭐⭐ |
| **Concurrency** | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Ecosystem** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Compile Time** | ⭐⭐⭐⭐⭐ | N/A | ⭐⭐⭐⭐ | ⭐⭐ |

## Interoperabilità

### C chiamato da Python (ctypes)

**mylib.c:**
```c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

void print_hello(void) {
    printf("Hello from C!\n");
}
```

Compila:
```bash
gcc -shared -fPIC mylib.c -o mylib.so
```

**use_c.py:**
```python
import ctypes

# Carica libreria
lib = ctypes.CDLL('./mylib.so')

# Chiama funzioni C
result = lib.add(5, 3)
print(f"5 + 3 = {result}")

lib.print_hello()
```

### C chiamato da Java (JNI)

**MyLib.c:**
```c
#include <jni.h>
#include "MyLib.h"

JNIEXPORT jint JNICALL Java_MyLib_add(JNIEnv *env, jobject obj, jint a, jint b) {
    return a + b;
}
```

**MyLib.java:**
```java
public class MyLib {
    static {
        System.loadLibrary("mylib");
    }
    
    public native int add(int a, int b);
    
    public static void main(String[] args) {
        MyLib lib = new MyLib();
        System.out.println("5 + 3 = " + lib.add(5, 3));
    }
}
```

### C chiamato da Rust (FFI)

**lib.c:**
```c
int add(int a, int b) {
    return a + b;
}
```

**main.rs:**
```rust
extern "C" {
    fn add(a: i32, b: i32) -> i32;
}

fn main() {
    unsafe {
        let result = add(5, 3);
        println!("5 + 3 = {}", result);
    }
}
```

## Conclusione

- **C**: Massima performance e controllo, ma richiede disciplina
- **Python**: Produttività e semplicità, sacrifica performance
- **Java**: Bilanciamento tra performance e sicurezza, grande ecosistema
- **Rust**: Performance di C con sicurezza moderna, curva apprendimento ripida

**Non esiste un linguaggio "migliore"** - scegli lo strumento giusto per il problema!

---

[← Assembly](../05_Assembly_LowLevel/README.md) | [Torna al Sommario](../README.md) | [Prossimo: Laboratori →](../07_Laboratori/README.md)
