# Lezione 10: Metodologia di Debug

## Introduzione al Debug

Il **debugging** è l'arte di trovare e correggere errori nel codice. È una competenza fondamentale che separa i programmatori principianti da quelli esperti.

> "Debugging è come essere un detective in un giallo dove sei anche il criminale." - Filipe Fortes

## Il Processo Sistematico di Debug

### Le 5 Fasi del Debug

```
1. RIPRODUCI   → Fai accadere l'errore in modo consistente
2. RIDUCI      → Semplifica al caso minimo che mostra il problema
3. LOCALIZZA   → Identifica dove nel codice si verifica l'errore
4. CORREGGI    → Implementa la soluzione
5. VERIFICA    → Assicurati che il problema sia risolto
```

## Fase 1: Riproduci l'Errore

### Perché è Importante?

Non puoi risolvere un problema che non puoi osservare. Un bug riproducibile è un bug risolvibile.

### Domande da Porsi

- **Quando** si verifica l'errore?
- Con **quale input** si verifica?
- Si verifica **sempre** o solo a volte?
- **Cosa** succede esattamente? (Crash, output sbagliato, loop infinito?)

### Esempio: Bug Intermittente

**Problema Riportato**: "A volte il programma dà risultati sbagliati"

**Debug**:
```
Test 1: Input "abc" → Output corretto
Test 2: Input "xyz" → Output sbagliato!
Test 3: Input "xyz" → Output sbagliato!
Test 4: Input "XYZ" → Output corretto
Test 5: Input "xyz" → Output sbagliato!

PATTERN TROVATO: Minuscole "xyz" causano l'errore
```

### Tecniche per Riprodurre

1. **Salva l'input** che causa l'errore
2. **Registra** le condizioni (stato del sistema, altre variabili)
3. **Isola** da fattori esterni (rete, file, tempo)
4. **Automatizza** il test per ripetere facilmente

## Fase 2: Riduci al Caso Minimo

### Esempio di Riduzione

**Codice Originale (complesso)**:
```
FUNZIONE ProcessaDati(array, n, filtro, opzioni)
    risultato ← Array vuoto
    
    SE opzioni.modalità = "avanzata":
        PER ogni elemento in array:
            SE ApplicaFiltro(elemento, filtro):
                SE CondizioneComplessa(elemento, opzioni):
                    trasformato ← Trasforma(elemento, opzioni.parametri)
                    SE ValoreValido(trasformato):
                        Aggiungi trasformato a risultato
    ALTRIMENTI:
        // modalità semplice...
    
    RITORNA risultato
FINE
```

**Caso Minimo che Mostra il Bug**:
```
array ← [1, 2, 3]
filtro ← x > 0
opzioni ← {modalità: "avanzata", parametri: default}

Chiamata: ProcessaDati(array, 3, filtro, opzioni)
Errore: L'elemento 2 manca nell'output!
```

**Semplificato Ulteriormente**:
```
elemento ← 2
SE CondizioneComplessa(elemento, opzioni):  // Restituisce Falso!
    // Questo non viene eseguito
```

**Bug Trovato**: `CondizioneComplessa` ha un problema con il valore 2.

### Strategia "Divide et Impera"

1. Commenta metà del codice
2. Se il bug persiste → è nell'altra metà
3. Se il bug scompare → è nella parte commentata
4. Ripeti fino a trovare la linea esatta

## Fase 3: Localizza il Problema

### Tecnica 1: Print Debugging

La tecnica più semplice ma efficace: stampa i valori delle variabili.

**Esempio: Bug in Calcolo Media**

```
FUNZIONE CalcolaMedia(voti, n)
    somma ← 0
    
    PER i da 0 a n:              // BUG: dovrebbe essere n-1
        somma ← somma + voti[i]
    
    media ← somma / n
    RITORNA media
FINE
```

**Debug con Print**:
```
FUNZIONE CalcolaMedia(voti, n)
    Stampa "Inizio: n =", n
    somma ← 0
    
    PER i da 0 a n:
        Stampa "Iterazione", i, "voti[i] =", voti[i]
        somma ← somma + voti[i]
    
    Stampa "Somma finale:", somma
    media ← somma / n
    Stampa "Media:", media
    RITORNA media
FINE
```

**Output**:
```
Inizio: n = 3
Iterazione 0 voti[0] = 80
Iterazione 1 voti[1] = 90
Iterazione 2 voti[2] = 85
Iterazione 3 voti[3] = ERRORE! Indice fuori limiti!
```

**Bug Identificato**: Il loop va fino a `n` invece che `n-1`.

### Tecnica 2: Asserzioni

Le asserzioni sono controlli che verificano che certe condizioni siano vere.

**Esempio: Funzione di Divisione**

```
FUNZIONE Dividi(a, b)
    ASSERT b ≠ 0, "Divisione per zero!"
    RITORNA a / b
FINE
```

**Uso delle Asserzioni**:
- Verifica precondizioni (input valido)
- Verifica postcondizioni (output corretto)
- Verifica invarianti (condizioni che devono sempre essere vere)

**Esempio: Ricerca Binaria**

```
FUNZIONE RicercaBinaria(array, n, target)
    ASSERT array è ordinato, "Array deve essere ordinato!"
    
    sinistra ← 0
    destra ← n - 1
    
    MENTRE sinistra ≤ destra:
        ASSERT sinistra ≥ 0 AND destra < n, "Indici fuori limiti"
        
        medio ← (sinistra + destra) / 2
        
        ASSERT medio ≥ sinistra AND medio ≤ destra, "Medio invalido"
        
        // ... resto dell'algoritmo
    
    RITORNA -1
FINE
```

### Tecnica 3: Rubber Duck Debugging

Spiega il codice **linea per linea** a un oggetto inanimato (come un papero di gomma).

**Perché Funziona?**
- Ti costringe a verbalizzare ogni assunzione
- Rallenti e pensi più attentamente
- Spesso trovi l'errore mentre spieghi

**Esempio di Sessione**:
```
Programmatore: "Okay, papero, questa funzione calcola il fattoriale."
Programmatore: "Prima verifico se n è 0 o 1... aspetta, e se n è negativo?"
Programmatore: "Ah! Non gestisco i numeri negativi! Ecco il bug!"
```

### Tecnica 4: Confronto con Caso Funzionante

Confronta un input che funziona con uno che non funziona.

**Esempio: Palindromo**

```
Input "radar" → Funziona ✓
Input "hello" → Funziona ✓
Input "Radar" → Non funziona ✗

DIFFERENZA: Lettere maiuscole!
BUG: Non converto a minuscole prima del confronto
```

## Fase 4: Correggi il Problema

### Regola d'Oro: Comprendi Prima di Correggere

❌ **Non fare**:
- Cambiare codice a caso sperando che funzioni
- Aggiungere controlli senza capire il problema
- Copiare soluzioni senza comprenderle

✅ **Fai**:
- Comprendi la causa radice
- Pensa agli effetti collaterali
- Considera casi limite
- Testa la correzione

### Esempio: Correzione Prematura

**Problema**: La funzione restituisce risultati negativi inaspettati.

**Correzione Rapida (sbagliata)**:
```
FUNZIONE Calcola(x, y)
    risultato ← x - y
    
    // "Fix" rapido
    SE risultato < 0:
        risultato ← 0
    
    RITORNA risultato
FINE
```

**Problema**: Mascheri il vero bug! Forse il risultato negativo è corretto e il problema è altrove.

**Correzione Corretta**:
1. Comprendi perché si aspettano risultati positivi
2. Verifica se `x - y` negativo è valido
3. Identifica se il bug è nell'input o nella logica
4. Correggi la vera causa

## Fase 5: Verifica la Soluzione

### Test Multipli

Non testare solo il caso che era rotto!

**Suite di Test Completa**:
```
1. Caso che mostrava il bug → Deve funzionare ora
2. Casi normali → Devono ancora funzionare
3. Casi limite:
   - Input vuoto
   - Input massimo
   - Input minimo
   - Valori speciali (zero, negativo, etc.)
4. Casi edge:
   - Un elemento
   - Tutti elementi uguali
   - Elementi in ordine/disordine
```

### Esempio: Test per Ricerca

```
FUNZIONE TestRicerca()
    array ← [1, 3, 5, 7, 9]
    
    // Test casi normali
    ASSERT Cerca(array, 5, 5) = 2, "Elemento nel mezzo"
    ASSERT Cerca(array, 5, 1) = 0, "Primo elemento"
    ASSERT Cerca(array, 5, 9) = 4, "Ultimo elemento"
    
    // Test elemento non presente
    ASSERT Cerca(array, 5, 4) = -1, "Elemento non presente"
    
    // Test casi limite
    array_vuoto ← []
    ASSERT Cerca(array_vuoto, 0, 5) = -1, "Array vuoto"
    
    array_uno ← [7]
    ASSERT Cerca(array_uno, 1, 7) = 0, "Un elemento trovato"
    ASSERT Cerca(array_uno, 1, 5) = -1, "Un elemento non trovato"
    
    Stampa "Tutti i test passati!"
FINE
```

## Errori Comuni e Come Evitarli

### Errore 1: Off-by-One

**Problema**: Errore negli indici di array o limiti di loop.

**Esempio**:
```
// SBAGLIATO
PER i da 0 a n:           // Va oltre l'ultimo elemento!
    Stampa array[i]

// CORRETTO
PER i da 0 a n-1:         // Si ferma all'ultimo elemento
    Stampa array[i]
```

**Debug**:
- Stampa gli indici e verifica i limiti
- Disegna i casi con array piccoli (es: n=3)
- Controlla il primo e ultimo elemento

### Errore 2: Variabili Non Inizializzate

**Problema**: Uso di variabili senza valore iniziale.

**Esempio**:
```
// SBAGLIATO
INT somma           // Valore casuale!
PER i da 0 a n-1:
    somma ← somma + array[i]  // Somma incorretta

// CORRETTO
INT somma ← 0       // Inizializzato correttamente
PER i da 0 a n-1:
    somma ← somma + array[i]
```

**Debug**:
- Stampa le variabili subito dopo la dichiarazione
- Usa asserzioni per verificare stati iniziali

### Errore 3: Confronto vs Assegnamento

**Esempio**:
```
// SBAGLIATO (assegnamento invece di confronto)
SE x = 5:           // Assegna 5 a x!
    Stampa "x è 5"

// CORRETTO
SE x == 5:          // Confronta x con 5
    Stampa "x è 5"
```

### Errore 4: Logica Booleana Errata

**Esempio**:
```
// Voglio verificare se x è tra 1 e 10
// SBAGLIATO
SE x > 1 OR x < 10:     // Sempre vero!

// CORRETTO
SE x > 1 AND x < 10:    // Corretto
```

**Debug**: Testa con valori estremi (x=0, x=1, x=10, x=11).

### Errore 5: Divisione per Zero

**Esempio**:
```
// SBAGLIATO
media ← somma / contatore  // Cosa se contatore = 0?

// CORRETTO
SE contatore = 0:
    ERRORE "Nessun elemento"
ALTRIMENTI:
    media ← somma / contatore
```

### Errore 6: Overflow

**Esempio**:
```
INT piccolo ← 100
INT risultato ← piccolo * piccolo * piccolo  // 1,000,000

// Se INT max è 32767, overflow!
```

**Debug**:
- Conosci i limiti dei tuoi tipi di dati
- Usa tipi più grandi se necessario
- Controlla i valori intermedi

## Strategie Avanzate

### 1. Test-Driven Debugging (TDD)

Scrivi un test che fallisce, poi correggi il codice.

```
// Test che espone il bug
FUNZIONE TestBug()
    ASSERT CalcolaMedia([1, 2, 3], 3) = 2, "Bug trovato!"
FINE

// Ora correggi la funzione finché il test passa
```

### 2. Bisection Debugging

Identifica quando è stato introdotto il bug.

```
Commit 1: Funziona ✓
Commit 2: Funziona ✓
Commit 3: Funziona ✓
Commit 4: Non funziona ✗
Commit 5: Non funziona ✗

Bug introdotto tra commit 3 e 4!
```

### 3. Isolamento dei Componenti

Testa ogni parte separatamente.

```
FUNZIONE SistemaComplesso()
    dati ← LeggiDati()         // Test 1: Dati letti correttamente?
    filtrati ← Filtra(dati)    // Test 2: Filtro funziona?
    ordinati ← Ordina(filtrati)// Test 3: Ordinamento corretto?
    risultato ← Elabora(ordinati) // Test 4: Elaborazione ok?
    Salva(risultato)           // Test 5: Salvataggio ok?
FINE
```

## Caso di Studio: Debug Completo

### Problema: Funzione che Conta Parole

**Codice con Bug**:
```
FUNZIONE ContaParole(testo)
    contatore ← 0
    in_parola ← Falso
    
    PER ogni carattere in testo:
        SE carattere è lettera:
            SE NON in_parola:
                contatore ← contatore + 1
            in_parola ← Vero
        ALTRIMENTI:
            in_parola ← Falso
    
    RITORNA contatore
FINE
```

**Test Iniziali**:
```
ContaParole("hello world") → 2 ✓
ContaParole("one")         → 1 ✓
ContaParole("  spaces  ") → 1 ✗ (dovrebbe essere 1 ma...)
ContaParole("a b c")      → 3 ✓
```

### Passo 1: Riproduci

```
Input problematico: "  spaces  "
Output atteso: 1
Output ricevuto: 1
Aspetta, funziona?

Test più approfonditi:
ContaParole("")           → 0 ✓
ContaParole("   ")        → 0 ✓
ContaParole("hello   world") → 2 ✓

Nessun bug trovato ancora...

ContaParole("hello-world") → 2 ✗ (dovrebbe essere 1!)
Bug trovato!
```

### Passo 2: Riduci

```
Input minimo: "a-b"
Output atteso: 1 parola (con trattino)
Output ricevuto: 2 parole
```

### Passo 3: Localizza

```
Tracciamento per "a-b":

Carattere 'a': è lettera → NON in_parola → contatore=1, in_parola=Vero
Carattere '-': NON lettera → in_parola=Falso
Carattere 'b': è lettera → NON in_parola → contatore=2, in_parola=Vero

Problema identificato: Il trattino interrompe la parola!
```

### Passo 4: Comprendi

Il bug è nella condizione "carattere è lettera". I trattini fanno parte delle parole composte ma non sono lettere.

### Passo 5: Correggi

```
FUNZIONE ContaParole(testo)
    contatore ← 0
    in_parola ← Falso
    
    PER ogni carattere in testo:
        // Modifica: accetta lettere, numeri, trattini
        SE carattere è alfanumerico OR carattere = '-' OR carattere = ''':
            SE NON in_parola:
                contatore ← contatore + 1
            in_parola ← Vero
        ALTRIMENTI:
            in_parola ← Falso
    
    RITORNA contatore
FINE
```

### Passo 6: Verifica

```
ContaParole("hello-world")  → 1 ✓
ContaParole("it's")         → 1 ✓
ContaParole("hello world")  → 2 ✓
ContaParole("")             → 0 ✓
ContaParole("   ")          → 0 ✓
ContaParole("a b c")        → 3 ✓
ContaParole("test-case-2")  → 1 ✓

Tutti i test passano!
```

## Strumenti Mentali per il Debugging

### La Lista di Controllo del Debugger

Quando sei bloccato, chiedi:

1. **Assunzioni**: Quali assunzioni sto facendo?
2. **Input**: L'input è quello che penso?
3. **Output**: L'output è dove penso che sia il problema?
4. **Flusso**: Il codice sta eseguendo il percorso che penso?
5. **Stato**: Le variabili hanno i valori che mi aspetto?
6. **Limiti**: Ho considerato i casi limite?

### Quando Sei Completamente Bloccato

1. **Fai una pausa**: A volte la soluzione arriva quando non ci pensi
2. **Spiega a qualcuno**: Anche solo spiegare aiuta
3. **Riparti da zero**: Riscrivi il codice da capo
4. **Cerca aiuto**: Due paia di occhi vedono più di uno
5. **Dormi su di esso**: Il cervello lavora in background

## Esercizi Pratici

### Esercizio 1: Trova il Bug

```
FUNZIONE Fattoriale(n)
    risultato ← 1
    PER i da 1 a n:
        risultato ← risultato + i
    RITORNA risultato
FINE
```

**Domanda**: Cosa c'è di sbagliato?

<details>
<summary>Soluzione</summary>

Bug: Usa addizione (+) invece di moltiplicazione (*)

Corretto:
```
risultato ← risultato * i
```
</details>

### Esercizio 2: Debug con Trace

```
FUNZIONE MissingNumber(array, n)
    // Trova il numero mancante da 1 a n+1
    somma_attesa ← (n + 1) * (n + 2) / 2
    somma_reale ← 0
    
    PER i da 0 a n:
        somma_reale ← somma_reale + array[i]
    
    RITORNA somma_attesa - somma_reale
FINE
```

**Test**: `MissingNumber([1, 2, 4, 5], 4)` dovrebbe ritornare 3.

Traccia l'esecuzione passo-passo.

### Esercizio 3: Crea Test

Per questa funzione, crea una suite completa di test:

```
FUNZIONE IsPalindrome(str)
    // Verifica se str è un palindromo
    // ...
FINE
```

**Includi**:
- Casi normali
- Casi limite
- Casi edge con spazi, maiuscole, punteggiatura

## Punti Chiave

✅ Il debug è un processo sistematico, non casuale
✅ Riproduci sempre l'errore in modo consistente
✅ Riduci al caso minimo che mostra il bug
✅ Usa print debugging, asserzioni, rubber duck
✅ Comprendi il problema prima di correggere
✅ Testa ampiamente dopo ogni correzione
✅ Impara dagli errori comuni
✅ Usa una checklist quando sei bloccato

## Filosofia del Debug

> "Il debugging è due volte più difficile che scrivere il codice. Quindi, se scrivi il codice nel modo più intelligente possibile, per definizione, non sei abbastanza intelligente per debuggarlo." - Brian Kernighan

**Morale**: Scrivi codice semplice e chiaro, non complicato!

---

[← Ricorsione](09_ricorsione.md) | [Torna al Modulo](README.md) | [Prossimo: Esercizi →](11_esercizi.md)
