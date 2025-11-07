# Lezione 3 - Endianness e Rappresentazione Binaria

## 🎯 Obiettivi

- Comprendere Little-Endian vs Big-Endian
- Visualizzare byte in memoria
- Convertire tra endianness
- Gestire network byte order

---

## 1. COS'È L'ENDIANNESS?

**Endianness**: Ordine in cui i byte di un valore multi-byte sono memorizzati.

```
NUMERO: int x = 0x12345678

BIT SIGNIFICATIVI:
0x12 = byte più significativo (Most Significant Byte - MSB)
0x78 = byte meno significativo (Least Significant Byte - LSB)

DUE MODI DI MEMORIZZARE:

LITTLE-ENDIAN (x86, x86-64, ARM moderno):
Indirizzo    0x1000  0x1001  0x1002  0x1003
Contenuto      0x78    0x56    0x34    0x12
               LSB →  →  →  →  →  →    MSB
               
Byte MENO significativo PRIMO

BIG-ENDIAN (Network, MIPS, PowerPC, ARM vecchio):
Indirizzo    0x1000  0x1001  0x1002  0x1003
Contenuto      0x12    0x34    0x56    0x78
               MSB →  →  →  →  →  →    LSB
               
Byte PIÙ significativo PRIMO

ANALOGIA:
Little-Endian = Scrivere data: 14/12/2024 (giorno prima)
Big-Endian = Scrivere data: 2024/12/14 (anno prima)
```

---

## 2. VEDERE I BYTE IN MEMORIA

### Metodo 1: Union Trick

```c
#include <stdio.h>
#include <stdint.h>

void print_bytes(void *ptr, size_t size) {
    unsigned char *byte_ptr = (unsigned char *)ptr;
    for (size_t i = 0; i < size; i++) {
        printf("Byte %zu: 0x%02X\n", i, byte_ptr[i]);
    }
}

int main() {
    int32_t x = 0x12345678;
    
    printf("Valore: 0x%X\n", x);
    printf("Byte in memoria:\n");
    print_bytes(&x, sizeof(x));
    
    return 0;
}

// Output su x86-64 (Little-Endian):
// Valore: 0x12345678
// Byte in memoria:
// Byte 0: 0x78  ← LSB primo!
// Byte 1: 0x56
// Byte 2: 0x34
// Byte 3: 0x12  ← MSB ultimo!
```

### Metodo 2: Union

```c
#include <stdio.h>
#include <stdint.h>

union View {
    uint32_t value;
    uint8_t bytes[4];
};

int main() {
    union View v;
    v.value = 0x12345678;
    
    printf("Byte[0] = 0x%02X\n", v.bytes[0]);
    printf("Byte[1] = 0x%02X\n", v.bytes[1]);
    printf("Byte[2] = 0x%02X\n", v.bytes[2]);
    printf("Byte[3] = 0x%02X\n", v.bytes[3]);
    
    return 0;
}
```

---

## 3. DETECTARE ENDIANNESS

```c
#include <stdio.h>
#include <stdint.h>

int is_little_endian() {
    uint32_t test = 1;
    uint8_t *byte_ptr = (uint8_t *)&test;
    return byte_ptr[0] == 1;
}

int main() {
    if (is_little_endian()) {
        printf("Sistema: Little-Endian\n");
    } else {
        printf("Sistema: Big-Endian\n");
    }
    return 0;
}

/* SPIEGAZIONE:
 * test = 0x00000001
 * 
 * Little-Endian:
 * byte[0] = 0x01 ← primo byte contiene LSB
 * byte[1] = 0x00
 * byte[2] = 0x00
 * byte[3] = 0x00
 * 
 * Big-Endian:
 * byte[0] = 0x00 ← primo byte contiene MSB
 * byte[1] = 0x00
 * byte[2] = 0x00
 * byte[3] = 0x01
 */
```

---

## 4. NETWORK BYTE ORDER

**Network Byte Order = BIG-ENDIAN**

```c
#include <arpa/inet.h>

// Host to Network (Long)
uint32_t host_value = 0x12345678;
uint32_t network_value = htonl(host_value);

// Network to Host (Long)
uint32_t received = ntohl(network_value);

// Host to Network (Short)
uint16_t port = 8080;
uint16_t network_port = htons(port);

// Network to Host (Short)
uint16_t local_port = ntohs(network_port);

/* FUNZIONI:
 * htonl() - Host TO Network Long (32-bit)
 * htons() - Host TO Network Short (16-bit)
 * ntohl() - Network TO Host Long
 * ntohs() - Network TO Host Short
 *
 * Su Little-Endian: CONVERSIONE
 * Su Big-Endian: NO-OP (già corretto)
 */
```

---

## 5. CONVERSIONE MANUALE

```c
uint32_t swap_endian_32(uint32_t value) {
    return ((value & 0xFF000000) >> 24) |
           ((value & 0x00FF0000) >> 8)  |
           ((value & 0x0000FF00) << 8)  |
           ((value & 0x000000FF) << 24);
}

uint16_t swap_endian_16(uint16_t value) {
    return (value >> 8) | (value << 8);
}

/* ESEMPIO: swap_endian_32(0x12345678)
 * 
 * 1. (0x12345678 & 0xFF000000) >> 24 = 0x00000012
 * 2. (0x12345678 & 0x00FF0000) >> 8  = 0x00003400
 * 3. (0x12345678 & 0x0000FF00) << 8  = 0x00560000
 * 4. (0x12345678 & 0x000000FF) << 24 = 0x78000000
 *
 * OR insieme: 0x78563412
 */
```

---

## 6. PORTABILITÀ

```c
// ❌ NON PORTABILE:
void write_int_to_file(FILE *fp, int value) {
    fwrite(&value, sizeof(int), 1, fp);
    // Endianness dipende dalla piattaforma!
}

// ✅ PORTABILE: Usa sempre Big-Endian per file/rete
void write_int_to_file_portable(FILE *fp, int32_t value) {
    uint32_t network_value = htonl((uint32_t)value);
    fwrite(&network_value, sizeof(uint32_t), 1, fp);
}

void read_int_from_file_portable(FILE *fp, int32_t *value) {
    uint32_t network_value;
    fread(&network_value, sizeof(uint32_t), 1, fp);
    *value = (int32_t)ntohl(network_value);
}
```

---

## 7. FLOATING POINT

```c
#include <stdio.h>
#include <string.h>

void print_float_bytes(float f) {
    unsigned char *bytes = (unsigned char *)&f;
    printf("Float %.2f bytes: ", f);
    for (int i = 0; i < sizeof(float); i++) {
        printf("%02X ", bytes[i]);
    }
    printf("\n");
}

int main() {
    float f = 3.14f;
    print_float_bytes(f);
    
    // Output su Little-Endian:
    // Float 3.14 bytes: C3 F5 48 40
    // 
    // IEEE 754:
    // 01000000 01001000 11110101 11000011 (binario)
    // Ma memorizzato little-endian!
    
    return 0;
}
```

---

## 8. 🔧 ESERCIZI

### Esercizio: Converti 0xAABBCCDD da Little a Big

<details>
<summary>Soluzione</summary>

```
Little-Endian (memoria):
[DD] [CC] [BB] [AA]

Converti:
Byte 0 ↔ Byte 3: DD ↔ AA
Byte 1 ↔ Byte 2: CC ↔ BB

Big-Endian (memoria):
[AA] [BB] [CC] [DD]

Codice:
uint32_t le = 0xAABBCCDD;
uint32_t be = swap_endian_32(le);
// be = 0xDDCCBBAA (come valore)
// ma in memoria big-endian: [AA][BB][CC][DD]
```
</details>

---

[← Allineamento](02_allineamento_padding.md) | [Torna al Modulo](README.md) | [Prossima: Segmenti →](04_segmenti_memoria.md)
