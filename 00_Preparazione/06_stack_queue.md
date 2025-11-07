# Lezione 6 - Strutture Dati: Stack e Queue

## 🎯 Obiettivi della Lezione

Alla fine di questa lezione sarai in grado di:
- Comprendere i principi LIFO (Stack) e FIFO (Queue)
- Visualizzare come stack e queue operano in memoria
- Implementare stack e queue usando array o liste
- Riconoscere quando usare stack vs queue
- Debuggare problemi di overflow e underflow

---

## 1. PROBLEMA: Perché Servono Stack e Queue?

### Scenario 1: Annulla/Ripristina (Undo/Redo)

Immagina un editor di testo. Ogni modifica viene salvata. Quando premi "Annulla", quale modifica viene annullata?

```
Modifiche:
1. Scrivi "Hello"
2. Scrivi "World"
3. Cancella "World"

Premi ANNULLA → Quale operazione si annulla?
```

**Risposta:** L'ULTIMA (la più recente) → **LIFO (Last In, First Out)** → **STACK**!

### Scenario 2: Stampante con Coda di Lavori

Una stampante riceve 3 documenti da stampare. In che ordine li stampa?

```
Documenti ricevuti:
1. Documento A (9:00)
2. Documento B (9:01)
3. Documento C (9:02)

Ordine di stampa?
```

**Risposta:** Il PRIMO arrivato viene stampato per primo → **FIFO (First In, First Out)** → **QUEUE**!

---

## 2. STACK (Pila) - LIFO

### 2.1 Concetto di Stack

Immagina una **pila di piatti**:
- Puoi aggiungere un piatto solo SOPRA
- Puoi togliere un piatto solo da SOPRA
- Non puoi prendere un piatto dal centro!

```
┌─────────┐
│  Piatto │ ← TOP (cima)
├─────────┤
│  Piatto │
├─────────┤
│  Piatto │
├─────────┤
│  Piatto │
└─────────┘
    ↑
  BOTTOM (fondo)

OPERAZIONI:
- PUSH: Aggiungi piatto in CIMA
- POP: Rimuovi piatto dalla CIMA
```

### 2.2 Operazioni Fondamentali

```
OPERAZIONI STACK:
┌──────────┬──────────────────────────────────┐
│ push(x)  │ Inserisce x in cima allo stack   │
│ pop()    │ Rimuove e ritorna l'elemento top │
│ top()    │ Ritorna top SENZA rimuoverlo     │
│ isEmpty()│ Verifica se stack è vuoto        │
│ isFull() │ Verifica se stack è pieno (array)│
└──────────┴──────────────────────────────────┘
```

### 2.3 Visualizzazione Passo-Passo

```
SEQUENZA: push(10), push(20), push(30), pop(), push(40)

STATO INIZIALE:
┌─────────┐
│ (vuoto) │
└─────────┘
top = -1

DOPO push(10):
┌─────────┐
│   10    │ ← top = 0
└─────────┘

DOPO push(20):
┌─────────┐
│   20    │ ← top = 1
├─────────┤
│   10    │
└─────────┘

DOPO push(30):
┌─────────┐
│   30    │ ← top = 2
├─────────┤
│   20    │
├─────────┤
│   10    │
└─────────┘

DOPO pop() → ritorna 30:
┌─────────┐
│   20    │ ← top = 1
├─────────┤
│   10    │
└─────────┘

DOPO push(40):
┌─────────┐
│   40    │ ← top = 2
├─────────┤
│   20    │
├─────────┤
│   10    │
└─────────┘
```

### 2.4 Implementazione con Array

**STRUTTURA:**
```
Stack:
┌────────────────────────────────────┐
│ array: [10][20][30][ ][ ][ ][ ]    │
│ top: 2                             │
│ capacity: 7                        │
└────────────────────────────────────┘
              ↑
         Prossimo slot libero

INVARIANTE: 
  - top punta all'ULTIMO elemento inserito
  - Se top = -1 → stack vuoto
  - Se top = capacity-1 → stack pieno
```

**PSEUDOCODICE:**
```
STRUTTURA Stack:
    array[MAX_SIZE]
    top = -1
    capacity = MAX_SIZE

FUNZIONE push(x):
    SE top == capacity - 1:
        ERRORE: "Stack Overflow!"
        RETURN False
    
    top++
    array[top] = x
    RETURN True

FUNZIONE pop():
    SE top == -1:
        ERRORE: "Stack Underflow!"
        RETURN NULL
    
    valore = array[top]
    top--
    RETURN valore

FUNZIONE peek():  // Guarda top senza rimuovere
    SE top == -1:
        RETURN NULL
    RETURN array[top]

FUNZIONE isEmpty():
    RETURN top == -1

FUNZIONE isFull():
    RETURN top == capacity - 1
```

**COMPLESSITÀ:**
```
┌───────────┬──────────────┐
│ Operazione│ Complessità  │
├───────────┼──────────────┤
│ push()    │ O(1) ⚡      │
│ pop()     │ O(1) ⚡      │
│ peek()    │ O(1) ⚡      │
│ isEmpty() │ O(1) ⚡      │
└───────────┴──────────────┘

Tutte le operazioni sono COSTANTI!
```

### 2.5 Implementazione con Lista Concatenata

**STRUTTURA:**
```
Stack con lista:
┌─ top ─────────┐
│ ptr: 0x3000 ──┼──→ [30] → [20] → [10] → NULL
└───────────────┘    ↑
                   CIMA (push/pop qui)

PRO:
✓ Nessun limite di dimensione
✓ Nessuno spreco di memoria

CONTRO:
✗ Overhead di 8 byte per elemento (puntatore)
✗ Allocazioni dinamiche più lente
```

**PSEUDOCODICE:**
```
STRUTTURA Node:
    data
    next

STRUTTURA Stack:
    top = NULL

FUNZIONE push(x):
    nuovo_nodo = ALLOCA(Node)
    nuovo_nodo.data = x
    nuovo_nodo.next = top
    top = nuovo_nodo

FUNZIONE pop():
    SE top == NULL:
        ERRORE: "Stack Underflow!"
        RETURN NULL
    
    valore = top.data
    temp = top
    top = top.next
    LIBERA(temp)
    RETURN valore
```

---

## 3. QUEUE (Coda) - FIFO

### 3.1 Concetto di Queue

Immagina una **fila al supermercato**:
- Le persone entrano dalla FINE della fila
- Le persone escono dall'INIZIO della fila
- Chi arriva PRIMA, viene servito PRIMA!

```
FRONT (inizio) ────────────────────→ REAR (fine)
   ↓                                     ↓
[Persona1] → [Persona2] → [Persona3] → [Persona4]
   ↑                                        ↑
 Esce qui                              Entra qui

OPERAZIONI:
- ENQUEUE: Aggiungi alla FINE (rear)
- DEQUEUE: Rimuovi dall'INIZIO (front)
```

### 3.2 Operazioni Fondamentali

```
OPERAZIONI QUEUE:
┌────────────┬─────────────────────────────────┐
│ enqueue(x) │ Inserisce x alla fine           │
│ dequeue()  │ Rimuove e ritorna primo elemento│
│ front()    │ Ritorna primo SENZA rimuoverlo  │
│ isEmpty()  │ Verifica se queue è vuota       │
│ isFull()   │ Verifica se queue è piena       │
└────────────┴─────────────────────────────────┘
```

### 3.3 Visualizzazione Passo-Passo

```
SEQUENZA: enqueue(10), enqueue(20), enqueue(30), dequeue(), enqueue(40)

STATO INIZIALE:
┌───────────────────────────────────┐
│ [ ][ ][ ][ ][ ]                   │
└───────────────────────────────────┘
front = -1, rear = -1

DOPO enqueue(10):
┌───────────────────────────────────┐
│ [10][ ][ ][ ][ ]                  │
└───────────────────────────────────┘
 ↑
front=0, rear=0

DOPO enqueue(20):
┌───────────────────────────────────┐
│ [10][20][ ][ ][ ]                 │
└───────────────────────────────────┘
  ↑   ↑
front=0, rear=1

DOPO enqueue(30):
┌───────────────────────────────────┐
│ [10][20][30][ ][ ]                │
└───────────────────────────────────┘
  ↑       ↑
front=0, rear=2

DOPO dequeue() → ritorna 10:
┌───────────────────────────────────┐
│ [  ][20][30][ ][ ]                │
└───────────────────────────────────┘
      ↑   ↑
   front=1, rear=2

DOPO enqueue(40):
┌───────────────────────────────────┐
│ [  ][20][30][40][ ]               │
└───────────────────────────────────┘
      ↑       ↑
   front=1, rear=3
```

### 3.4 Problema: Queue Lineare Spreca Spazio!

```
DOPO molti enqueue/dequeue:
┌───────────────────────────────────┐
│ [  ][  ][  ][  ][50][60]          │
└───────────────────────────────────┘
                  ↑   ↑
               front=4, rear=5

PROBLEMA: Gli slot 0-3 sono VUOTI ma NON RIUTILIZZABILI!
rear è vicino alla fine, ma c'è spazio all'inizio.

SOLUZIONE: CIRCULAR QUEUE (Coda Circolare)!
```

### 3.5 Circular Queue (Coda Circolare)

**CONCETTO:**
```
Tratta l'array come se fosse CIRCOLARE:

    [0] ← inizio
    [1]
    [2]
    [3]
    [4] ← fine
     ↓
    [0] ← ricomincia!

Quando rear raggiunge la fine, "avvolge" e torna all'inizio.
```

**VISUALIZZAZIONE:**
```
ARRAY LOGICO (vista lineare):
┌───────────────────────────────────┐
│ [10][20][30][40][50]              │
└───────────────────────────────────┘
  ↑                   ↑
front=0             rear=4

DOPO enqueue(60) - rear avvolge:
┌───────────────────────────────────┐
│ [60][20][30][40][50]              │
└───────────────────────────────────┘
   ↑  ↑
 rear=0  front=1 (dopo dequeue di 10)

VISTA CIRCOLARE:
        [0]
    [1]     [4]
    [2] [3]
    
    ↻ Avvolge circolarmente
```

**PSEUDOCODICE:**
```
STRUTTURA CircularQueue:
    array[MAX_SIZE]
    front = -1
    rear = -1
    capacity = MAX_SIZE
    size = 0

FUNZIONE enqueue(x):
    SE isFull():
        ERRORE: "Queue Overflow!"
        RETURN False
    
    SE isEmpty():
        front = 0
        rear = 0
    ALTRIMENTI:
        rear = (rear + 1) % capacity  // Avvolgimento circolare!
    
    array[rear] = x
    size++
    RETURN True

FUNZIONE dequeue():
    SE isEmpty():
        ERRORE: "Queue Underflow!"
        RETURN NULL
    
    valore = array[front]
    
    SE size == 1:  // Un solo elemento
        front = -1
        rear = -1
    ALTRIMENTI:
        front = (front + 1) % capacity  // Avvolgimento!
    
    size--
    RETURN valore

FUNZIONE isEmpty():
    RETURN size == 0

FUNZIONE isFull():
    RETURN size == capacity
```

**OPERATORE MODULO (%):**
```
(rear + 1) % capacity assicura l'avvolgimento:

capacity = 5
rear = 4
(rear + 1) % capacity = (4 + 1) % 5 = 0  ← Torna all'inizio!

rear = 2
(rear + 1) % capacity = (2 + 1) % 5 = 3  ← Incremento normale
```

### 3.6 Implementazione con Lista Concatenata

**STRUTTURA:**
```
Queue con lista:
┌─ front ────┐      ┌─ rear ─────┐
│ ptr: 0x2000│──→[10]→[20]→[30]←──│ptr: 0x4000│
└────────────┘                    └───────────┘
     ↑                                  ↑
   DEQUEUE qui                    ENQUEUE qui

PRO:
✓ Nessun limite di dimensione
✓ Nessuno spreco di spazio
✓ Nessun problema di avvolgimento

CONTRO:
✗ Overhead memoria (puntatori)
✗ Allocazioni dinamiche
```

**PSEUDOCODICE:**
```
STRUTTURA Node:
    data
    next

STRUTTURA Queue:
    front = NULL
    rear = NULL

FUNZIONE enqueue(x):
    nuovo_nodo = ALLOCA(Node)
    nuovo_nodo.data = x
    nuovo_nodo.next = NULL
    
    SE isEmpty():
        front = nuovo_nodo
        rear = nuovo_nodo
    ALTRIMENTI:
        rear.next = nuovo_nodo
        rear = nuovo_nodo

FUNZIONE dequeue():
    SE isEmpty():
        ERRORE: "Queue Underflow!"
        RETURN NULL
    
    valore = front.data
    temp = front
    front = front.next
    
    SE front == NULL:  // Queue diventa vuota
        rear = NULL
    
    LIBERA(temp)
    RETURN valore
```

---

## 4. CONFRONTO: Stack vs Queue

```
┌─────────────────┬──────────────────┬──────────────────┐
│  CARATTERISTICA │      STACK       │      QUEUE       │
├─────────────────┼──────────────────┼──────────────────┤
│ Principio       │ LIFO             │ FIFO             │
│ Inserimento     │ push() - top     │ enqueue() - rear │
│ Rimozione       │ pop() - top      │ dequeue() - front│
│ Accesso         │ Solo top         │ Solo front       │
│ Applicazioni    │ Undo/Redo        │ Scheduling       │
│                 │ Backtracking     │ Buffering        │
│                 │ Call stack       │ BFS (algoritmi)  │
│ Implementazione │ Array o Lista    │ Array o Lista    │
│ Complessità ops │ O(1) tutte       │ O(1) tutte       │
└─────────────────┴──────────────────┴──────────────────┘
```

---

## 5. APPLICAZIONI PRATICHE

### 5.1 Stack: Call Stack (Stack delle Chiamate)

Quando chiami una funzione, il sistema usa uno stack:

```
CODICE:
void funzioneC() {
    int z = 30;
}

void funzioneB() {
    int y = 20;
    funzioneC();
}

void funzioneA() {
    int x = 10;
    funzioneB();
}

CALL STACK durante l'esecuzione:

INIZIO: funzioneA()
┌──────────────────┐
│ funzioneA        │
│ x = 10           │
└──────────────────┘

CHIAMA funzioneB():
┌──────────────────┐
│ funzioneB        │ ← TOP
│ y = 20           │
├──────────────────┤
│ funzioneA        │
│ x = 10           │
└──────────────────┘

CHIAMA funzioneC():
┌──────────────────┐
│ funzioneC        │ ← TOP
│ z = 30           │
├──────────────────┤
│ funzioneB        │
│ y = 20           │
├──────────────────┤
│ funzioneA        │
│ x = 10           │
└──────────────────┘

funzioneC() termina → POP:
┌──────────────────┐
│ funzioneB        │ ← TOP
│ y = 20           │
├──────────────────┤
│ funzioneA        │
│ x = 10           │
└──────────────────┘

funzioneB() termina → POP:
┌──────────────────┐
│ funzioneA        │ ← TOP
│ x = 10           │
└──────────────────┘

funzioneA() termina → POP → STACK VUOTO
```

### 5.2 Stack: Validazione Parentesi

```
PROBLEMA: Verifica se parentesi sono bilanciate

Input: "((()))" → Valido ✓
Input: "(()"   → Invalido ✗

ALGORITMO:
1. Per ogni carattere:
   - Se '(' → push nello stack
   - Se ')' → pop dallo stack
2. Alla fine: stack vuoto? → Valido

ESEMPIO: "(())"
Step 1: '(' → push → Stack: [(]
Step 2: '(' → push → Stack: [(, (]
Step 3: ')' → pop  → Stack: [(]
Step 4: ')' → pop  → Stack: []
STACK VUOTO → VALIDO! ✓
```

### 5.3 Queue: Gestione Processi (Scheduling)

```
SISTEMA OPERATIVO: CPU scheduler

Processi in attesa:
┌──────────────────────────────────────────┐
│ [P1] → [P2] → [P3] → [P4]                │
└──────────────────────────────────────────┘
  ↑                          ↑
FRONT (prossimo da eseguire) REAR (nuovo processo)

1. CPU prende P1 dalla FRONT → dequeue()
2. Esegue P1 per un time slice
3. P1 non finito? → enqueue(P1) alla REAR
4. CPU prende P2 dalla FRONT → dequeue()
...

ROUND-ROBIN SCHEDULING = Queue!
```

### 5.4 Queue: Buffer di Stampa

```
STAMPANTE riceve documenti:

┌─ Buffer (Queue) ──────────────────────┐
│ FRONT                          REAR   │
│ [Doc A] → [Doc B] → [Doc C]           │
└────────────────────────────────────────┘
    ↑
  Stampo questo

1. Arriva Doc D → enqueue(Doc D)
2. Doc A stampato → dequeue()
3. Ora stampo Doc B
4. ...

FIFO garantisce EQUITÀ: primo arrivato, primo servito!
```

---

## 6. ⚠️ ERRORI MORTALI

### ERRORE #1: Stack Overflow

```
// ❌ DISASTRO!
void push(int x) {
    stack[++top] = x;  // NESSUN CONTROLLO!
}

/* SE top == capacity-1 e chiami push():
 * 
 * top diventa capacity (FUORI BOUNDS!)
 * stack[capacity] = SCRITTURA OLTRE L'ARRAY!
 * 
 * ┌─ Array stack ──────┐
 * │ [0][1]...[cap-1]   │
 * └────────────────────┘
 *                      ↓
 *                   stack[cap] ← OVERFLOW!
 * 
 * CONSEGUENZE:
 *   - Corruzione memoria
 *   - Crash
 *   - Comportamento imprevedibile
 */

// ✅ SOLUZIONE: Controlla sempre!
bool push(int x) {
    if (top >= capacity - 1) {
        fprintf(stderr, "Stack Overflow!\n");
        return false;
    }
    stack[++top] = x;
    return true;
}
```

### ERRORE #2: Stack Underflow

```
// ❌ DISASTRO!
int pop() {
    return stack[top--];  // NESSUN CONTROLLO!
}

/* SE stack è VUOTO (top == -1):
 * 
 * Ritorna stack[-1] ← ACCESSO INVALIDO!
 * top diventa -2, -3, ... ← DISASTRO!
 * 
 * CONSEGUENZE:
 *   - Lettura memoria casuale
 *   - Valori spazzatura
 *   - Crash potenziale
 */

// ✅ SOLUZIONE: Verifica isEmpty()
int pop() {
    if (isEmpty()) {
        fprintf(stderr, "Stack Underflow!\n");
        return -1;  // O gestisci l'errore
    }
    return stack[top--];
}
```

### ERRORE #3: Queue Circolare - Condizione Full Sbagliata

```
// ❌ APPROCCIO INGENUO:
bool isFull() {
    return (rear + 1) % capacity == front;
}

/* PROBLEMA: Questa condizione è vera ANCHE quando queue è VUOTA!
 * 
 * Queue vuota:    front=0, rear=0 → (0+1)%cap == 0 ? NO
 * Queue piena:    front=0, rear=cap-1 → (cap-1+1)%cap == 0 ? SÌ
 * 
 * Ma cosa se front=rear in generale?
 * NON PUOI DISTINGUERE pieno da vuoto!
 */

// ✅ SOLUZIONE: Usa variabile size
struct Queue {
    int array[MAX_SIZE];
    int front;
    int rear;
    int size;  // ← Traccia numero elementi
};

bool isEmpty() { return size == 0; }
bool isFull() { return size == capacity; }
```

### ERRORE #4: Memory Leak con Lista

```
// ❌ DISASTRO!
void clear_stack() {
    top = NULL;  // Perdi riferimento a TUTTI i nodi!
}

/* MEMORIA PRIMA:
 * top → [A] → [B] → [C] → NULL
 * 
 * MEMORIA DOPO clear_stack():
 * top → NULL
 * [A] → [B] → [C] → ??? ← MEMORIA PERSA! LEAK!
 */

// ✅ SOLUZIONE: Libera ogni nodo
void clear_stack() {
    while (top != NULL) {
        Node *temp = top;
        top = top->next;
        free(temp);
    }
}
```

---

## 7. 🔧 ESERCIZI

### 🟢 LIVELLO 1 - Comprensione

**Esercizio 1:** Data la sequenza di operazioni su uno stack vuoto:
```
push(5), push(10), pop(), push(15), push(20), pop(), pop()
```
Qual è lo stato finale dello stack?

<details>
<summary>Soluzione</summary>

```
Passo-passo:
push(5):     [5]
push(10):    [10, 5]
pop():       [5]        (rimuove 10)
push(15):    [15, 5]
push(20):    [20, 15, 5]
pop():       [15, 5]    (rimuove 20)
pop():       [5]        (rimuove 15)

STATO FINALE: [5]
top = 0
```
</details>

**Esercizio 2:** Data la sequenza su una queue vuota:
```
enqueue(A), enqueue(B), dequeue(), enqueue(C), dequeue()
```
Qual è lo stato finale?

<details>
<summary>Soluzione</summary>

```
enqueue(A):  [A]
enqueue(B):  [A, B]
dequeue():   [B]        (rimuove A)
enqueue(C):  [B, C]
dequeue():   [C]        (rimuove B)

STATO FINALE: [C]
front = rear = 0
```
</details>

### 🟡 LIVELLO 2 - Applicazione

**Esercizio 3:** Scrivi pseudocodice per invertire una stringa usando uno stack.

<details>
<summary>Soluzione</summary>

```
FUNZIONE inverti_stringa(str):
    stack = CREA_STACK()
    
    // Push tutti i caratteri
    PER ogni carattere c in str:
        stack.push(c)
    
    risultato = ""
    
    // Pop in ordine inverso
    MENTRE NON stack.isEmpty():
        risultato += stack.pop()
    
    RETURN risultato

ESEMPIO:
Input: "HELLO"
1. Push: H, E, L, L, O
   Stack: [O, L, L, E, H]
2. Pop: O, L, L, E, H
   Output: "OLLEH" ✓

Complessità: O(n) tempo, O(n) spazio
```
</details>

### 🟠 LIVELLO 3 - Analisi

**Esercizio 4:** Implementa una funzione che verifica se le parentesi sono bilanciate.

<details>
<summary>Soluzione</summary>

```
FUNZIONE parentesi_bilanciate(espressione):
    stack = CREA_STACK()
    
    PER ogni carattere c in espressione:
        SE c == '(' O c == '[' O c == '{':
            stack.push(c)
        
        ALTRIMENTI SE c == ')' O c == ']' O c == '}':
            SE stack.isEmpty():
                RETURN False  // Chiusura senza apertura
            
            apertura = stack.pop()
            
            // Verifica corrispondenza
            SE (c == ')' E apertura != '(') O
               (c == ']' E apertura != '[') O
               (c == '}' E apertura != '{'):
                RETURN False  // Mismatch
    
    // Alla fine, stack deve essere vuoto
    RETURN stack.isEmpty()

ESEMPI:
"(())"      → True  ✓
"([{}])"    → True  ✓
"(()"       → False ✗ (stack non vuoto alla fine)
"())"       → False ✗ (pop su stack vuoto)
"([)]"      → False ✗ (mismatch: [ con ))
```
</details>

### 🔴 LIVELLO 4 - Sintesi

**Esercizio 5:** Implementa una queue usando DUE stack.

<details>
<summary>Soluzione</summary>

```
IDEA: Usa due stack: inbox e outbox

STRUTTURA QueueWith2Stacks:
    inbox = STACK()   // Per enqueue
    outbox = STACK()  // Per dequeue

FUNZIONE enqueue(x):
    inbox.push(x)
    // O(1)

FUNZIONE dequeue():
    // Se outbox vuoto, trasferisci da inbox
    SE outbox.isEmpty():
        MENTRE NON inbox.isEmpty():
            outbox.push(inbox.pop())
    
    SE outbox.isEmpty():
        ERRORE: "Queue vuota!"
    
    RETURN outbox.pop()
    // O(1) ammortizzato

ESEMPIO:
enqueue(1), enqueue(2), enqueue(3), dequeue()

1. enqueue(1): inbox=[1], outbox=[]
2. enqueue(2): inbox=[2,1], outbox=[]
3. enqueue(3): inbox=[3,2,1], outbox=[]
4. dequeue():
   - outbox vuoto → trasferisci
   - inbox.pop(): 3 → outbox=[3]
   - inbox.pop(): 2 → outbox=[2,3]
   - inbox.pop(): 1 → outbox=[1,2,3]
   - inbox=[], outbox=[1,2,3]
   - outbox.pop() → ritorna 1 ✓

COMPLESSITÀ:
- enqueue: O(1)
- dequeue: O(1) ammortizzato
  (ogni elemento viene spostato max 1 volta)
```
</details>

---

## 8. 🎓 PUNTI CHIAVE

**STACK (LIFO):**
- ✅ Accesso solo al top
- ✅ push/pop in O(1)
- ✅ Ideale per: undo/redo, backtracking, call stack
- ⚠️ Rischio: overflow (stack pieno), underflow (stack vuoto)

**QUEUE (FIFO):**
- ✅ Accesso a front e rear
- ✅ enqueue/dequeue in O(1)
- ✅ Ideale per: scheduling, buffering, BFS
- ⚠️ Con array: usa circular queue per evitare spreco spazio

**REGOLA D'ORO:**
> "Ultimo arrivato, primo servito" → **STACK**
> "Primo arrivato, primo servito" → **QUEUE**

---

## 9. DEBUG: Problemi Comuni

### Problema 1: Stack Overflow in Ricorsione

```
SINTOMO: "Segmentation fault" o "Stack overflow"

CAUSA:
Ricorsione troppo profonda esaurisce lo stack

ESEMPIO:
void ricorsione_infinita(int n) {
    ricorsione_infinita(n + 1);  // Nessun caso base!
}

OGNI CHIAMATA aggiunge un frame allo stack:
┌────────┐
│ frame n│
├────────┤
│ ...    │
├────────┤
│ frame 2│
├────────┤
│ frame 1│
└────────┘
↓ OVERFLOW quando stack pieno!

DEBUG:
1. Assicurati di avere un CASO BASE
2. Limita la profondità ricorsiva
3. Usa -fstack-usage (GCC) per vedere uso stack
```

### Problema 2: Queue "Piena" ma con Spazi Vuoti

```
SINTOMO: isFull() ritorna true ma ci sono slot vuoti all'inizio

CAUSA: Queue lineare (non circolare)

SOLUZIONE: Usa circular queue con operatore modulo %
```

---

## 📚 Prossimi Passi

Ora che comprendi stack e queue, sei pronto per:
- [Lezione 7: Alberi](07_alberi.md) - Strutture dati gerarchiche
- [Lezione 8: Hash Table](08_hash_table.md) - Accesso O(1) con chiavi

---

[← Array e Liste](05_array_liste.md) | [Torna al Modulo](README.md) | [Alberi →](07_alberi.md)
