# Lezione 7 - Strutture Dati: Alberi

## 🎯 Obiettivi della Lezione

Alla fine di questa lezione sarai in grado di:
- Comprendere le strutture dati gerarchiche (alberi)
- Visualizzare alberi binari e alberi binari di ricerca
- Eseguire attraversamenti (in-order, pre-order, post-order)
- Analizzare la complessità delle operazioni sugli alberi
- Riconoscere quando usare alberi invece di liste o array

---

## 1. PROBLEMA: Limiti delle Strutture Lineari

### Scenario: Ricerca in Array Ordinato

```
Array ordinato con 1000 elementi:
[1, 3, 5, 7, 9, 11, 13, ..., 1999]

Cercare il numero 847:
- Ricerca lineare: O(n) → fino a 1000 confronti ⚠️
- Ricerca binaria: O(log n) → max 10 confronti ✓

Ricerca binaria è MOLTO più veloce!
Ma richiede array ORDINATO.
```

**DOMANDA:** Possiamo avere ricerca O(log n) anche con inserimenti/rimozioni frequenti?

**RISPOSTA:** SÌ! Con gli **Alberi Binari di Ricerca (BST)**!

---

## 2. ALBERI: Strutture Gerarchiche

### 2.1 Cos'è un Albero?

Un **albero** è una struttura dati gerarchica composta da **nodi** collegati da **archi**.

```
TERMINOLOGIA:

           A  ← RADICE (root)
          / \
         B   C  ← FIGLI di A
        / \   \
       D   E   F  ← FOGLIE (leaf nodes - nodi senza figli)
      /
     G

DEFINIZIONI:
- RADICE: Nodo senza genitore (A)
- FOGLIA: Nodo senza figli (E, F, G)
- NODO INTERNO: Nodo con almeno un figlio (A, B, C, D)
- ARCO: Connessione tra due nodi
- PERCORSO: Sequenza di nodi connessi (es: A → B → D → G)
- ALTEZZA: Lunghezza del percorso più lungo dalla radice a una foglia (3)
- LIVELLO: Distanza dalla radice (A=0, B/C=1, D/E/F=2, G=3)
- SOTTOALBERO: Albero radicato in un nodo (es: sottoalbero di B contiene D, E, G)
```

### 2.2 Proprietà degli Alberi

```
PROPRIETA FONDAMENTALI:
✓ Un albero con n nodi ha esattamente n-1 archi
✓ Esiste ESATTAMENTE un percorso tra qualsiasi coppia di nodi
✓ Se rimuovi un arco, l'albero si divide in due alberi disgiunti
✓ Non ci sono cicli (albero = grafo aciclico connesso)
```

---

## 3. ALBERI BINARI

### 3.1 Definizione

Un **albero binario** è un albero in cui ogni nodo ha **al massimo 2 figli**: sinistro e destro.

```
STRUTTURA NODO:
┌─────────────────────────┐
│ left │ data │ right     │
│  •   │  10  │   •       │
└──┼───┴──────┴────┼──────┘
   │               │
   ↓               ↓
Figlio SX      Figlio DX

ESEMPIO ALBERO BINARIO:
        10
       /  \
      5    15
     / \     \
    3   7    20
```

### 3.2 Tipi di Alberi Binari

**ALBERO BINARIO COMPLETO** (Complete Binary Tree):
```
Tutti i livelli sono completamente riempiti, tranne eventualmente l'ultimo
che è riempito da sinistra.

        10
       /  \
      5    15      ← Livello 1: pieno
     / \   / \     ← Livello 2: pieno
    3   7 12  20   ← Livello 3: riempito da SX

✓ COMPLETO
```

**ALBERO BINARIO PIENO** (Full Binary Tree):
```
Ogni nodo ha 0 o 2 figli (mai 1 solo figlio).

        10
       /  \
      5    15
     / \        ← 5 ha 2 figli, 15 ha 0 figli
    3   7

✓ PIENO
```

**ALBERO BINARIO PERFETTO** (Perfect Binary Tree):
```
Tutti i livelli sono completamente riempiti.

        10
       /  \
      5    15
     / \   / \
    3   7 12  20

✓ PERFETTO (anche completo e pieno!)
```

**ALBERO BINARIO SBILANCIATO** (Skewed Tree):
```
Degenera in una lista (peggiore caso).

    10
      \
       15
         \
          20
            \
             25

✗ SBILANCIATO - performance degradate!
```

### 3.3 Visualizzazione in Memoria

```
ALBERO:
    10
   /  \
  5    15

MEMORIA (nodi nell'heap):

┌─ Nodo @ 0x2000 ──────┐
│ left:  0x3000  ──────┼───┐
│ data:  10            │   │
│ right: 0x4000  ──────┼─┐ │
└──────────────────────┘ │ │
                         │ │
┌─ Nodo @ 0x3000 ──────┐ │ │
│ left:  NULL          │◄┘ │
│ data:  5             │   │
│ right: NULL          │   │
└──────────────────────┘   │
                           │
┌─ Nodo @ 0x4000 ──────┐   │
│ left:  NULL          │◄──┘
│ data:  15            │
│ right: NULL          │
└──────────────────────┘

CARATTERISTICHE:
✗ Memoria NON contigua (come liste)
✓ Navigazione tramite puntatori
✓ Struttura gerarchica
```

---

## 4. ALBERI BINARI DI RICERCA (BST)

### 4.1 Definizione

Un **Binary Search Tree** è un albero binario con la proprietà:
> Per ogni nodo X:
> - Tutti i nodi nel sottoalbero SINISTRO hanno valori < X
> - Tutti i nodi nel sottoalbero DESTRO hanno valori > X

```
ESEMPIO BST:
        50
       /  \
      30   70
     / \   / \
   20  40 60  80

VERIFICA:
- 50: SX (30,20,40) < 50 < DX (70,60,80) ✓
- 30: SX (20) < 30 < DX (40) ✓
- 70: SX (60) < 70 < DX (80) ✓
Tutti rispettano la proprietà BST!

NON-BST ESEMPIO:
        50
       /  \
      70   30  ← ERRORE! 70 > 50 ma è a sinistra!
```

### 4.2 Operazioni BST

#### RICERCA: O(log n) nel caso medio, O(n) nel peggiore

```
RICERCA di 40 nell'albero:

        50
       /  \
      30   70
     / \   / \
   20  40 60  80

ALGORITMO:
1. Parti dalla radice (50)
   40 < 50 → VAI A SINISTRA

2. Nodo corrente = 30
   40 > 30 → VAI A DESTRA

3. Nodo corrente = 40
   40 == 40 → TROVATO! ✓

PSEUDOCODICE:
FUNZIONE cerca(nodo, valore):
    SE nodo == NULL:
        RETURN NULL  // Non trovato
    
    SE valore == nodo.data:
        RETURN nodo  // Trovato!
    
    SE valore < nodo.data:
        RETURN cerca(nodo.left, valore)  // Cerca a sinistra
    ALTRIMENTI:
        RETURN cerca(nodo.right, valore)  // Cerca a destra

COMPLESSITÀ:
- Caso migliore: O(1) - radice
- Caso medio: O(log n) - albero bilanciato
- Caso peggiore: O(n) - albero sbilanciato (lista)
```

#### INSERIMENTO: O(log n) medio

```
INSERIMENTO di 65:

        50
       /  \
      30   70
     / \   / \
   20  40 60  80

ALGORITMO:
1. 65 < 50? NO → vai a DX
2. 65 < 70? SÌ → vai a SX
3. 65 < 60? NO → vai a DX
4. 60.right è NULL → INSERISCI QUI!

RISULTATO:
        50
       /  \
      30   70
     / \   / \
   20  40 60  80
            \
            65  ← NUOVO NODO

PSEUDOCODICE:
FUNZIONE inserisci(nodo, valore):
    SE nodo == NULL:
        RETURN crea_nodo(valore)
    
    SE valore < nodo.data:
        nodo.left = inserisci(nodo.left, valore)
    ALTRIMENTI SE valore > nodo.data:
        nodo.right = inserisci(nodo.right, valore)
    // Se valore == nodo.data, non inseriamo duplicati
    
    RETURN nodo
```

#### RIMOZIONE: O(log n) medio (caso più complesso!)

```
TRE CASI:

CASO 1: Nodo FOGLIA (nessun figlio)
Rimuovi 20:
        50               50
       /  \             /  \
      30   70    →    30   70
     / \   / \         \   / \
   20  40 60  80       40 60  80

Soluzione: Semplicemente elimina il nodo.

CASO 2: Nodo con UN FIGLIO
Rimuovi 30 (ha solo figlio destro 40):
        50               50
       /  \             /  \
      30   70    →    40   70
       \   / \             / \
       40 60  80          60  80

Soluzione: Sostituisci il nodo con suo figlio.

CASO 3: Nodo con DUE FIGLI (più complesso!)
Rimuovi 50:
        50
       /  \
      30   70
     / \   / \
   20  40 60  80

STRATEGIA: Sostituisci con il SUCCESSORE IN-ORDER
(il più piccolo nel sottoalbero destro)

1. Trova successore: 60 (min nel sottoalbero DX)
2. Copia valore 60 in nodo 50
3. Rimuovi nodo 60 originale (caso 1 o 2)

RISULTATO:
        60
       /  \
      30   70
     / \     \
   20  40    80
```

---

## 5. ATTRAVERSAMENTI (Traversals)

Gli **attraversamenti** definiscono l'ordine in cui visitiamo i nodi.

### 5.1 In-Order (Simmetrico)

**ORDINE:** Sinistra → Radice → Destra

```
ALBERO:
        50
       /  \
      30   70
     / \   / \
   20  40 60  80

IN-ORDER:
1. Visita sottoalbero SX di 50
   1.1. Visita sottoalbero SX di 30
        1.1.1. Visita 20 → OUTPUT: 20
   1.2. Visita 30 → OUTPUT: 30
   1.3. Visita sottoalbero DX di 30
        1.3.1. Visita 40 → OUTPUT: 40
2. Visita 50 → OUTPUT: 50
3. Visita sottoalbero DX di 50
   ... (stesso processo)

OUTPUT: 20, 30, 40, 50, 60, 70, 80

✓ Per BST: produce sequenza ORDINATA!

PSEUDOCODICE:
FUNZIONE inOrder(nodo):
    SE nodo != NULL:
        inOrder(nodo.left)
        STAMPA(nodo.data)
        inOrder(nodo.right)
```

### 5.2 Pre-Order (Anticipato)

**ORDINE:** Radice → Sinistra → Destra

```
STESSO ALBERO:
        50
       /  \
      30   70
     / \   / \
   20  40 60  80

PRE-ORDER:
1. Visita 50 → OUTPUT: 50
2. Visita sottoalbero SX
   2.1. Visita 30 → OUTPUT: 30
   2.2. Visita sottoalbero SX
        2.2.1. Visita 20 → OUTPUT: 20
   2.3. Visita sottoalbero DX
        2.3.1. Visita 40 → OUTPUT: 40
3. Visita sottoalbero DX
   ...

OUTPUT: 50, 30, 20, 40, 70, 60, 80

✓ Utile per COPIARE l'albero (crei radice prima dei figli)

PSEUDOCODICE:
FUNZIONE preOrder(nodo):
    SE nodo != NULL:
        STAMPA(nodo.data)
        preOrder(nodo.left)
        preOrder(nodo.right)
```

### 5.3 Post-Order (Posticipato)

**ORDINE:** Sinistra → Destra → Radice

```
STESSO ALBERO:
        50
       /  \
      30   70
     / \   / \
   20  40 60  80

POST-ORDER:
1. Visita sottoalbero SX
   1.1. Visita sottoalbero SX
        1.1.1. Visita 20 → OUTPUT: 20
   1.2. Visita sottoalbero DX
        1.2.1. Visita 40 → OUTPUT: 40
   1.3. Visita 30 → OUTPUT: 30
2. Visita sottoalbero DX
   ...
3. Visita 50 → OUTPUT: 50

OUTPUT: 20, 40, 30, 60, 80, 70, 50

✓ Utile per ELIMINARE l'albero (elimini figli prima della radice)

PSEUDOCODICE:
FUNZIONE postOrder(nodo):
    SE nodo != NULL:
        postOrder(nodo.left)
        postOrder(nodo.right)
        STAMPA(nodo.data)
```

### 5.4 Level-Order (Per Livelli)

**ORDINE:** Livello per livello, da sinistra a destra

```
STESSO ALBERO:
        50         ← Livello 0
       /  \
      30   70      ← Livello 1
     / \   / \
   20  40 60  80   ← Livello 2

LEVEL-ORDER:
Livello 0: 50
Livello 1: 30, 70
Livello 2: 20, 40, 60, 80

OUTPUT: 50, 30, 70, 20, 40, 60, 80

✓ Usa una QUEUE (BFS - Breadth-First Search)

PSEUDOCODICE:
FUNZIONE levelOrder(radice):
    SE radice == NULL:
        RETURN
    
    queue = CREA_QUEUE()
    queue.enqueue(radice)
    
    MENTRE NON queue.isEmpty():
        nodo = queue.dequeue()
        STAMPA(nodo.data)
        
        SE nodo.left != NULL:
            queue.enqueue(nodo.left)
        SE nodo.right != NULL:
            queue.enqueue(nodo.right)
```

---

## 6. CONFRONTO: Array vs BST

```
┌─────────────────┬─────────────────┬──────────────────────┐
│   OPERAZIONE    │ ARRAY ORDINATO  │   BST BILANCIATO     │
├─────────────────┼─────────────────┼──────────────────────┤
│ Ricerca         │ O(log n) binaria│ O(log n)             │
│ Inserimento     │ O(n) (shift)    │ O(log n) ⚡          │
│ Rimozione       │ O(n) (shift)    │ O(log n) ⚡          │
│ Min/Max         │ O(1)            │ O(log n)             │
│ Successore      │ O(1)            │ O(log n)             │
│ Ordinamento     │ Già ordinato    │ O(n) - in-order      │
│ Memoria         │ Contigua        │ Sparsa + overhead ptr│
└─────────────────┴─────────────────┴──────────────────────┘

🎯 USA ARRAY QUANDO:
  ✅ Pochi inserimenti/rimozioni
  ✅ Accesso per indice frequente
  ✅ Dati già ordinati

🎯 USA BST QUANDO:
  ✅ Molti inserimenti/rimozioni
  ✅ Ricerche frequenti
  ✅ Dimensione dinamica
```

---

## 7. ⚠️ ERRORI MORTALI

### ERRORE #1: BST Sbilanciato

```
// ❌ DISASTRO: Inserimenti in ordine crescente

Inserisci: 10, 20, 30, 40, 50

RISULTATO:
10
  \
   20
     \
      30
        \
         40
           \
            50

ALBERO DEGENERA IN LISTA!
- Ricerca: O(n) ✗ (non più O(log n))
- Altezza: n ✗ (dovrebbe essere log n)

/* CAUSA:
 * BST NON garantisce bilanciamento automatico.
 * Sequenze ordinate creano alberi sbilanciati.
 */

// ✅ SOLUZIONE: Usa alberi auto-bilancianti
AVL Tree, Red-Black Tree mantengono altezza O(log n)
(argomento avanzato, oltre questo corso)
```

### ERRORE #2: Violazione Proprietà BST

```
// ❌ ERRORE: Inserimento senza rispettare l'ordine

ALBERO:
    50
   /  \
  30   70

INSERIMENTO ERRATO di 40:
    50
   /  \
  30   70
 /
40  ← ERRORE! 40 > 30, dovrebbe essere a DESTRA!

/* CONSEGUENZA:
 * Ricerca NON funziona correttamente
 * Algoritmo assume proprietà BST rispettata
 */

// ✅ SOLUZIONE: Segui SEMPRE l'algoritmo di inserimento BST
```

### ERRORE #3: Memory Leak

```
// ❌ DISASTRO: Eliminare albero senza liberare nodi

void delete_tree(Node *root) {
    root = NULL;  // Perdi riferimento a TUTTO l'albero!
}

/* MEMORIA PRIMA:
 *     root → [50] → ...
 *            /   \
 *          [30] [70]
 * 
 * MEMORIA DOPO delete_tree():
 * root → NULL
 * [50], [30], [70] ... ← TUTTI I NODI PERSI! LEAK!
 */

// ✅ SOLUZIONE: Usa post-order (elimina figli prima della radice)
void delete_tree(Node *root) {
    if (root == NULL) return;
    
    delete_tree(root->left);   // Elimina sottoalbero SX
    delete_tree(root->right);  // Elimina sottoalbero DX
    free(root);                // Poi elimina radice
}
```

### ERRORE #4: Ricorsione Infinita

```
// ❌ ERRORE: Nessun caso base

void inOrder(Node *node) {
    inOrder(node->left);   // ← Cosa se node è NULL?
    printf("%d ", node->data);
    inOrder(node->right);
}

/* SE node == NULL:
 * node->left causa SEGMENTATION FAULT!
 */

// ✅ SOLUZIONE: Controlla sempre NULL (caso base!)
void inOrder(Node *node) {
    if (node == NULL) return;  // ← FONDAMENTALE!
    
    inOrder(node->left);
    printf("%d ", node->data);
    inOrder(node->right);
}
```

---

## 8. 🔧 ESERCIZI

### 🟢 LIVELLO 1 - Comprensione

**Esercizio 1:** Dato il BST, determina l'output degli attraversamenti:
```
    20
   /  \
  10   30
   \
    15
```

<details>
<summary>Soluzione</summary>

```
In-Order:   10, 15, 20, 30  (ORDINATO)
Pre-Order:  20, 10, 15, 30  (radice prima)
Post-Order: 15, 10, 30, 20  (radice ultima)
Level-Order: 20, 10, 30, 15 (per livelli)
```
</details>

**Esercizio 2:** È un BST valido?
```
    15
   /  \
  10   20
   \   /
   12 18
```

<details>
<summary>Soluzione</summary>

```
SÌ ✓

Verifica:
- 15: SX (10,12) < 15 < DX (20,18) ✓
- 10: SX (nessuno) < 10 < DX (12) ✓
- 20: SX (18) < 20 < DX (nessuno) ✓

ATTENZIONE: Non basta che ogni nodo abbia SX < nodo < DX.
Devi verificare che TUTTI i nodi nel sottoalbero SX siano < radice!
```
</details>

### 🟡 LIVELLO 2 - Applicazione

**Esercizio 3:** Inserisci 25 nel BST:
```
    20
   /  \
  10   30
       /
      28
```

<details>
<summary>Soluzione</summary>

```
PASSI:
1. 25 < 20? NO → vai a DX (30)
2. 25 < 30? SÌ → vai a SX (28)
3. 25 < 28? SÌ → vai a SX (NULL)
4. Inserisci 25 come figlio SX di 28

RISULTATO:
    20
   /  \
  10   30
       /
      28
      /
     25
```
</details>

### 🟠 LIVELLO 3 - Analisi

**Esercizio 4:** Scrivi pseudocodice per trovare il MINIMO in un BST.

<details>
<summary>Soluzione</summary>

```
FUNZIONE trova_minimo(nodo):
    SE nodo == NULL:
        RETURN NULL
    
    // Il minimo è il nodo più a SINISTRA
    MENTRE nodo.left != NULL:
        nodo = nodo.left
    
    RETURN nodo

COMPLESSITÀ: O(h) dove h è l'altezza
- Albero bilanciato: O(log n)
- Albero sbilanciato: O(n)

ESEMPIO:
    50
   /  \
  30   70
 /
20
 \
  25

trova_minimo() → segue sempre SX → 20
```
</details>

### 🔴 LIVELLO 4 - Sintesi

**Esercizio 5:** Implementa una funzione che conta i nodi foglia in un albero.

<details>
<summary>Soluzione</summary>

```
FUNZIONE conta_foglie(nodo):
    // Caso base 1: albero vuoto
    SE nodo == NULL:
        RETURN 0
    
    // Caso base 2: nodo foglia (nessun figlio)
    SE nodo.left == NULL E nodo.right == NULL:
        RETURN 1
    
    // Caso ricorsivo: somma foglie nei sottoalberi
    foglie_sx = conta_foglie(nodo.left)
    foglie_dx = conta_foglie(nodo.right)
    
    RETURN foglie_sx + foglie_dx

ESEMPIO:
        50
       /  \
      30   70
     /      \
   20       80

conta_foglie(50):
  conta_foglie(30):
    conta_foglie(20): FOGLIA → 1
    conta_foglie(NULL): 0
    RETURN 1 + 0 = 1
  conta_foglie(70):
    conta_foglie(NULL): 0
    conta_foglie(80): FOGLIA → 1
    RETURN 0 + 1 = 1
  RETURN 1 + 1 = 2

FOGLIE: 20, 80 → OUTPUT: 2 ✓
```
</details>

---

## 9. 🎓 PUNTI CHIAVE

**ALBERI:**
- ✅ Strutture gerarchiche (non lineari)
- ✅ Altezza ottimale: O(log n) per n nodi
- ✅ Navigazione tramite ricorsione naturale

**BST (Binary Search Tree):**
- ✅ Ricerca/Inserimento/Rimozione: O(log n) se bilanciato
- ✅ In-order traversal → sequenza ordinata
- ⚠️ Può degenerare in lista se sbilanciato: O(n)

**ATTRAVERSAMENTI:**
- In-Order: SX → Radice → DX (ordinato per BST)
- Pre-Order: Radice → SX → DX (copia albero)
- Post-Order: SX → DX → Radice (elimina albero)
- Level-Order: Livello per livello (usa queue)

---

## 10. DEBUG: Problemi Comuni

### Problema 1: Stack Overflow in Ricorsione

```
SINTOMO: "Stack overflow" su alberi grandi

CAUSA: Albero molto profondo, troppe chiamate ricorsive

SOLUZIONE:
1. Usa iterazione con stack esplicito
2. Bilancia l'albero (AVL, Red-Black)
3. Limita profondità ricorsione
```

### Problema 2: Albero Sbilanciato

```
SINTOMO: Performance degradate (O(n) invece di O(log n))

CAUSA: Inserimenti in ordine causano sbilanciamento

DEBUG:
1. Calcola altezza: se >> log n, albero sbilanciato
2. Usa AVL/Red-Black tree (auto-bilancianti)
3. Randomizza ordine inserimenti (workaround temporaneo)
```

---

## 📚 Prossimi Passi

Ora che comprendi gli alberi, sei pronto per:
- [Lezione 8: Hash Table](08_hash_table.md) - Accesso O(1) con chiavi
- [Lezione 9: Ricorsione](09_ricorsione.md) - Padroneggiare la ricorsione

---

[← Stack e Queue](06_stack_queue.md) | [Torna al Modulo](README.md) | [Hash Table →](08_hash_table.md)
