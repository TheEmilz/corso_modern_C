# Programmazione Retro Gaming: Commodore 64 e Amiga 600

## Introduzione alla Programmazione Retro

La programmazione di videogiochi in stile Commodore 64 e Amiga rappresenta un'epoca dove ogni byte contava e la creatività nasceva dalle limitazioni hardware. Programmare a questo livello insegna l'essenza dell'ottimizzazione e del controllo diretto dell'hardware.

### Architettura Commodore 64

**Specifiche Hardware:**
- CPU: MOS 6510 (8-bit, ~1 MHz)
- RAM: 64 KB (effettivi ~38 KB disponibili per programmi)
- Chip grafico: VIC-II (320x200, 16 colori)
- Chip audio: SID (3 voci polifoniche)
- ROM: KERNAL + BASIC (16 KB)

**Memory Map:**
```
$0000-$00FF   Zero Page (256 byte - accesso veloce)
$0100-$01FF   Stack
$0200-$9FFF   RAM libera per programmi
$A000-$BFFF   BASIC ROM / RAM
$C000-$CFFF   RAM
$D000-$DFFF   I/O (VIC-II, SID, CIA)
$E000-$FFFF   KERNAL ROM
```

## Programmazione con Opcode - Livello Base

### Simulazione 6502/6510 in C

Creiamo un emulatore base per comprendere come funzionano gli opcode:

**cpu_6502.h:**
```c
#ifndef CPU_6502_H
#define CPU_6502_H

#include <stdint.h>
#include <stdbool.h>

/* Registri CPU 6502 */
typedef struct {
    uint8_t A;      // Accumulator
    uint8_t X;      // Index Register X
    uint8_t Y;      // Index Register Y
    uint8_t SP;     // Stack Pointer
    uint16_t PC;    // Program Counter
    
    /* Status Register (P) */
    struct {
        bool C : 1;  // Carry
        bool Z : 1;  // Zero
        bool I : 1;  // Interrupt Disable
        bool D : 1;  // Decimal Mode
        bool B : 1;  // Break
        bool U : 1;  // Unused (always 1)
        bool V : 1;  // Overflow
        bool N : 1;  // Negative
    } P;
} CPU6502;

/* Memoria */
#define MEM_SIZE 65536
extern uint8_t memory[MEM_SIZE];

/* Opcode principali */
#define OP_LDA_IMM  0xA9  // Load Accumulator Immediate
#define OP_LDA_ZP   0xA5  // Load Accumulator Zero Page
#define OP_STA_ZP   0x85  // Store Accumulator Zero Page
#define OP_ADC_IMM  0x69  // Add with Carry Immediate
#define OP_SBC_IMM  0xE9  // Subtract with Carry Immediate
#define OP_INX      0xE8  // Increment X
#define OP_DEX      0xCA  // Decrement X
#define OP_JMP_ABS  0x4C  // Jump Absolute
#define OP_JSR      0x20  // Jump to Subroutine
#define OP_RTS      0x60  // Return from Subroutine
#define OP_BNE      0xD0  // Branch if Not Equal
#define OP_BEQ      0xF0  // Branch if Equal
#define OP_NOP      0xEA  // No Operation
#define OP_BRK      0x00  // Break

/* Funzioni CPU */
void cpu_reset(CPU6502 *cpu);
void cpu_execute(CPU6502 *cpu, int cycles);
uint8_t cpu_read_byte(uint16_t address);
void cpu_write_byte(uint16_t address, uint8_t value);

#endif
```

**cpu_6502.c:**
```c
#include <stdio.h>
#include <string.h>
#include "cpu_6502.h"

uint8_t memory[MEM_SIZE];

void cpu_reset(CPU6502 *cpu) {
    cpu->A = 0;
    cpu->X = 0;
    cpu->Y = 0;
    cpu->SP = 0xFF;
    cpu->PC = 0x0600;  // Indirizzo di start tipico
    
    cpu->P.C = 0;
    cpu->P.Z = 0;
    cpu->P.I = 0;
    cpu->P.D = 0;
    cpu->P.B = 0;
    cpu->P.U = 1;
    cpu->P.V = 0;
    cpu->P.N = 0;
}

uint8_t cpu_read_byte(uint16_t address) {
    return memory[address];
}

void cpu_write_byte(uint16_t address, uint8_t value) {
    memory[address] = value;
}

static void set_zero_negative(CPU6502 *cpu, uint8_t value) {
    cpu->P.Z = (value == 0);
    cpu->P.N = (value & 0x80) != 0;
}

void cpu_execute(CPU6502 *cpu, int cycles) {
    while (cycles > 0) {
        uint8_t opcode = cpu_read_byte(cpu->PC++);
        
        switch (opcode) {
            case OP_LDA_IMM: {  // LDA #$nn
                cpu->A = cpu_read_byte(cpu->PC++);
                set_zero_negative(cpu, cpu->A);
                cycles -= 2;
                break;
            }
            
            case OP_LDA_ZP: {  // LDA $nn
                uint8_t addr = cpu_read_byte(cpu->PC++);
                cpu->A = cpu_read_byte(addr);
                set_zero_negative(cpu, cpu->A);
                cycles -= 3;
                break;
            }
            
            case OP_STA_ZP: {  // STA $nn
                uint8_t addr = cpu_read_byte(cpu->PC++);
                cpu_write_byte(addr, cpu->A);
                cycles -= 3;
                break;
            }
            
            case OP_ADC_IMM: {  // ADC #$nn
                uint8_t value = cpu_read_byte(cpu->PC++);
                uint16_t result = cpu->A + value + cpu->P.C;
                cpu->P.C = (result > 0xFF);
                cpu->P.V = ((cpu->A ^ result) & (value ^ result) & 0x80) != 0;
                cpu->A = result & 0xFF;
                set_zero_negative(cpu, cpu->A);
                cycles -= 2;
                break;
            }
            
            case OP_INX:  // INX
                cpu->X++;
                set_zero_negative(cpu, cpu->X);
                cycles -= 2;
                break;
            
            case OP_DEX:  // DEX
                cpu->X--;
                set_zero_negative(cpu, cpu->X);
                cycles -= 2;
                break;
            
            case OP_JMP_ABS: {  // JMP $nnnn
                uint16_t addr = cpu_read_byte(cpu->PC++);
                addr |= (cpu_read_byte(cpu->PC++) << 8);
                cpu->PC = addr;
                cycles -= 3;
                break;
            }
            
            case OP_BNE: {  // BNE $nn
                int8_t offset = cpu_read_byte(cpu->PC++);
                if (!cpu->P.Z) {
                    cpu->PC += offset;
                }
                cycles -= 2;
                break;
            }
            
            case OP_NOP:  // NOP
                cycles -= 2;
                break;
            
            case OP_BRK:  // BRK
                printf("BREAK at $%04X\n", cpu->PC - 1);
                return;
            
            default:
                printf("Unknown opcode: $%02X at $%04X\n", opcode, cpu->PC - 1);
                return;
        }
    }
}
```

**Esempio: Mini-gioco Retro Style**

**retro_game.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include "cpu_6502.h"

/* Aree di memoria per il gioco */
#define PLAYER_X     0x20
#define PLAYER_Y     0x21
#define ENEMY_X      0x22
#define ENEMY_Y      0x23
#define SCORE        0x24
#define GAME_STATE   0x25

/* Stati gioco */
#define STATE_PLAYING 0x01
#define STATE_GAMEOVER 0x02

/* Programma in "assembly" (opcode) */
void load_game_program(void) {
    uint16_t pc = 0x0600;
    
    // Inizializzazione
    memory[pc++] = OP_LDA_IMM;    // LDA #$05
    memory[pc++] = 0x05;
    memory[pc++] = OP_STA_ZP;     // STA PLAYER_X
    memory[pc++] = PLAYER_X;
    
    memory[pc++] = OP_LDA_IMM;    // LDA #$05
    memory[pc++] = 0x05;
    memory[pc++] = OP_STA_ZP;     // STA PLAYER_Y
    memory[pc++] = PLAYER_Y;
    
    memory[pc++] = OP_LDA_IMM;    // LDA #$00
    memory[pc++] = 0x00;
    memory[pc++] = OP_STA_ZP;     // STA SCORE
    memory[pc++] = SCORE;
    
    memory[pc++] = OP_LDA_IMM;    // LDA #STATE_PLAYING
    memory[pc++] = STATE_PLAYING;
    memory[pc++] = OP_STA_ZP;     // STA GAME_STATE
    memory[pc++] = GAME_STATE;
    
    // Loop principale
    memory[pc++] = OP_NOP;        // Placeholder per logica gioco
    memory[pc++] = OP_JMP_ABS;    // JMP loop
    memory[pc++] = 0x0E;
    memory[pc++] = 0x06;
    
    memory[pc++] = OP_BRK;        // Fine
}

/* Visualizzazione ASCII del gioco */
void render_game(void) {
    system("clear");  // Linux/Mac
    
    int px = memory[PLAYER_X];
    int py = memory[PLAYER_Y];
    int ex = memory[ENEMY_X];
    int ey = memory[ENEMY_Y];
    int score = memory[SCORE];
    
    printf("╔════════════════════╗\n");
    
    for (int y = 0; y < 10; y++) {
        printf("║");
        for (int x = 0; x < 20; x++) {
            if (x == px && y == py) {
                printf("@");  // Player
            } else if (x == ex && y == ey) {
                printf("X");  // Enemy
            } else {
                printf(" ");
            }
        }
        printf("║\n");
    }
    
    printf("╚════════════════════╝\n");
    printf("Score: %d | WASD=Move Q=Quit\n", score);
}

int main(void) {
    CPU6502 cpu;
    
    printf("=== Retro Game in stile C64 ===\n");
    printf("Emulazione CPU 6502 con opcode\n\n");
    
    cpu_reset(&cpu);
    load_game_program();
    
    // Posizione iniziale nemico
    memory[ENEMY_X] = 15;
    memory[ENEMY_Y] = 5;
    
    render_game();
    
    printf("\nProgramma caricato all'indirizzo $0600\n");
    printf("CPU inizializzata. PC=$%04X\n", cpu.PC);
    printf("\nPremere INVIO per eseguire...");
    getchar();
    
    // Esegui alcuni cicli
    cpu_execute(&cpu, 20);
    
    printf("\nStato CPU dopo esecuzione:\n");
    printf("A=$%02X X=$%02X Y=$%02X\n", cpu.A, cpu.X, cpu.Y);
    printf("PC=$%04X SP=$%02X\n", cpu.PC, cpu.SP);
    printf("Flags: N=%d V=%d Z=%d C=%d\n", 
           cpu.P.N, cpu.P.V, cpu.P.Z, cpu.P.C);
    
    printf("\nMemoria Zero Page:\n");
    printf("PLAYER_X ($%02X) = $%02X\n", PLAYER_X, memory[PLAYER_X]);
    printf("PLAYER_Y ($%02X) = $%02X\n", PLAYER_Y, memory[PLAYER_Y]);
    printf("SCORE ($%02X) = $%02X\n", SCORE, memory[SCORE]);
    printf("GAME_STATE ($%02X) = $%02X\n", GAME_STATE, memory[GAME_STATE]);
    
    return 0;
}
```

## Programmazione Amiga 600

### Architettura Amiga

**Specifiche:**
- CPU: Motorola 68000 (16/32-bit, 7.14 MHz)
- RAM: 1 MB (espandibile)
- Chip grafico: OCS (640x256, 32 colori da 4096)
- Audio: 4 canali stereo 8-bit
- Copiloti grafici: Blitter, Copper

### Emulazione Base 68000

**m68k_simple.c:**
```c
#include <stdio.h>
#include <stdint.h>

/* Registri 68000 */
typedef struct {
    uint32_t D[8];  // Data registers
    uint32_t A[7];  // Address registers
    uint32_t SP;    // Stack Pointer (A7)
    uint32_t PC;    // Program Counter
    
    uint16_t SR;    // Status Register
} M68K;

/* Opcode base 68000 */
#define MOVE_L  0x20  // MOVE.L
#define ADD_L   0xD0  // ADD.L
#define SUB_L   0x90  // SUB.L
#define MOVEA_L 0x20  // MOVEA.L

void m68k_init(M68K *cpu) {
    for (int i = 0; i < 8; i++) {
        cpu->D[i] = 0;
    }
    for (int i = 0; i < 7; i++) {
        cpu->A[i] = 0;
    }
    cpu->SP = 0x1000;
    cpu->PC = 0x0000;
    cpu->SR = 0x2700;  // Supervisor mode
}

void m68k_example(void) {
    M68K cpu;
    m68k_init(&cpu);
    
    printf("=== Motorola 68000 Simulator ===\n\n");
    
    /* Simula: MOVE.L #$1234,D0 */
    cpu.D[0] = 0x1234;
    printf("MOVE.L #$1234,D0\n");
    printf("D0 = $%08X\n\n", cpu.D[0]);
    
    /* Simula: ADD.L #$5678,D0 */
    cpu.D[0] += 0x5678;
    printf("ADD.L #$5678,D0\n");
    printf("D0 = $%08X\n\n", cpu.D[0]);
    
    /* Simula: MOVEA.L #$10000,A0 */
    cpu.A[0] = 0x10000;
    printf("MOVEA.L #$10000,A0\n");
    printf("A0 = $%08X\n\n", cpu.A[0]);
}

int main(void) {
    m68k_example();
    return 0;
}
```

## Sprite e Grafica Retro

### Sistema Sprite Base (stile C64)

**sprite_system.c:**
```c
#include <stdio.h>
#include <stdint.h>
#include <string.h>

#define SPRITE_WIDTH  24
#define SPRITE_HEIGHT 21

/* Sprite data (3 byte x 21 righe = 63 byte) */
typedef uint8_t Sprite[63];

/* Esempio: Sprite astronave */
Sprite spaceship = {
    0x00,0x18,0x00,  // ........ ...##... ........
    0x00,0x3C,0x00,  // ........ ..####.. ........
    0x00,0x7E,0x00,  // ........ .######. ........
    0x00,0xFF,0x00,  // ........ ######## ........
    0x01,0xFF,0x80,  // .......# ######## #.......
    0x03,0xFF,0xC0,  // ......## ######## ##......
    0x07,0xFF,0xE0,  // .....### ######## ###.....
    0x0F,0xFF,0xF0,  // ....#### ######## ####....
    0x1F,0xFF,0xF8,  // ...##### ######## #####...
    0x3F,0xFF,0xFC,  // ..###### ######## ######..
    0x7F,0xFF,0xFE,  // .####### ######## #######.
    0xFF,0xFF,0xFF,  // ######## ######## ########
    0xFF,0xFF,0xFF,  // ######## ######## ########
    0x7F,0xFF,0xFE,  // .####### ######## #######.
    0x3F,0xFF,0xFC,  // ..###### ######## ######..
    0x1F,0xFF,0xF8,  // ...##### ######## #####...
    0x0F,0xFF,0xF0,  // ....#### ######## ####....
    0x07,0x81,0xE0,  // .....### #......# ###.....
    0x03,0x00,0xC0,  // ......## ........ ##......
    0x01,0x00,0x80,  // .......# ........ #.......
    0x00,0x00,0x00,  // ........ ........ ........
};

void draw_sprite(Sprite sprite, int x, int y) {
    printf("\nSprite at (%d, %d):\n", x, y);
    
    for (int row = 0; row < SPRITE_HEIGHT; row++) {
        int idx = row * 3;
        uint32_t line = (sprite[idx] << 16) | (sprite[idx+1] << 8) | sprite[idx+2];
        
        for (int col = 0; col < SPRITE_WIDTH; col++) {
            if (line & (1 << (23 - col))) {
                printf("█");
            } else {
                printf(" ");
            }
        }
        printf("\n");
    }
}

/* Collision detection */
int sprite_collision(int x1, int y1, int x2, int y2) {
    int dx = (x1 > x2) ? (x1 - x2) : (x2 - x1);
    int dy = (y1 > y2) ? (y1 - y2) : (y2 - y1);
    
    return (dx < SPRITE_WIDTH && dy < SPRITE_HEIGHT);
}

int main(void) {
    printf("=== Sistema Sprite Retro ===\n");
    
    draw_sprite(spaceship, 10, 5);
    
    printf("\n\nCollision Test:\n");
    printf("Sprite A at (10, 10)\n");
    printf("Sprite B at (20, 15)\n");
    
    if (sprite_collision(10, 10, 20, 15)) {
        printf("COLLISION!\n");
    } else {
        printf("No collision\n");
    }
    
    return 0;
}
```

## Ottimizzazioni Retro

### Tecniche di Ottimizzazione C64/Amiga

```c
/* 1. Unrolled loops - risparmia controlli e jump */
void clear_screen_fast(uint8_t *screen) {
    // Invece di loop
    for (int i = 0; i < 1000; i++) screen[i] = 0x20;
    
    // Unroll (esempio parziale)
    uint8_t *p = screen;
    for (int i = 0; i < 125; i++) {
        p[0] = p[1] = p[2] = p[3] = 0x20;
        p[4] = p[5] = p[6] = p[7] = 0x20;
        p += 8;
    }
}

/* 2. Lookup tables - evita calcoli costosi */
static uint8_t sin_table[256];  // Tabella precalcolata

void init_sin_table(void) {
    for (int i = 0; i < 256; i++) {
        sin_table[i] = (uint8_t)(127.5 + 127.5 * sin(i * 2 * M_PI / 256));
    }
}

uint8_t fast_sin(uint8_t angle) {
    return sin_table[angle];  // Lookup invece di calcolo
}

/* 3. Fixed-point arithmetic - evita FPU lenta/assente */
typedef int16_t fixed16;  // 8.8 fixed point

fixed16 fixed_mul(fixed16 a, fixed16 b) {
    return (int16_t)(((int32_t)a * (int32_t)b) >> 8);
}

fixed16 fixed_div(fixed16 a, fixed16 b) {
    return (int16_t)(((int32_t)a << 8) / b);
}

/* 4. Bit tricks - operazioni veloci */
#define FAST_MOD_256(x)  ((x) & 0xFF)       // x % 256
#define FAST_DIV_256(x)  ((x) >> 8)         // x / 256
#define FAST_MUL_256(x)  ((x) << 8)         // x * 256
#define IS_POWER_OF_2(x) (((x) & ((x)-1)) == 0)
```

## Mini-Progetto: Space Invaders Style

**space_invaders.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>

#define WIDTH 40
#define HEIGHT 20
#define MAX_BULLETS 10
#define MAX_ENEMIES 15

typedef struct {
    int x, y;
    int active;
} GameObject;

typedef struct {
    GameObject player;
    GameObject bullets[MAX_BULLETS];
    GameObject enemies[MAX_ENEMIES];
    int score;
    int game_over;
} Game;

void game_init(Game *g) {
    g->player.x = WIDTH / 2;
    g->player.y = HEIGHT - 2;
    g->player.active = 1;
    
    for (int i = 0; i < MAX_BULLETS; i++) {
        g->bullets[i].active = 0;
    }
    
    // Inizializza nemici in formazione
    for (int i = 0; i < MAX_ENEMIES; i++) {
        g->enemies[i].x = 5 + (i % 5) * 6;
        g->enemies[i].y = 2 + (i / 5) * 2;
        g->enemies[i].active = 1;
    }
    
    g->score = 0;
    g->game_over = 0;
}

void game_render(Game *g) {
    system("clear");
    
    char screen[HEIGHT][WIDTH + 1];
    
    // Pulisci schermo
    for (int y = 0; y < HEIGHT; y++) {
        for (int x = 0; x < WIDTH; x++) {
            screen[y][x] = ' ';
        }
        screen[y][WIDTH] = '\0';
    }
    
    // Disegna player
    if (g->player.active) {
        screen[g->player.y][g->player.x] = 'A';
    }
    
    // Disegna bullets
    for (int i = 0; i < MAX_BULLETS; i++) {
        if (g->bullets[i].active) {
            int x = g->bullets[i].x;
            int y = g->bullets[i].y;
            if (y >= 0 && y < HEIGHT && x >= 0 && x < WIDTH) {
                screen[y][x] = '|';
            }
        }
    }
    
    // Disegna nemici
    for (int i = 0; i < MAX_ENEMIES; i++) {
        if (g->enemies[i].active) {
            int x = g->enemies[i].x;
            int y = g->enemies[i].y;
            if (y >= 0 && y < HEIGHT && x >= 0 && x < WIDTH) {
                screen[y][x] = 'W';
            }
        }
    }
    
    // Stampa
    for (int y = 0; y < HEIGHT; y++) {
        printf("%s\n", screen[y]);
    }
    
    printf("\nScore: %d | A/D=Move SPACE=Fire Q=Quit\n", g->score);
}

void game_update(Game *g) {
    // Muovi bullets
    for (int i = 0; i < MAX_BULLETS; i++) {
        if (g->bullets[i].active) {
            g->bullets[i].y--;
            if (g->bullets[i].y < 0) {
                g->bullets[i].active = 0;
            }
        }
    }
    
    // Controlla collisioni
    for (int i = 0; i < MAX_BULLETS; i++) {
        if (!g->bullets[i].active) continue;
        
        for (int j = 0; j < MAX_ENEMIES; j++) {
            if (!g->enemies[j].active) continue;
            
            if (g->bullets[i].x == g->enemies[j].x && 
                g->bullets[i].y == g->enemies[j].y) {
                g->bullets[i].active = 0;
                g->enemies[j].active = 0;
                g->score += 10;
            }
        }
    }
    
    // Controlla vittoria
    int enemies_left = 0;
    for (int i = 0; i < MAX_ENEMIES; i++) {
        if (g->enemies[i].active) enemies_left++;
    }
    
    if (enemies_left == 0) {
        g->game_over = 1;
    }
}

void game_shoot(Game *g) {
    for (int i = 0; i < MAX_BULLETS; i++) {
        if (!g->bullets[i].active) {
            g->bullets[i].x = g->player.x;
            g->bullets[i].y = g->player.y - 1;
            g->bullets[i].active = 1;
            break;
        }
    }
}

int main(void) {
    Game game;
    
    printf("=== SPACE INVADERS (Retro Style) ===\n");
    printf("Implementato con tecniche C64/Amiga\n\n");
    printf("Premere INVIO per iniziare...");
    getchar();
    
    game_init(&game);
    
    // Simulazione di alcune iterazioni
    for (int frame = 0; frame < 5 && !game.game_over; frame++) {
        game_render(&game);
        
        // Simula input (in un vero gioco useremmo input non-blocking)
        if (frame == 1) {
            game_shoot(&game);
        }
        
        game_update(&game);
        usleep(500000);  // 0.5 secondi
    }
    
    printf("\nDemo completata!\n");
    printf("In un gioco reale implementeresti:\n");
    printf("- Input non-blocking (ncurses o simile)\n");
    printf("- Movimento nemici\n");
    printf("- Nemici che sparano\n");
    printf("- Sound effects\n");
    printf("- Multiple lives\n");
    
    return 0;
}
```

## Esercizi Retro

1. **Estendi l'emulatore 6502** con più opcode (CMP, BPL, BMI, etc.)
2. **Crea un sistema di scroll** verticale o orizzontale
3. **Implementa una starfield animata** (effetto 3D)
4. **Crea un semplice synth audio** con forme d'onda (vedi sezione audio)
5. **Implementa un sistema di tiles** per mappe di gioco
6. **Crea un effetto raster bar** (cambia colori per linea di scan)

---

[← Torna a Low-Level](README.md) | [Audio Synthesis →](audio_synthesis.md)
