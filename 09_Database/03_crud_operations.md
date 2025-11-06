# Lezione 3: Operazioni CRUD Complete

## Introduzione a CRUD

**CRUD** è l'acronimo di:
- **C**reate - Creare/Inserire dati
- **R**ead - Leggere/Recuperare dati
- **U**pdate - Aggiornare dati esistenti
- **D**elete - Eliminare dati

Queste sono le quattro operazioni fondamentali di qualsiasi applicazione che gestisce dati persistenti.

## Setup: Creazione Database di Esempio

Prima di iniziare con le operazioni CRUD, creiamo un database di esempio per un sistema di gestione studenti.

### Schema del Database

```sql
CREATE TABLE Students (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    age INTEGER CHECK(age > 0 AND age < 150),
    email TEXT UNIQUE,
    grade REAL DEFAULT 0.0
);
```

### Programma di Setup

```c
/*
 * setup_db.c - Crea il database studenti
 */

#include <stdio.h>
#include <stdlib.h>
#include <sqlite3.h>

void handle_error(sqlite3 *db, const char *msg)
{
    fprintf(stderr, "ERRORE: %s\n", msg);
    if (db) {
        fprintf(stderr, "SQLite: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
    }
    exit(1);
}

int main(void)
{
    sqlite3 *db;
    char *err_msg = NULL;
    int rc;
    
    /* Apri database */
    rc = sqlite3_open("students.db", &db);
    if (rc != SQLITE_OK) {
        handle_error(db, "Impossibile aprire database");
    }
    
    printf("Database aperto\n");
    
    /* Crea tabella */
    const char *sql_create = 
        "DROP TABLE IF EXISTS Students;"
        "CREATE TABLE Students ("
        "    id INTEGER PRIMARY KEY AUTOINCREMENT,"
        "    name TEXT NOT NULL,"
        "    age INTEGER CHECK(age > 0 AND age < 150),"
        "    email TEXT UNIQUE,"
        "    grade REAL DEFAULT 0.0"
        ");";
    
    rc = sqlite3_exec(db, sql_create, NULL, NULL, &err_msg);
    if (rc != SQLITE_OK) {
        sqlite3_free(err_msg);
        handle_error(db, "Errore creazione tabella");
    }
    
    printf("Tabella Students creata con successo\n");
    
    sqlite3_close(db);
    return 0;
}
```

**Compilazione**:
```bash
gcc -Wall -Wextra -std=c11 setup_db.c -o setup_db -lsqlite3
./setup_db
```

## CREATE - Inserimento Dati

### Inserimento Singolo

```c
/*
 * create.c - Inserimento dati
 */

#include <stdio.h>
#include <stdlib.h>
#include <sqlite3.h>

int insert_student(sqlite3 *db, const char *name, int age, 
                   const char *email, double grade)
{
    char *err_msg = NULL;
    char sql[512];
    
    /* Costruisci query SQL */
    snprintf(sql, sizeof(sql),
             "INSERT INTO Students (name, age, email, grade) "
             "VALUES ('%s', %d, '%s', %.2f);",
             name, age, email, grade);
    
    /* Esegui query */
    int rc = sqlite3_exec(db, sql, NULL, NULL, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore inserimento: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    printf("Studente '%s' inserito con ID: %lld\n", 
           name, sqlite3_last_insert_rowid(db));
    
    return 0;
}

int main(void)
{
    sqlite3 *db;
    int rc = sqlite3_open("students.db", &db);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore apertura: %s\n", sqlite3_errmsg(db));
        return 1;
    }
    
    /* Inserisci alcuni studenti */
    insert_student(db, "Mario Rossi", 20, "mario@email.com", 28.5);
    insert_student(db, "Anna Verdi", 22, "anna@email.com", 30.0);
    insert_student(db, "Luigi Bianchi", 21, "luigi@email.com", 27.0);
    
    sqlite3_close(db);
    return 0;
}
```

### Spiegazione Dettagliata

**`snprintf()` per Costruire Query**:
```c
snprintf(sql, sizeof(sql),
         "INSERT INTO Students (name, age, email, grade) "
         "VALUES ('%s', %d, '%s', %.2f);",
         name, age, email, grade);
```
- `snprintf`: Formatta stringa in modo sicuro (evita buffer overflow)
- `sizeof(sql)`: Dimensione massima del buffer
- `'%s'`: Placeholder per stringa (con apici per SQL)
- `%d`: Placeholder per intero
- `%.2f`: Placeholder per float con 2 decimali

⚠️ **ATTENZIONE**: Questo metodo è vulnerabile a SQL injection! Nella prossima lezione vedremo prepared statements.

**`sqlite3_last_insert_rowid()`**:
```c
long long id = sqlite3_last_insert_rowid(db);
```
- Ritorna l'ID dell'ultimo record inserito
- Utile per AUTO_INCREMENT
- Funziona solo immediatamente dopo INSERT

### Inserimento Multiplo con Transazione

```c
int insert_multiple_students(sqlite3 *db)
{
    char *err_msg = NULL;
    int rc;
    
    /* Inizia transazione */
    rc = sqlite3_exec(db, "BEGIN TRANSACTION;", NULL, NULL, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore BEGIN: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    /* Inserisci multipli studenti */
    const char *students[] = {
        "INSERT INTO Students (name, age, email, grade) VALUES ('Paolo Neri', 23, 'paolo@email.com', 26.5);",
        "INSERT INTO Students (name, age, email, grade) VALUES ('Sara Rosa', 19, 'sara@email.com', 29.0);",
        "INSERT INTO Students (name, age, email, grade) VALUES ('Marco Blu', 24, 'marco@email.com', 25.0);"
    };
    
    for (int i = 0; i < 3; i++) {
        rc = sqlite3_exec(db, students[i], NULL, NULL, &err_msg);
        if (rc != SQLITE_OK) {
            fprintf(stderr, "Errore inserimento: %s\n", err_msg);
            sqlite3_free(err_msg);
            
            /* Rollback in caso di errore */
            sqlite3_exec(db, "ROLLBACK;", NULL, NULL, NULL);
            return -1;
        }
    }
    
    /* Commit transazione */
    rc = sqlite3_exec(db, "COMMIT;", NULL, NULL, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore COMMIT: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    printf("Inseriti 3 studenti con successo\n");
    return 0;
}
```

**Perché Usare Transazioni?**
- **Atomicità**: Tutti gli inserimenti hanno successo o nessuno
- **Performance**: Molto più veloce per inserimenti multipli
- **Integrità**: Stato consistente del database

## READ - Lettura Dati

### Callback per Elaborare Risultati

SQLite usa callback per processare i risultati di SELECT:

```c
/*
 * read.c - Lettura dati con callback
 */

#include <stdio.h>
#include <sqlite3.h>

/* Callback per processare ogni riga */
int callback(void *data, int argc, char **argv, char **col_names)
{
    /* 
     * data: Puntatore passato come ultimo arg a sqlite3_exec
     * argc: Numero di colonne
     * argv: Array di valori (come stringhe)
     * col_names: Array di nomi colonne
     */
    
    int *count = (int *)data;
    (*count)++;
    
    printf("\nRecord #%d:\n", *count);
    
    for (int i = 0; i < argc; i++) {
        printf("  %s = %s\n", col_names[i], argv[i] ? argv[i] : "NULL");
    }
    
    return 0;  // 0 = continua, non-zero = interrompi
}

int read_all_students(sqlite3 *db)
{
    char *err_msg = NULL;
    int count = 0;
    
    const char *sql = "SELECT * FROM Students ORDER BY name;";
    
    printf("=== Tutti gli Studenti ===\n");
    
    int rc = sqlite3_exec(db, sql, callback, &count, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore SELECT: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    printf("\nTotale studenti: %d\n", count);
    return 0;
}

int main(void)
{
    sqlite3 *db;
    
    if (sqlite3_open("students.db", &db) != SQLITE_OK) {
        fprintf(stderr, "Errore apertura: %s\n", sqlite3_errmsg(db));
        return 1;
    }
    
    read_all_students(db);
    
    sqlite3_close(db);
    return 0;
}
```

### Lettura con Filtri

```c
int read_students_by_grade(sqlite3 *db, double min_grade)
{
    char sql[256];
    char *err_msg = NULL;
    int count = 0;
    
    snprintf(sql, sizeof(sql),
             "SELECT name, grade FROM Students "
             "WHERE grade >= %.2f "
             "ORDER BY grade DESC;",
             min_grade);
    
    printf("\n=== Studenti con voto >= %.2f ===\n", min_grade);
    
    int rc = sqlite3_exec(db, sql, callback, &count, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    if (count == 0) {
        printf("Nessuno studente trovato con voto >= %.2f\n", min_grade);
    }
    
    return 0;
}
```

### Query con Aggregazione

```c
/* Callback per risultato singolo */
int single_result_callback(void *data, int argc, char **argv, char **col_names)
{
    if (argc > 0 && argv[0]) {
        printf("Risultato: %s\n", argv[0]);
    }
    return 0;
}

int get_statistics(sqlite3 *db)
{
    char *err_msg = NULL;
    
    printf("\n=== Statistiche ===\n");
    
    /* Numero totale studenti */
    printf("Totale studenti: ");
    sqlite3_exec(db, "SELECT COUNT(*) FROM Students;", 
                 single_result_callback, NULL, &err_msg);
    
    /* Media voti */
    printf("Media voti: ");
    sqlite3_exec(db, "SELECT AVG(grade) FROM Students;",
                 single_result_callback, NULL, &err_msg);
    
    /* Voto massimo */
    printf("Voto massimo: ");
    sqlite3_exec(db, "SELECT MAX(grade) FROM Students;",
                 single_result_callback, NULL, &err_msg);
    
    /* Voto minimo */
    printf("Voto minimo: ");
    sqlite3_exec(db, "SELECT MIN(grade) FROM Students;",
                 single_result_callback, NULL, &err_msg);
    
    return 0;
}
```

## UPDATE - Aggiornamento Dati

### Aggiornamento Singolo Record

```c
/*
 * update.c - Aggiornamento dati
 */

int update_student_grade(sqlite3 *db, int student_id, double new_grade)
{
    char sql[256];
    char *err_msg = NULL;
    
    snprintf(sql, sizeof(sql),
             "UPDATE Students SET grade = %.2f WHERE id = %d;",
             new_grade, student_id);
    
    int rc = sqlite3_exec(db, sql, NULL, NULL, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore UPDATE: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    /* Verifica quante righe sono state modificate */
    int rows = sqlite3_changes(db);
    
    if (rows == 0) {
        printf("Nessuno studente con ID %d trovato\n", student_id);
        return -1;
    }
    
    printf("Voto aggiornato per studente ID %d (righe modificate: %d)\n", 
           student_id, rows);
    
    return 0;
}
```

### Aggiornamento Multiplo

```c
int update_grades_by_percentage(sqlite3 *db, double percentage)
{
    char sql[256];
    char *err_msg = NULL;
    
    /* Aumenta tutti i voti del X% */
    snprintf(sql, sizeof(sql),
             "UPDATE Students SET grade = grade * %.2f;",
             1.0 + (percentage / 100.0));
    
    int rc = sqlite3_exec(db, sql, NULL, NULL, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    int rows = sqlite3_changes(db);
    printf("Voti aumentati del %.1f%% per %d studenti\n", percentage, rows);
    
    return 0;
}
```

### Aggiornamento con Condizioni

```c
int update_email(sqlite3 *db, const char *old_email, const char *new_email)
{
    char sql[512];
    char *err_msg = NULL;
    
    snprintf(sql, sizeof(sql),
             "UPDATE Students SET email = '%s' WHERE email = '%s';",
             new_email, old_email);
    
    int rc = sqlite3_exec(db, sql, NULL, NULL, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    int rows = sqlite3_changes(db);
    
    if (rows == 0) {
        printf("Email '%s' non trovata\n", old_email);
        return -1;
    }
    
    printf("Email aggiornata per %d studente(i)\n", rows);
    return 0;
}
```

## DELETE - Eliminazione Dati

### Eliminazione Singolo Record

```c
/*
 * delete.c - Eliminazione dati
 */

int delete_student_by_id(sqlite3 *db, int student_id)
{
    char sql[256];
    char *err_msg = NULL;
    
    snprintf(sql, sizeof(sql),
             "DELETE FROM Students WHERE id = %d;",
             student_id);
    
    int rc = sqlite3_exec(db, sql, NULL, NULL, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore DELETE: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    int rows = sqlite3_changes(db);
    
    if (rows == 0) {
        printf("Nessuno studente con ID %d\n", student_id);
        return -1;
    }
    
    printf("Studente ID %d eliminato\n", student_id);
    return 0;
}
```

### Eliminazione con Condizioni

```c
int delete_students_by_grade(sqlite3 *db, double max_grade)
{
    char sql[256];
    char *err_msg = NULL;
    
    /* Elimina studenti con voto inferiore a soglia */
    snprintf(sql, sizeof(sql),
             "DELETE FROM Students WHERE grade < %.2f;",
             max_grade);
    
    int rc = sqlite3_exec(db, sql, NULL, NULL, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    int rows = sqlite3_changes(db);
    printf("Eliminati %d studenti con voto < %.2f\n", rows, max_grade);
    
    return 0;
}
```

### Eliminazione con Conferma

```c
int delete_all_students_safe(sqlite3 *db)
{
    char confirm[10];
    char *err_msg = NULL;
    
    printf("ATTENZIONE: Stai per eliminare TUTTI gli studenti!\n");
    printf("Sei sicuro? (digita 'SI' per confermare): ");
    fgets(confirm, sizeof(confirm), stdin);
    
    /* Rimuovi newline */
    confirm[strcspn(confirm, "\n")] = '\0';
    
    if (strcmp(confirm, "SI") != 0) {
        printf("Operazione annullata\n");
        return 0;
    }
    
    int rc = sqlite3_exec(db, "DELETE FROM Students;", NULL, NULL, &err_msg);
    
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Errore: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    printf("Tutti gli studenti eliminati\n");
    return 0;
}
```

## Programma Completo CRUD

```c
/*
 * crud_complete.c - Sistema CRUD completo
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sqlite3.h>

/* Prototipi funzioni */
void show_menu(void);
int create_student(sqlite3 *db);
int read_students(sqlite3 *db);
int update_student(sqlite3 *db);
int delete_student(sqlite3 *db);

/* Callback per visualizzare risultati */
int display_callback(void *data, int argc, char **argv, char **col_names)
{
    for (int i = 0; i < argc; i++) {
        printf("%-15s: %s\n", col_names[i], argv[i] ? argv[i] : "NULL");
    }
    printf("-----------------------------------\n");
    return 0;
}

int main(void)
{
    sqlite3 *db;
    int choice;
    
    /* Apri database */
    if (sqlite3_open("students.db", &db) != SQLITE_OK) {
        fprintf(stderr, "Errore apertura database\n");
        return 1;
    }
    
    printf("Sistema di Gestione Studenti\n");
    printf("=============================\n\n");
    
    /* Loop principale */
    while (1) {
        show_menu();
        printf("Scelta: ");
        scanf("%d", &choice);
        getchar();  // Consuma newline
        
        printf("\n");
        
        switch (choice) {
            case 1:
                create_student(db);
                break;
            case 2:
                read_students(db);
                break;
            case 3:
                update_student(db);
                break;
            case 4:
                delete_student(db);
                break;
            case 5:
                printf("Arrivederci!\n");
                sqlite3_close(db);
                return 0;
            default:
                printf("Scelta non valida\n");
        }
        
        printf("\nPremi INVIO per continuare...");
        getchar();
    }
    
    return 0;
}

void show_menu(void)
{
    printf("\n=== MENU ===\n");
    printf("1. Crea nuovo studente\n");
    printf("2. Visualizza studenti\n");
    printf("3. Aggiorna studente\n");
    printf("4. Elimina studente\n");
    printf("5. Esci\n");
}

int create_student(sqlite3 *db)
{
    char name[100], email[100];
    int age;
    double grade;
    char sql[512];
    char *err_msg = NULL;
    
    printf("=== Nuovo Studente ===\n");
    printf("Nome: ");
    fgets(name, sizeof(name), stdin);
    name[strcspn(name, "\n")] = '\0';
    
    printf("Età: ");
    scanf("%d", &age);
    getchar();
    
    printf("Email: ");
    fgets(email, sizeof(email), stdin);
    email[strcspn(email, "\n")] = '\0';
    
    printf("Voto: ");
    scanf("%lf", &grade);
    getchar();
    
    snprintf(sql, sizeof(sql),
             "INSERT INTO Students (name, age, email, grade) "
             "VALUES ('%s', %d, '%s', %.2f);",
             name, age, email, grade);
    
    if (sqlite3_exec(db, sql, NULL, NULL, &err_msg) != SQLITE_OK) {
        fprintf(stderr, "Errore: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    printf("\n✓ Studente creato con ID: %lld\n", sqlite3_last_insert_rowid(db));
    return 0;
}

int read_students(sqlite3 *db)
{
    char *err_msg = NULL;
    
    printf("=== Tutti gli Studenti ===\n\n");
    
    const char *sql = "SELECT * FROM Students ORDER BY name;";
    
    if (sqlite3_exec(db, sql, display_callback, NULL, &err_msg) != SQLITE_OK) {
        fprintf(stderr, "Errore: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    return 0;
}

int update_student(sqlite3 *db)
{
    int id;
    double new_grade;
    char sql[256];
    char *err_msg = NULL;
    
    printf("=== Aggiorna Voto Studente ===\n");
    printf("ID studente: ");
    scanf("%d", &id);
    getchar();
    
    printf("Nuovo voto: ");
    scanf("%lf", &new_grade);
    getchar();
    
    snprintf(sql, sizeof(sql),
             "UPDATE Students SET grade = %.2f WHERE id = %d;",
             new_grade, id);
    
    if (sqlite3_exec(db, sql, NULL, NULL, &err_msg) != SQLITE_OK) {
        fprintf(stderr, "Errore: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    if (sqlite3_changes(db) == 0) {
        printf("\n✗ Studente con ID %d non trovato\n", id);
        return -1;
    }
    
    printf("\n✓ Voto aggiornato con successo\n");
    return 0;
}

int delete_student(sqlite3 *db)
{
    int id;
    char sql[256];
    char *err_msg = NULL;
    
    printf("=== Elimina Studente ===\n");
    printf("ID studente: ");
    scanf("%d", &id);
    getchar();
    
    snprintf(sql, sizeof(sql),
             "DELETE FROM Students WHERE id = %d;",
             id);
    
    if (sqlite3_exec(db, sql, NULL, NULL, &err_msg) != SQLITE_OK) {
        fprintf(stderr, "Errore: %s\n", err_msg);
        sqlite3_free(err_msg);
        return -1;
    }
    
    if (sqlite3_changes(db) == 0) {
        printf("\n✗ Studente con ID %d non trovato\n", id);
        return -1;
    }
    
    printf("\n✓ Studente eliminato\n");
    return 0;
}
```

**Compilazione**:
```bash
gcc -Wall -Wextra -std=c11 crud_complete.c -o crud -lsqlite3
./crud
```

## Debug: Errori Comuni CRUD

### Errore 1: SQL Injection

**Problema**: Query costruite con concatenazione stringhe
```c
// VULNERABILE!
sprintf(sql, "SELECT * FROM Students WHERE name = '%s';", user_input);
```

**Problema**: Se `user_input = "Mario'; DROP TABLE Students; --"`, la query diventa:
```sql
SELECT * FROM Students WHERE name = 'Mario'; DROP TABLE Students; --';
```

**Soluzione**: Usa prepared statements (prossima lezione)

### Errore 2: Dimenticare `sqlite3_changes()`

```c
// Senza controllo
sqlite3_exec(db, "DELETE FROM Students WHERE id = 999;", ...);
printf("Eliminato!\n");  // Ma esisteva davvero?

// Con controllo
sqlite3_exec(db, "DELETE FROM Students WHERE id = 999;", ...);
if (sqlite3_changes(db) == 0) {
    printf("ID non trovato\n");
}
```

### Errore 3: Non Liberare `err_msg`

```c
// Memory leak!
if (rc != SQLITE_OK) {
    fprintf(stderr, "Errore: %s\n", err_msg);
    return -1;  // err_msg non liberato!
}

// Corretto
if (rc != SQLITE_OK) {
    fprintf(stderr, "Errore: %s\n", err_msg);
    sqlite3_free(err_msg);
    return -1;
}
```

## Funzioni Utili SQLite

```c
/* Numero di righe modificate dall'ultima operazione */
int rows = sqlite3_changes(db);

/* ID dell'ultimo INSERT */
long long id = sqlite3_last_insert_rowid(db);

/* Numero totale di modifiche dalla connessione */
int total = sqlite3_total_changes(db);

/* Messaggio ultimo errore */
const char *msg = sqlite3_errmsg(db);

/* Codice ultimo errore */
int code = sqlite3_errcode(db);
```

## Punti Chiave

✅ CRUD = Create, Read, Update, Delete
✅ `INSERT` per creare, `SELECT` per leggere, `UPDATE` per aggiornare, `DELETE` per eliminare
✅ Usa callback per processare risultati SELECT
✅ `sqlite3_changes()` dice quante righe sono state modificate
✅ `sqlite3_last_insert_rowid()` ritorna l'ID dell'ultimo inserimento
✅ Transazioni migliorano performance e garantiscono atomicità
✅ **Mai** costruire query concatenando input utente (SQL injection!)
✅ Sempre liberare `err_msg` con `sqlite3_free()`

## Prossimi Passi

Nella prossima lezione impareremo:
- **Prepared statements** per sicurezza
- Come evitare SQL injection
- Binding di parametri
- Performance migliorate

---

[← SQLite Basics](02_sqlite_basics.md) | [Torna al Modulo](README.md) | [Prossimo: Prepared Statements →](04_prepared_statements.md)
