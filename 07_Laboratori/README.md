# Modulo 07 - Mini Laboratori

Questa sezione contiene progetti pratici completi per applicare le conoscenze acquisite.

## Lab 1: Sistema di Gestione File

### Obiettivo
Creare un programma per gestire un database di contatti salvato su file.

### Funzionalità
- Aggiungere contatti
- Cercare contatti per nome
- Modificare contatti esistenti
- Eliminare contatti
- Listare tutti i contatti
- Persistenza su file

### Implementazione

**contacts.h:**
```c
#ifndef CONTACTS_H
#define CONTACTS_H

#define MAX_NAME 50
#define MAX_PHONE 20
#define MAX_EMAIL 50
#define MAX_CONTACTS 100

typedef struct {
    int id;
    char name[MAX_NAME];
    char phone[MAX_PHONE];
    char email[MAX_EMAIL];
} Contact;

typedef struct {
    Contact contacts[MAX_CONTACTS];
    int count;
} ContactDatabase;

/* Funzioni del database */
void db_init(ContactDatabase *db);
int db_add_contact(ContactDatabase *db, const char *name, const char *phone, const char *email);
int db_find_by_name(const ContactDatabase *db, const char *name);
int db_remove_contact(ContactDatabase *db, int id);
void db_list_all(const ContactDatabase *db);
int db_save_to_file(const ContactDatabase *db, const char *filename);
int db_load_from_file(ContactDatabase *db, const char *filename);

#endif
```

**contacts.c:**
```c
#include <stdio.h>
#include <string.h>
#include "contacts.h"

static int next_id = 1;

void db_init(ContactDatabase *db) {
    db->count = 0;
    memset(db->contacts, 0, sizeof(db->contacts));
}

int db_add_contact(ContactDatabase *db, const char *name, const char *phone, const char *email) {
    if (db->count >= MAX_CONTACTS) {
        return -1;  // Database pieno
    }
    
    Contact *c = &db->contacts[db->count];
    c->id = next_id++;
    strncpy(c->name, name, MAX_NAME - 1);
    strncpy(c->phone, phone, MAX_PHONE - 1);
    strncpy(c->email, email, MAX_EMAIL - 1);
    
    db->count++;
    return c->id;
}

int db_find_by_name(const ContactDatabase *db, const char *name) {
    for (int i = 0; i < db->count; i++) {
        if (strstr(db->contacts[i].name, name) != NULL) {
            const Contact *c = &db->contacts[i];
            printf("\nID: %d\n", c->id);
            printf("Nome: %s\n", c->name);
            printf("Telefono: %s\n", c->phone);
            printf("Email: %s\n", c->email);
            return i;
        }
    }
    return -1;
}

int db_remove_contact(ContactDatabase *db, int id) {
    for (int i = 0; i < db->count; i++) {
        if (db->contacts[i].id == id) {
            /* Sposta tutti i contatti successivi */
            for (int j = i; j < db->count - 1; j++) {
                db->contacts[j] = db->contacts[j + 1];
            }
            db->count--;
            return 0;
        }
    }
    return -1;
}

void db_list_all(const ContactDatabase *db) {
    if (db->count == 0) {
        printf("Nessun contatto nel database.\n");
        return;
    }
    
    printf("\n=== Lista Contatti (%d totali) ===\n", db->count);
    for (int i = 0; i < db->count; i++) {
        const Contact *c = &db->contacts[i];
        printf("\n[%d] %s\n", c->id, c->name);
        printf("    Tel: %s\n", c->phone);
        printf("    Email: %s\n", c->email);
    }
}

int db_save_to_file(const ContactDatabase *db, const char *filename) {
    FILE *file = fopen(filename, "wb");
    if (!file) {
        return -1;
    }
    
    /* Salva conteggio e next_id */
    fwrite(&db->count, sizeof(int), 1, file);
    fwrite(&next_id, sizeof(int), 1, file);
    
    /* Salva contatti */
    fwrite(db->contacts, sizeof(Contact), db->count, file);
    
    fclose(file);
    return 0;
}

int db_load_from_file(ContactDatabase *db, const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (!file) {
        return -1;
    }
    
    /* Carica conteggio e next_id */
    fread(&db->count, sizeof(int), 1, file);
    fread(&next_id, sizeof(int), 1, file);
    
    /* Carica contatti */
    fread(db->contacts, sizeof(Contact), db->count, file);
    
    fclose(file);
    return 0;
}
```

**main.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "contacts.h"

#define DB_FILE "contacts.dat"

void print_menu(void) {
    printf("\n=== Gestione Contatti ===\n");
    printf("1. Aggiungi contatto\n");
    printf("2. Cerca contatto\n");
    printf("3. Lista tutti\n");
    printf("4. Elimina contatto\n");
    printf("5. Salva e Esci\n");
    printf("Scelta: ");
}

void add_contact_interactive(ContactDatabase *db) {
    char name[MAX_NAME], phone[MAX_PHONE], email[MAX_EMAIL];
    
    printf("\nNome: ");
    fgets(name, MAX_NAME, stdin);
    name[strcspn(name, "\n")] = 0;
    
    printf("Telefono: ");
    fgets(phone, MAX_PHONE, stdin);
    phone[strcspn(phone, "\n")] = 0;
    
    printf("Email: ");
    fgets(email, MAX_EMAIL, stdin);
    email[strcspn(email, "\n")] = 0;
    
    int id = db_add_contact(db, name, phone, email);
    if (id >= 0) {
        printf("Contatto aggiunto con ID: %d\n", id);
    } else {
        printf("Errore: database pieno!\n");
    }
}

void search_contact_interactive(const ContactDatabase *db) {
    char name[MAX_NAME];
    
    printf("\nCerca nome: ");
    fgets(name, MAX_NAME, stdin);
    name[strcspn(name, "\n")] = 0;
    
    if (db_find_by_name(db, name) < 0) {
        printf("Nessun contatto trovato.\n");
    }
}

void remove_contact_interactive(ContactDatabase *db) {
    int id;
    
    printf("\nID da eliminare: ");
    scanf("%d", &id);
    getchar();  // Consuma newline
    
    if (db_remove_contact(db, id) == 0) {
        printf("Contatto eliminato.\n");
    } else {
        printf("Contatto non trovato.\n");
    }
}

int main(void) {
    ContactDatabase db;
    
    /* Carica database esistente o inizializza nuovo */
    if (db_load_from_file(&db, DB_FILE) != 0) {
        printf("Creazione nuovo database...\n");
        db_init(&db);
    } else {
        printf("Database caricato: %d contatti\n", db.count);
    }
    
    int choice;
    do {
        print_menu();
        scanf("%d", &choice);
        getchar();  // Consuma newline
        
        switch (choice) {
            case 1:
                add_contact_interactive(&db);
                break;
            case 2:
                search_contact_interactive(&db);
                break;
            case 3:
                db_list_all(&db);
                break;
            case 4:
                remove_contact_interactive(&db);
                break;
            case 5:
                if (db_save_to_file(&db, DB_FILE) == 0) {
                    printf("Database salvato.\n");
                }
                printf("Arrivederci!\n");
                break;
            default:
                printf("Scelta non valida.\n");
        }
    } while (choice != 5);
    
    return 0;
}
```

**Compilazione:**
```bash
gcc -Wall -Wextra -o contacts main.c contacts.c
./contacts
```

## Lab 2: Implementazione Strutture Dati

### Obiettivo
Implementare strutture dati fondamentali da zero.

### A. Linked List

**linked_list.h:**
```c
#ifndef LINKED_LIST_H
#define LINKED_LIST_H

typedef struct Node {
    int data;
    struct Node *next;
} Node;

typedef struct {
    Node *head;
    int size;
} LinkedList;

void list_init(LinkedList *list);
void list_push_front(LinkedList *list, int data);
void list_push_back(LinkedList *list, int data);
int list_pop_front(LinkedList *list, int *data);
void list_print(const LinkedList *list);
void list_free(LinkedList *list);
int list_find(const LinkedList *list, int data);
void list_reverse(LinkedList *list);

#endif
```

**linked_list.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include "linked_list.h"

void list_init(LinkedList *list) {
    list->head = NULL;
    list->size = 0;
}

void list_push_front(LinkedList *list, int data) {
    Node *new_node = malloc(sizeof(Node));
    new_node->data = data;
    new_node->next = list->head;
    list->head = new_node;
    list->size++;
}

void list_push_back(LinkedList *list, int data) {
    Node *new_node = malloc(sizeof(Node));
    new_node->data = data;
    new_node->next = NULL;
    
    if (list->head == NULL) {
        list->head = new_node;
    } else {
        Node *current = list->head;
        while (current->next != NULL) {
            current = current->next;
        }
        current->next = new_node;
    }
    list->size++;
}

int list_pop_front(LinkedList *list, int *data) {
    if (list->head == NULL) {
        return -1;
    }
    
    Node *temp = list->head;
    *data = temp->data;
    list->head = list->head->next;
    free(temp);
    list->size--;
    return 0;
}

void list_print(const LinkedList *list) {
    printf("List (%d elements): ", list->size);
    Node *current = list->head;
    while (current != NULL) {
        printf("%d ", current->data);
        current = current->next;
    }
    printf("\n");
}

void list_free(LinkedList *list) {
    Node *current = list->head;
    while (current != NULL) {
        Node *temp = current;
        current = current->next;
        free(temp);
    }
    list->head = NULL;
    list->size = 0;
}

int list_find(const LinkedList *list, int data) {
    Node *current = list->head;
    int index = 0;
    while (current != NULL) {
        if (current->data == data) {
            return index;
        }
        current = current->next;
        index++;
    }
    return -1;
}

void list_reverse(LinkedList *list) {
    Node *prev = NULL;
    Node *current = list->head;
    Node *next = NULL;
    
    while (current != NULL) {
        next = current->next;
        current->next = prev;
        prev = current;
        current = next;
    }
    
    list->head = prev;
}
```

### B. Stack

**stack.h:**
```c
#ifndef STACK_H
#define STACK_H

#define STACK_MAX_SIZE 100

typedef struct {
    int data[STACK_MAX_SIZE];
    int top;
} Stack;

void stack_init(Stack *stack);
int stack_push(Stack *stack, int value);
int stack_pop(Stack *stack, int *value);
int stack_peek(const Stack *stack, int *value);
int stack_is_empty(const Stack *stack);
int stack_is_full(const Stack *stack);
int stack_size(const Stack *stack);

#endif
```

**stack.c:**
```c
#include "stack.h"

void stack_init(Stack *stack) {
    stack->top = -1;
}

int stack_push(Stack *stack, int value) {
    if (stack_is_full(stack)) {
        return -1;
    }
    stack->data[++stack->top] = value;
    return 0;
}

int stack_pop(Stack *stack, int *value) {
    if (stack_is_empty(stack)) {
        return -1;
    }
    *value = stack->data[stack->top--];
    return 0;
}

int stack_peek(const Stack *stack, int *value) {
    if (stack_is_empty(stack)) {
        return -1;
    }
    *value = stack->data[stack->top];
    return 0;
}

int stack_is_empty(const Stack *stack) {
    return stack->top == -1;
}

int stack_is_full(const Stack *stack) {
    return stack->top == STACK_MAX_SIZE - 1;
}

int stack_size(const Stack *stack) {
    return stack->top + 1;
}
```

### C. Hash Table

**hash_table.h:**
```c
#ifndef HASH_TABLE_H
#define HASH_TABLE_H

#define TABLE_SIZE 100

typedef struct Entry {
    char *key;
    int value;
    struct Entry *next;
} Entry;

typedef struct {
    Entry *buckets[TABLE_SIZE];
} HashTable;

void ht_init(HashTable *ht);
void ht_insert(HashTable *ht, const char *key, int value);
int ht_get(const HashTable *ht, const char *key, int *value);
int ht_remove(HashTable *ht, const char *key);
void ht_free(HashTable *ht);
void ht_print(const HashTable *ht);

#endif
```

**hash_table.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "hash_table.h"

static unsigned int hash(const char *key) {
    unsigned int hash = 5381;
    int c;
    
    while ((c = *key++)) {
        hash = ((hash << 5) + hash) + c;
    }
    
    return hash % TABLE_SIZE;
}

void ht_init(HashTable *ht) {
    for (int i = 0; i < TABLE_SIZE; i++) {
        ht->buckets[i] = NULL;
    }
}

void ht_insert(HashTable *ht, const char *key, int value) {
    unsigned int index = hash(key);
    
    /* Cerca se la chiave esiste già */
    Entry *entry = ht->buckets[index];
    while (entry != NULL) {
        if (strcmp(entry->key, key) == 0) {
            entry->value = value;  /* Aggiorna valore esistente */
            return;
        }
        entry = entry->next;
    }
    
    /* Crea nuova entry */
    Entry *new_entry = malloc(sizeof(Entry));
    new_entry->key = strdup(key);
    new_entry->value = value;
    new_entry->next = ht->buckets[index];
    ht->buckets[index] = new_entry;
}

int ht_get(const HashTable *ht, const char *key, int *value) {
    unsigned int index = hash(key);
    
    Entry *entry = ht->buckets[index];
    while (entry != NULL) {
        if (strcmp(entry->key, key) == 0) {
            *value = entry->value;
            return 0;
        }
        entry = entry->next;
    }
    
    return -1;  /* Non trovato */
}

int ht_remove(HashTable *ht, const char *key) {
    unsigned int index = hash(key);
    
    Entry *entry = ht->buckets[index];
    Entry *prev = NULL;
    
    while (entry != NULL) {
        if (strcmp(entry->key, key) == 0) {
            if (prev == NULL) {
                ht->buckets[index] = entry->next;
            } else {
                prev->next = entry->next;
            }
            free(entry->key);
            free(entry);
            return 0;
        }
        prev = entry;
        entry = entry->next;
    }
    
    return -1;  /* Non trovato */
}

void ht_free(HashTable *ht) {
    for (int i = 0; i < TABLE_SIZE; i++) {
        Entry *entry = ht->buckets[i];
        while (entry != NULL) {
            Entry *temp = entry;
            entry = entry->next;
            free(temp->key);
            free(temp);
        }
        ht->buckets[i] = NULL;
    }
}

void ht_print(const HashTable *ht) {
    printf("Hash Table Contents:\n");
    for (int i = 0; i < TABLE_SIZE; i++) {
        if (ht->buckets[i] != NULL) {
            printf("Bucket %d: ", i);
            Entry *entry = ht->buckets[i];
            while (entry != NULL) {
                printf("(%s: %d) ", entry->key, entry->value);
                entry = entry->next;
            }
            printf("\n");
        }
    }
}
```

## Lab 3: Network Programming

### Obiettivo
Creare un server TCP semplice e un client.

### Server TCP

**server.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024

int main(void) {
    int server_fd, client_fd;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};
    
    /* Crea socket */
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }
    
    /* Imposta opzioni socket */
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }
    
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);
    
    /* Bind */
    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }
    
    /* Listen */
    if (listen(server_fd, 3) < 0) {
        perror("listen");
        exit(EXIT_FAILURE);
    }
    
    printf("Server in ascolto sulla porta %d...\n", PORT);
    
    /* Accept loop */
    while (1) {
        if ((client_fd = accept(server_fd, (struct sockaddr *)&address, 
                                (socklen_t*)&addrlen)) < 0) {
            perror("accept");
            continue;
        }
        
        printf("Client connesso!\n");
        
        /* Ricevi messaggio */
        int valread = read(client_fd, buffer, BUFFER_SIZE);
        printf("Ricevuto: %s\n", buffer);
        
        /* Invia risposta */
        const char *response = "Messaggio ricevuto!";
        send(client_fd, response, strlen(response), 0);
        printf("Risposta inviata\n");
        
        close(client_fd);
        memset(buffer, 0, BUFFER_SIZE);
    }
    
    close(server_fd);
    return 0;
}
```

### Client TCP

**client.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024

int main(void) {
    int sock = 0;
    struct sockaddr_in serv_addr;
    char buffer[BUFFER_SIZE] = {0};
    
    /* Crea socket */
    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        printf("Errore creazione socket\n");
        return -1;
    }
    
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    
    /* Converti indirizzo da testo a binario */
    if (inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr) <= 0) {
        printf("Indirizzo non valido\n");
        return -1;
    }
    
    /* Connetti al server */
    if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        printf("Connessione fallita\n");
        return -1;
    }
    
    /* Invia messaggio */
    const char *message = "Ciao dal client!";
    send(sock, message, strlen(message), 0);
    printf("Messaggio inviato: %s\n", message);
    
    /* Ricevi risposta */
    int valread = read(sock, buffer, BUFFER_SIZE);
    printf("Risposta dal server: %s\n", buffer);
    
    close(sock);
    return 0;
}
```

**Compilazione ed esecuzione:**
```bash
# Compila
gcc -o server server.c
gcc -o client client.c

# Esegui server in un terminale
./server

# Esegui client in altro terminale
./client
```

## Lab 4: Sistema Embedded Simulato

### Obiettivo
Simulare un sistema embedded con sensori e attuatori.

**embedded_sim.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <time.h>
#include <unistd.h>

/* Simulazione registri hardware */
typedef struct {
    volatile unsigned int control;
    volatile unsigned int status;
    volatile unsigned int data;
} HardwareRegister;

/* Sensore di temperatura */
typedef struct {
    HardwareRegister reg;
    float temperature;
} TemperatureSensor;

/* LED */
typedef struct {
    HardwareRegister reg;
    bool state;
} LED;

/* Inizializza sensore */
void sensor_init(TemperatureSensor *sensor) {
    sensor->reg.control = 0x01;  /* Enable */
    sensor->reg.status = 0x00;
    sensor->temperature = 20.0f;
}

/* Leggi temperatura (simulata) */
float sensor_read(TemperatureSensor *sensor) {
    /* Simula variazione temperatura */
    float variation = ((rand() % 100) - 50) / 100.0f;
    sensor->temperature += variation;
    
    /* Limita range */
    if (sensor->temperature < 15.0f) sensor->temperature = 15.0f;
    if (sensor->temperature > 30.0f) sensor->temperature = 30.0f;
    
    sensor->reg.data = (unsigned int)(sensor->temperature * 100);
    sensor->reg.status = 0x01;  /* Data ready */
    
    return sensor->temperature;
}

/* Inizializza LED */
void led_init(LED *led) {
    led->reg.control = 0x01;  /* Enable */
    led->state = false;
}

/* Controlla LED */
void led_set(LED *led, bool state) {
    led->state = state;
    led->reg.data = state ? 0xFF : 0x00;
}

/* Sistema di controllo */
typedef struct {
    TemperatureSensor temp_sensor;
    LED warning_led;
    LED status_led;
    float threshold_high;
    float threshold_low;
} ControlSystem;

void system_init(ControlSystem *sys) {
    sensor_init(&sys->temp_sensor);
    led_init(&sys->warning_led);
    led_init(&sys->status_led);
    sys->threshold_high = 25.0f;
    sys->threshold_low = 18.0f;
}

void system_update(ControlSystem *sys) {
    float temp = sensor_read(&sys->temp_sensor);
    
    /* Status LED: always blinking */
    static bool blink = false;
    led_set(&sys->status_led, blink);
    blink = !blink;
    
    /* Warning LED: on if temperature out of range */
    if (temp > sys->threshold_high || temp < sys->threshold_low) {
        led_set(&sys->warning_led, true);
    } else {
        led_set(&sys->warning_led, false);
    }
    
    /* Stampa stato */
    printf("Temp: %.2f°C | Status LED: %s | Warning LED: %s\n",
           temp,
           sys->status_led.state ? "ON " : "OFF",
           sys->warning_led.state ? "ON " : "OFF");
}

int main(void) {
    srand(time(NULL));
    
    ControlSystem system;
    system_init(&system);
    
    printf("=== Sistema Embedded Simulato ===\n");
    printf("Soglie: %.1f°C - %.1f°C\n\n", 
           system.threshold_low, system.threshold_high);
    
    /* Loop principale (simula 30 secondi) */
    for (int i = 0; i < 30; i++) {
        system_update(&system);
        sleep(1);
    }
    
    printf("\nSimulazione terminata.\n");
    return 0;
}
```

## Lab 5: Parser e Compiler Basics

### Obiettivo
Creare un semplice parser per espressioni matematiche e un evaluator.

**expression_parser.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

typedef enum {
    TOKEN_NUMBER,
    TOKEN_PLUS,
    TOKEN_MINUS,
    TOKEN_MULTIPLY,
    TOKEN_DIVIDE,
    TOKEN_LPAREN,
    TOKEN_RPAREN,
    TOKEN_END
} TokenType;

typedef struct {
    TokenType type;
    double value;
} Token;

/* Tokenizer */
typedef struct {
    const char *input;
    size_t pos;
    Token current;
} Tokenizer;

void tokenizer_init(Tokenizer *t, const char *input) {
    t->input = input;
    t->pos = 0;
}

void tokenizer_skip_whitespace(Tokenizer *t) {
    while (isspace(t->input[t->pos])) {
        t->pos++;
    }
}

Token tokenizer_next(Tokenizer *t) {
    tokenizer_skip_whitespace(t);
    
    Token token;
    
    if (t->input[t->pos] == '\0') {
        token.type = TOKEN_END;
        return token;
    }
    
    if (isdigit(t->input[t->pos])) {
        char *end;
        token.type = TOKEN_NUMBER;
        token.value = strtod(&t->input[t->pos], &end);
        t->pos = end - t->input;
        return token;
    }
    
    switch (t->input[t->pos]) {
        case '+': token.type = TOKEN_PLUS; break;
        case '-': token.type = TOKEN_MINUS; break;
        case '*': token.type = TOKEN_MULTIPLY; break;
        case '/': token.type = TOKEN_DIVIDE; break;
        case '(': token.type = TOKEN_LPAREN; break;
        case ')': token.type = TOKEN_RPAREN; break;
        default:
            fprintf(stderr, "Carattere non valido: %c\n", t->input[t->pos]);
            token.type = TOKEN_END;
            return token;
    }
    
    t->pos++;
    return token;
}

/* Parser ricorsivo-discendente */
double parse_expression(Tokenizer *t);
double parse_term(Tokenizer *t);
double parse_factor(Tokenizer *t);

double parse_factor(Tokenizer *t) {
    Token token = tokenizer_next(t);
    
    if (token.type == TOKEN_NUMBER) {
        return token.value;
    }
    
    if (token.type == TOKEN_LPAREN) {
        double value = parse_expression(t);
        token = tokenizer_next(t);  /* Consuma ')' */
        if (token.type != TOKEN_RPAREN) {
            fprintf(stderr, "Errore: ')' atteso\n");
        }
        return value;
    }
    
    fprintf(stderr, "Errore: numero o '(' atteso\n");
    return 0;
}

double parse_term(Tokenizer *t) {
    double left = parse_factor(t);
    
    while (1) {
        size_t saved_pos = t->pos;
        Token token = tokenizer_next(t);
        
        if (token.type == TOKEN_MULTIPLY) {
            left *= parse_factor(t);
        } else if (token.type == TOKEN_DIVIDE) {
            double right = parse_factor(t);
            if (right == 0) {
                fprintf(stderr, "Errore: divisione per zero\n");
                return 0;
            }
            left /= right;
        } else {
            t->pos = saved_pos;  /* Backtrack */
            break;
        }
    }
    
    return left;
}

double parse_expression(Tokenizer *t) {
    double left = parse_term(t);
    
    while (1) {
        size_t saved_pos = t->pos;
        Token token = tokenizer_next(t);
        
        if (token.type == TOKEN_PLUS) {
            left += parse_term(t);
        } else if (token.type == TOKEN_MINUS) {
            left -= parse_term(t);
        } else {
            t->pos = saved_pos;  /* Backtrack */
            break;
        }
    }
    
    return left;
}

double evaluate(const char *expr) {
    Tokenizer t;
    tokenizer_init(&t, expr);
    return parse_expression(&t);
}

int main(void) {
    char input[256];
    
    printf("=== Calcolatrice con Parser ===\n");
    printf("Operazioni supportate: +, -, *, /, ()\n");
    printf("Inserisci 'quit' per uscire\n\n");
    
    while (1) {
        printf(">>> ");
        if (!fgets(input, sizeof(input), stdin)) {
            break;
        }
        
        input[strcspn(input, "\n")] = 0;
        
        if (strcmp(input, "quit") == 0) {
            break;
        }
        
        double result = evaluate(input);
        printf("Risultato: %.2f\n\n", result);
    }
    
    return 0;
}
```

**Test:**
```
>>> 2 + 3 * 4
Risultato: 14.00

>>> (2 + 3) * 4
Risultato: 20.00

>>> 10 / 2 + 3
Risultato: 8.00
```

## Esercizi Aggiuntivi

1. Estendi il sistema di gestione contatti con ricerca avanzata
2. Implementa una coda (queue) usando linked list
3. Crea un server multi-threaded che gestisce più client
4. Aggiungi interrupts simulati al sistema embedded
5. Estendi il parser per supportare variabili e funzioni

---

[← Comparazioni](../06_Comparazioni/README.md) | [Torna al Sommario](../README.md) | [Prossimo: Esercizi →](../08_Esercizi/README.md)
