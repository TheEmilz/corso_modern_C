# Lezione 1: Pensiero Computazionale

## Cos'è un Algoritmo?

Un **algoritmo** è una sequenza finita di istruzioni ben definite che risolvono un problema o completano un compito. Pensa agli algoritmi come "ricette" per il computer.

### Caratteristiche di un Algoritmo

1. **Finito**: Deve terminare dopo un numero finito di passi
2. **Definito**: Ogni passo deve essere chiaro e non ambiguo
3. **Input**: Accetta zero o più input
4. **Output**: Produce almeno un output
5. **Efficace**: Ogni istruzione deve essere eseguibile

### Esempio: Preparare un Tè (Algoritmo della Vita Reale)

```
ALGORITMO PreparaTè
INPUT: Acqua, bustina di tè, tazza, zucchero (opzionale)
OUTPUT: Tazza di tè pronta

PASSI:
1. Riempi il bollitore con acqua
2. Accendi il bollitore
3. MENTRE l'acqua non bolle:
   - Aspetta
4. Versa l'acqua nella tazza
5. Metti la bustina di tè nella tazza
6. Aspetta 3-5 minuti
7. Rimuovi la bustina
8. SE vuoi zucchero:
   - Aggiungi zucchero
   - Mescola
9. FINE
```

**Nota**: Anche in un compito semplice come fare il tè, vediamo:
- Sequenza di passi
- Condizioni (SE vuoi zucchero)
- Cicli (MENTRE l'acqua non bolle)

## Decomposizione del Problema

La **decomposizione** è il processo di suddividere un problema complesso in parti più piccole e gestibili.

### Esempio: Organizzare una Festa

**Problema Complesso**: Organizzare una festa di compleanno

**Decomposizione**:
```
1. Pianificazione
   - Scegliere la data
   - Decidere il budget
   - Fare lista invitati

2. Inviti
   - Creare inviti
   - Inviare inviti
   - Tracciare RSVP

3. Location
   - Scegliere il luogo
   - Prenotare
   - Preparare decorazioni

4. Cibo e Bevande
   - Pianificare menu
   - Fare shopping
   - Preparare cibo

5. Intrattenimento
   - Musica
   - Giochi
   - Attività
```

Ogni sottoproblema può essere ulteriormente scomposto fino a diventare un compito semplice.

### Esempio Computazionale: Trovare il Massimo in una Lista

**Problema**: Trovare il numero più grande in una lista di numeri

**Decomposizione**:
```
1. Inizializzazione
   - Assumere che il primo numero sia il massimo

2. Confronto
   - Per ogni numero rimanente nella lista:
     - Confrontare con il massimo corrente
     - Se è più grande, aggiornare il massimo

3. Risultato
   - Restituire il massimo trovato
```

**Pseudocodice**:
```
FUNZIONE TrovaMaximo(lista)
    SE lista è vuota:
        ERRORE "Lista vuota"
    
    massimo ← primo elemento della lista
    
    PER ogni elemento nella lista:
        SE elemento > massimo:
            massimo ← elemento
    
    RITORNA massimo
FINE
```

**Esempio di Esecuzione**:
```
Lista: [3, 7, 2, 9, 1, 5]

Passo 1: massimo = 3 (primo elemento)
Passo 2: confronta 7 con 3 → 7 > 3 → massimo = 7
Passo 3: confronta 2 con 7 → 2 non è > 7 → massimo = 7
Passo 4: confronta 9 con 7 → 9 > 7 → massimo = 9
Passo 5: confronta 1 con 9 → 1 non è > 9 → massimo = 9
Passo 6: confronta 5 con 9 → 5 non è > 9 → massimo = 9

Risultato: 9
```

## Riconoscimento di Pattern

Il **riconoscimento di pattern** è l'abilità di identificare somiglianze, differenze e regolarità in problemi diversi.

### Esempio 1: Sequenze Numeriche

**Sequenza 1**: 2, 4, 6, 8, 10, ...
- Pattern: Numeri pari
- Regola: aggiungi 2
- Prossimo numero: 12

**Sequenza 2**: 1, 1, 2, 3, 5, 8, ...
- Pattern: Fibonacci
- Regola: somma dei due precedenti
- Prossimo numero: 13

**Sequenza 3**: 1, 4, 9, 16, 25, ...
- Pattern: Quadrati perfetti
- Regola: n²
- Prossimo numero: 36

### Esempio 2: Pattern in Problemi di Programmazione

Molti problemi seguono pattern simili:

**Pattern "Trova l'Elemento"**:
- Trova il massimo
- Trova il minimo
- Trova un elemento specifico
- Conta elementi con proprietà X

**Struttura Comune**:
```
INIZIALIZZA variabile risultato
PER ogni elemento:
    APPLICA test/confronto
    SE condizione soddisfatta:
        AGGIORNA risultato
RITORNA risultato
```

**Pattern "Elabora Tutti gli Elementi"**:
- Somma tutti gli elementi
- Moltiplica tutti gli elementi
- Trasforma ogni elemento

**Struttura Comune**:
```
INIZIALIZZA accumulatore
PER ogni elemento:
    APPLICA operazione
    AGGIORNA accumulatore
RITORNA accumulatore
```

## Astrazione e Generalizzazione

L'**astrazione** è il processo di rimuovere dettagli non essenziali per concentrarsi su quelli importanti.

### Esempio: Calcolare la Media

**Problema Specifico**: Calcolare la media di 3 numeri
```
media = (a + b + c) / 3
```

**Astrazione**: Calcolare la media di N numeri
```
FUNZIONE CalcolaMedia(numeri)
    somma ← 0
    conteggio ← 0
    
    PER ogni numero in numeri:
        somma ← somma + numero
        conteggio ← conteggio + 1
    
    SE conteggio = 0:
        ERRORE "Nessun numero"
    
    media ← somma / conteggio
    RITORNA media
FINE
```

**Generalizzazione Ulteriore**: Calcolare una statistica generica
```
FUNZIONE CalcolaStatistica(numeri, operazione)
    accumulatore ← valore iniziale
    
    PER ogni numero in numeri:
        accumulatore ← operazione(accumulatore, numero)
    
    RITORNA valore finale
FINE
```

## Debug: Come Ragionare su un Problema

### Step 1: Comprendi il Problema

**Domande da Porsi**:
- Qual è l'input?
- Qual è l'output desiderato?
- Ci sono casi speciali?
- Quali sono i vincoli?

**Esempio**: Verificare se un numero è primo

```
INPUT: Un numero intero positivo n
OUTPUT: Vero se n è primo, Falso altrimenti

CASI SPECIALI:
- n = 1 → Non è primo
- n = 2 → È primo (unico numero primo pari)
- n < 0 → Errore o input non valido

VINCOLI:
- n deve essere intero
- Efficienza per numeri grandi
```

### Step 2: Pianifica la Soluzione

**Strategia "Divide et Impera"**:
1. Scomponi in sottoproblemi
2. Risolvi ogni sottoproblema
3. Combina le soluzioni

**Esempio**: Numero Primo
```
Sottoproblemi:
1. Gestire casi speciali (n ≤ 1, n = 2)
2. Per n > 2: verificare se ha divisori
3. Un numero è primo se non ha divisori tra 2 e √n
```

### Step 3: Scrivi Pseudocodice

```
FUNZIONE ÈPrimo(n)
    // Casi speciali
    SE n ≤ 1:
        RITORNA Falso
    SE n = 2:
        RITORNA Vero
    SE n % 2 = 0:
        RITORNA Falso  // Pari
    
    // Controlla divisori dispari da 3 a √n
    i ← 3
    MENTRE i * i ≤ n:
        SE n % i = 0:
            RITORNA Falso
        i ← i + 2
    
    RITORNA Vero
FINE
```

### Step 4: Testa con Esempi

**Test Case 1**: n = 7
```
n = 7
n > 1 ✓
n ≠ 2
n % 2 = 1 (dispari) ✓

Ciclo:
i = 3: 3*3 = 9 > 7 → esci dal ciclo

Risultato: Vero ✓
```

**Test Case 2**: n = 9
```
n = 9
n > 1 ✓
n ≠ 2
n % 2 = 1 (dispari) ✓

Ciclo:
i = 3: 3*3 = 9 ≤ 9 ✓
9 % 3 = 0 → divisibile!

Risultato: Falso ✓
```

**Test Case 3**: n = 2
```
n = 2
n > 1 ✓
n = 2 → caso speciale

Risultato: Vero ✓
```

### Step 5: Debug Passo-Passo

Quando qualcosa non funziona:

**Tecnica 1: Tracciamento**
- Scrivi i valori di ogni variabile a ogni passo
- Confronta con il comportamento atteso

**Tecnica 2: Casi Semplici**
- Inizia con input minimali
- Es: n = 2, n = 3 prima di testare n = 97

**Tecnica 3: Verifica Assunzioni**
- Controlla i tuoi presupposti
- "Perché penso che questo dovrebbe funzionare?"

## Esercizio Guidato: Conta Vocali

**Problema**: Conta quante vocali ci sono in una stringa

### Soluzione Passo-Passo

**Step 1: Comprensione**
```
INPUT: Una stringa di testo
OUTPUT: Numero di vocali
VOCALI: a, e, i, o, u (maiuscole e minuscole)
```

**Step 2: Decomposizione**
```
1. Inizializza contatore a 0
2. Per ogni carattere nella stringa:
   - Verifica se è una vocale
   - Se sì, incrementa contatore
3. Ritorna contatore
```

**Step 3: Pseudocodice**
```
FUNZIONE ContaVocali(testo)
    vocali ← ['a', 'e', 'i', 'o', 'u', 'A', 'E', 'I', 'O', 'U']
    contatore ← 0
    
    PER ogni carattere in testo:
        SE carattere è in vocali:
            contatore ← contatore + 1
    
    RITORNA contatore
FINE
```

**Step 4: Tracciamento**
```
Input: "Hello World"

Tracciamento:
H: non è vocale, contatore = 0
e: è vocale, contatore = 1
l: non è vocale, contatore = 1
l: non è vocale, contatore = 1
o: è vocale, contatore = 2
 : non è vocale, contatore = 2
W: non è vocale, contatore = 2
o: è vocale, contatore = 3
r: non è vocale, contatore = 3
l: non è vocale, contatore = 3
d: non è vocale, contatore = 3

Output: 3 ✓
```

## Esercizi Pratici

### Esercizio 1: Palindromo
Scrivi un algoritmo (pseudocodice) per verificare se una parola è un palindromo.

**Suggerimento**: Confronta caratteri dall'inizio e dalla fine.

### Esercizio 2: Somma Pari
Scrivi un algoritmo per sommare solo i numeri pari in una lista.

**Suggerimento**: Usa il pattern "Elabora Tutti gli Elementi" con una condizione.

### Esercizio 3: Secondo Massimo
Scrivi un algoritmo per trovare il secondo numero più grande in una lista.

**Suggerimento**: Mantieni traccia di due valori: massimo e secondo_massimo.

### Esercizio 4: Rimozione Duplicati (Concettuale)
Descrivi a parole un metodo per rimuovere elementi duplicati da una lista.

**Suggerimento**: Pensa a come verifichi se hai già visto un elemento.

## Punti Chiave

✅ Un algoritmo è una sequenza finita e ben definita di passi
✅ La decomposizione semplifica problemi complessi
✅ Il riconoscimento di pattern aiuta a riutilizzare soluzioni
✅ L'astrazione elimina dettagli non essenziali
✅ Il debug inizia con la comprensione del problema
✅ Testa sempre con casi semplici prima di quelli complessi

## Prossimi Passi

Nel prossimo modulo esploreremo la **complessità degli algoritmi** e come misurare l'efficienza delle nostre soluzioni.

---

[← Torna al Modulo](README.md) | [Prossimo: Complessità →](02_complessita.md)
