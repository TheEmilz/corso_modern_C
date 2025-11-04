# Guida per Contribuire

Grazie per il tuo interesse nel contribuire a questo corso di C! Ecco come puoi aiutare.

## Come Contribuire

### Tipi di Contributi Benvenuti

1. **Correzioni di Errori**
   - Typos e errori grammaticali
   - Errori di codice
   - Link non funzionanti

2. **Miglioramenti al Contenuto**
   - Spiegazioni più chiare
   - Esempi aggiuntivi
   - Diagrammi e visualizzazioni

3. **Nuovi Esercizi**
   - Esercizi pratici
   - Progetti esempio
   - Test cases

4. **Traduzioni**
   - Traduzione in altre lingue
   - Mantenimento versioni tradotte

## Processo di Contribuzione

1. **Fork il Repository**
   ```bash
   # Clicca il pulsante "Fork" su GitHub
   ```

2. **Clona il Fork**
   ```bash
   git clone https://github.com/TUO_USERNAME/corso_modern_C.git
   cd corso_modern_C
   ```

3. **Crea un Branch**
   ```bash
   git checkout -b feature/mio-miglioramento
   ```

4. **Fai le Modifiche**
   - Segui lo stile esistente
   - Testa tutti gli esempi di codice
   - Aggiungi commenti dove necessario

5. **Commit e Push**
   ```bash
   git add .
   git commit -m "Descrizione chiara delle modifiche"
   git push origin feature/mio-miglioramento
   ```

6. **Apri una Pull Request**
   - Vai al tuo fork su GitHub
   - Clicca "New Pull Request"
   - Descrivi le modifiche in dettaglio

## Linee Guida per il Codice

### Stile di Codifica

```c
/* Usa commenti per spiegare cosa fa il codice */

/* Nomi di funzioni: snake_case */
int calcola_somma(int a, int b);

/* Nomi di costanti: UPPER_CASE */
#define MAX_SIZE 100

/* Nomi di tipi: PascalCase */
typedef struct {
    int id;
    char name[50];
} Student;

/* Indentazione: 4 spazi (no tabs) */
if (condition) {
    // codice indentato
    printf("Hello\n");
}

/* Parentesi graffe su nuova riga per funzioni */
int main(void)
{
    // codice
    return 0;
}

/* Parentesi graffe sulla stessa riga per controllo di flusso */
if (x > 0) {
    // codice
}
```

### Compilazione e Test

Tutti gli esempi devono:
1. Compilare senza errori con GCC
2. Compilare senza warning con `-Wall -Wextra`
3. Non avere memory leaks (verificare con valgrind)
4. Includere commenti esplicativi

```bash
# Compila con warnings
gcc -Wall -Wextra -std=c11 esempio.c -o esempio

# Verifica memory leaks
valgrind --leak-check=full ./esempio
```

## Linee Guida per la Documentazione

### Struttura dei Moduli

Ogni modulo dovrebbe avere:
- Introduzione chiara
- Spiegazione dei concetti
- Esempi di codice completi e funzionanti
- Esercizi pratici
- Link al modulo precedente e successivo

### Esempi di Codice

```c
/* BUONO: esempio completo e auto-contenuto */
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    return 0;
}

/* EVITA: snippet incompleti senza contesto */
printf("Hello");  // Manca include, main, etc.
```

### Spiegazioni

- Usa linguaggio chiaro e semplice
- Spiega il "perché", non solo il "come"
- Fornisci esempi pratici
- Confronta con approcci alternativi quando utile

## Priorità dei Contributi

Alta priorità:
- 🐛 Correzioni di bug nel codice
- 📚 Miglioramenti alla chiarezza delle spiegazioni
- ✅ Aggiunta di test e validazioni

Media priorità:
- ➕ Nuovi esempi pratici
- 🔧 Ottimizzazioni del codice
- 🌐 Traduzioni

Bassa priorità:
- 🎨 Formattazione e stile
- 📝 Typos minori

## Segnalare Problemi

Quando apri un issue, includi:

1. **Descrizione chiara del problema**
   - Cosa ti aspettavi
   - Cosa è successo invece

2. **Passi per riprodurre** (se applicabile)
   ```bash
   gcc esempio.c -o esempio
   ./esempio
   ```

3. **Informazioni sull'ambiente**
   - Sistema operativo
   - Versione GCC: `gcc --version`
   - Versione standard C usato

4. **Codice o screenshot** (se rilevante)

## Processo di Review

Le Pull Request saranno revisionate per:
- ✅ Correttezza tecnica
- ✅ Qualità del codice
- ✅ Chiarezza delle spiegazioni
- ✅ Completezza degli esempi
- ✅ Consistenza con il resto del corso

## Codice di Condotta

- Sii rispettoso e professionale
- Accetta feedback costruttivo
- Concentrati sul migliorare il corso
- Aiuta altri contributors quando possibile

## Domande?

Se hai domande:
1. Controlla le [Issues](https://github.com/TheEmilz/corso_modern_C/issues) esistenti
2. Apri una nuova Issue con tag "question"
3. Chiedi chiarimenti nella tua Pull Request

## Licenza

Contribuendo a questo progetto, accetti che il tuo contributo sarà rilasciato sotto la [MIT License](LICENSE).

---

Grazie per aver contribuito a rendere questo corso migliore per tutti! 🚀
