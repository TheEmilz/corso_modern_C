# Programmazione Hardware: Arduino e Raspberry Pi

## Introduzione alla Programmazione Embedded

La programmazione hardware rappresenta l'applicazione diretta dei concetti di basso livello: manipolazione di registri, timing preciso, gestione di interrupt, e comunicazione con periferiche fisiche.

### Arduino vs Raspberry Pi

**Arduino:**
- Microcontrollore (AVR/ARM)
- Bare-metal programming (no OS)
- Real-time deterministic
- Basso consumo energetico
- Perfetto per controllo hardware diretto

**Raspberry Pi:**
- Microprocessore (ARM)
- Linux OS
- Multi-tasking
- GPIO + interfacce avanzate
- Perfetto per progetti complessi con networking

## Programmazione Arduino in C

### Anatomia di un Programma Arduino

Anche se usiamo l'IDE Arduino, sotto usa C/C++ compilato con avr-gcc.

**blink_raw.c:**
```c
/* Blink LED senza framework Arduino - C puro */
#include <avr/io.h>
#include <util/delay.h>

#define LED_PIN 5  // Pin 13 su Arduino Uno (PORTB5)

int main(void) {
    // Configura pin come output
    DDRB |= (1 << LED_PIN);  // Data Direction Register B
    
    while (1) {
        // Accendi LED
        PORTB |= (1 << LED_PIN);
        _delay_ms(1000);
        
        // Spegni LED
        PORTB &= ~(1 << LED_PIN);
        _delay_ms(1000);
    }
    
    return 0;
}
```

### Manipolazione Registri Hardware

**Registri fondamentali AVR:**

```c
/* Pin I/O - ogni porta ha 3 registri */
// DDRx  - Data Direction Register (0=input, 1=output)
// PORTx - Port Data Register (scrittura su pin output)
// PINx  - Port Input Register (lettura pin input)

/* Esempio: Configurazione PORTD */
void configure_portd(void) {
    // Pin 0-3 input, 4-7 output
    DDRD = 0b11110000;
    
    // Abilita pull-up su input
    PORTD |= 0b00001111;
}

/* Manipolazione bit-wise */
#define SET_BIT(reg, bit)     ((reg) |= (1 << (bit)))
#define CLEAR_BIT(reg, bit)   ((reg) &= ~(1 << (bit)))
#define TOGGLE_BIT(reg, bit)  ((reg) ^= (1 << (bit)))
#define READ_BIT(reg, bit)    (((reg) >> (bit)) & 1)

/* Esempio uso */
void led_on(uint8_t pin) {
    SET_BIT(PORTB, pin);
}

void led_off(uint8_t pin) {
    CLEAR_BIT(PORTB, pin);
}

void led_toggle(uint8_t pin) {
    TOGGLE_BIT(PORTB, pin);
}

int is_button_pressed(uint8_t pin) {
    return !READ_BIT(PIND, pin);  // Active low
}
```

### Interrupt Service Routines (ISR)

**interrupt_example.c:**
```c
#include <avr/io.h>
#include <avr/interrupt.h>
#include <util/delay.h>

volatile uint16_t timer_counter = 0;
volatile uint8_t button_pressed = 0;

/* Timer1 compare match interrupt */
ISR(TIMER1_COMPA_vect) {
    timer_counter++;
    PORTB ^= (1 << PB5);  // Toggle LED
}

/* External interrupt 0 (pin D2) */
ISR(INT0_vect) {
    button_pressed = 1;
}

void setup_timer1(void) {
    // Modalità CTC (Clear Timer on Compare)
    TCCR1B |= (1 << WGM12);
    
    // Prescaler 1024
    TCCR1B |= (1 << CS12) | (1 << CS10);
    
    // Compare value per 1 Hz (16MHz / 1024 / 15625 ≈ 1Hz)
    OCR1A = 15625;
    
    // Abilita interrupt compare match
    TIMSK1 |= (1 << OCIE1A);
}

void setup_external_interrupt(void) {
    // Abilita INT0 (pin D2)
    EIMSK |= (1 << INT0);
    
    // Trigger su falling edge
    EICRA |= (1 << ISC01);
    
    // Abilita pull-up
    PORTD |= (1 << PD2);
}

int main(void) {
    DDRB |= (1 << PB5);  // LED output
    
    setup_timer1();
    setup_external_interrupt();
    
    sei();  // Abilita interrupts globali
    
    while (1) {
        if (button_pressed) {
            button_pressed = 0;
            // Fai qualcosa...
            _delay_ms(50);  // Debounce
        }
    }
    
    return 0;
}
```

### ADC (Analog to Digital Converter)

**adc_example.c:**
```c
#include <avr/io.h>

void adc_init(void) {
    // Reference voltage = AVCC
    ADMUX = (1 << REFS0);
    
    // Prescaler 128 (16MHz/128 = 125kHz, nel range 50-200kHz)
    ADCSRA = (1 << ADEN) | (1 << ADPS2) | (1 << ADPS1) | (1 << ADPS0);
}

uint16_t adc_read(uint8_t channel) {
    // Seleziona canale (0-7)
    ADMUX = (ADMUX & 0xF0) | (channel & 0x0F);
    
    // Avvia conversione
    ADCSRA |= (1 << ADSC);
    
    // Aspetta completamento
    while (ADCSRA & (1 << ADSC));
    
    return ADC;  // Leggi risultato (10-bit)
}

/* Esempio: Lettura potenziometro */
void read_potentiometer(void) {
    adc_init();
    
    while (1) {
        uint16_t value = adc_read(0);  // Canale A0
        
        // Converti a voltage (0-5V)
        float voltage = value * (5.0f / 1023.0f);
        
        // Usa il valore...
        _delay_ms(100);
    }
}
```

### PWM (Pulse Width Modulation)

**pwm_example.c:**
```c
#include <avr/io.h>

/* Configura PWM su pin 9 (OC1A) e 10 (OC1B) */
void pwm_init(void) {
    // Fast PWM, 10-bit
    TCCR1A = (1 << WGM11) | (1 << WGM10);
    TCCR1B = (1 << WGM12);
    
    // Non-inverting mode
    TCCR1A |= (1 << COM1A1) | (1 << COM1B1);
    
    // Prescaler 64
    TCCR1B |= (1 << CS11) | (1 << CS10);
    
    // Pin come output
    DDRB |= (1 << PB1) | (1 << PB2);
}

void pwm_set_duty(uint8_t channel, uint16_t duty) {
    // duty: 0-1023 (10-bit)
    if (channel == 0) {
        OCR1A = duty;
    } else {
        OCR1B = duty;
    }
}

/* Esempio: Fading LED */
void led_fade(void) {
    pwm_init();
    
    while (1) {
        // Fade in
        for (uint16_t i = 0; i < 1024; i++) {
            pwm_set_duty(0, i);
            _delay_ms(2);
        }
        
        // Fade out
        for (uint16_t i = 1023; i > 0; i--) {
            pwm_set_duty(0, i);
            _delay_ms(2);
        }
    }
}
```

## Comunicazione Seriale

### UART/Serial

**uart.h:**
```c
#ifndef UART_H
#define UART_H

#include <avr/io.h>

#define BAUD 9600
#define UBRR_VALUE ((F_CPU/16/BAUD) - 1)

void uart_init(void);
void uart_transmit(uint8_t data);
uint8_t uart_receive(void);
void uart_print_string(const char *str);
void uart_print_number(uint16_t num);

#endif
```

**uart.c:**
```c
#include "uart.h"
#include <stdio.h>

void uart_init(void) {
    // Set baud rate
    UBRR0H = (uint8_t)(UBRR_VALUE >> 8);
    UBRR0L = (uint8_t)UBRR_VALUE;
    
    // Enable receiver and transmitter
    UCSR0B = (1 << RXEN0) | (1 << TXEN0);
    
    // Set frame format: 8 data bits, 1 stop bit
    UCSR0C = (1 << UCSZ01) | (1 << UCSZ00);
}

void uart_transmit(uint8_t data) {
    // Wait for empty transmit buffer
    while (!(UCSR0A & (1 << UDRE0)));
    
    // Put data into buffer, sends the data
    UDR0 = data;
}

uint8_t uart_receive(void) {
    // Wait for data to be received
    while (!(UCSR0A & (1 << RXC0)));
    
    // Get and return received data from buffer
    return UDR0;
}

void uart_print_string(const char *str) {
    while (*str) {
        uart_transmit(*str++);
    }
}

void uart_print_number(uint16_t num) {
    char buffer[6];
    sprintf(buffer, "%u", num);
    uart_print_string(buffer);
}
```

### I2C/TWI (Two Wire Interface)

**i2c.c:**
```c
#include <avr/io.h>

#define F_SCL 100000UL  // 100kHz
#define TWI_FREQ ((F_CPU/F_SCL)-16)/2

void i2c_init(void) {
    TWBR = TWI_FREQ;
    TWSR = 0x00;  // Prescaler = 1
}

void i2c_start(void) {
    TWCR = (1 << TWINT) | (1 << TWSTA) | (1 << TWEN);
    while (!(TWCR & (1 << TWINT)));
}

void i2c_stop(void) {
    TWCR = (1 << TWINT) | (1 << TWSTO) | (1 << TWEN);
}

void i2c_write(uint8_t data) {
    TWDR = data;
    TWCR = (1 << TWINT) | (1 << TWEN);
    while (!(TWCR & (1 << TWINT)));
}

uint8_t i2c_read_ack(void) {
    TWCR = (1 << TWINT) | (1 << TWEN) | (1 << TWEA);
    while (!(TWCR & (1 << TWINT)));
    return TWDR;
}

uint8_t i2c_read_nack(void) {
    TWCR = (1 << TWINT) | (1 << TWEN);
    while (!(TWCR & (1 << TWINT)));
    return TWDR;
}

/* Esempio: Lettura da sensore I2C */
uint8_t read_sensor_i2c(uint8_t device_addr, uint8_t reg_addr) {
    uint8_t data;
    
    i2c_start();
    i2c_write(device_addr << 1);      // Write mode
    i2c_write(reg_addr);               // Register address
    
    i2c_start();                       // Repeated start
    i2c_write((device_addr << 1) | 1); // Read mode
    data = i2c_read_nack();
    i2c_stop();
    
    return data;
}
```

### SPI (Serial Peripheral Interface)

**spi.c:**
```c
#include <avr/io.h>

void spi_init(void) {
    // Set MOSI, SCK, and SS as output
    DDRB |= (1 << PB3) | (1 << PB5) | (1 << PB2);
    
    // Enable SPI, Master mode, clock rate fck/16
    SPCR = (1 << SPE) | (1 << MSTR) | (1 << SPR0);
}

uint8_t spi_transfer(uint8_t data) {
    // Start transmission
    SPDR = data;
    
    // Wait for transmission complete
    while (!(SPSR & (1 << SPIF)));
    
    return SPDR;
}

/* Esempio: Controllo display TFT via SPI */
void tft_write_command(uint8_t cmd) {
    PORTB &= ~(1 << PB2);  // SS low (CS)
    PORTD &= ~(1 << PD7);  // DC low (command mode)
    
    spi_transfer(cmd);
    
    PORTB |= (1 << PB2);   // SS high
}

void tft_write_data(uint8_t data) {
    PORTB &= ~(1 << PB2);  // SS low
    PORTD |= (1 << PD7);   // DC high (data mode)
    
    spi_transfer(data);
    
    PORTB |= (1 << PB2);   // SS high
}
```

## Progetto Completo Arduino: Stazione Meteo

**weather_station.c:**
```c
#include <avr/io.h>
#include <avr/interrupt.h>
#include <util/delay.h>
#include "uart.h"

/* DHT22 - Sensore temperatura/umidità */
typedef struct {
    float temperature;
    float humidity;
} DHT22_Data;

/* BMP180 - Sensore pressione */
typedef struct {
    float pressure;
    float altitude;
} BMP180_Data;

/* Simula lettura DHT22 */
DHT22_Data read_dht22(void) {
    DHT22_Data data;
    
    // In un'implementazione reale:
    // 1. Invia start signal
    // 2. Attendi risposta
    // 3. Leggi 40 bit di dati
    // 4. Verifica checksum
    
    // Simulato per esempio
    data.temperature = 22.5f;
    data.humidity = 65.0f;
    
    return data;
}

/* Simula lettura BMP180 via I2C */
BMP180_Data read_bmp180(void) {
    BMP180_Data data;
    
    // In implementazione reale:
    // Leggi calibration data
    // Leggi raw pressure e temperature
    // Applica formule di compensazione
    
    data.pressure = 1013.25f;  // hPa
    data.altitude = 100.0f;    // metri
    
    return data;
}

/* Display LCD (16x2) via I2C */
void lcd_init(void) {
    i2c_init();
    _delay_ms(50);
    
    // Inizializzazione LCD...
}

void lcd_print(const char *str) {
    // Invia caratteri via I2C...
}

/* Main program */
int main(void) {
    uart_init();
    lcd_init();
    
    uart_print_string("Weather Station v1.0\r\n");
    
    while (1) {
        DHT22_Data dht = read_dht22();
        BMP180_Data bmp = read_bmp180();
        
        // Invia dati via UART
        uart_print_string("Temp: ");
        uart_print_number((uint16_t)(dht.temperature * 10));
        uart_print_string(" Humidity: ");
        uart_print_number((uint16_t)dht.humidity);
        uart_print_string(" Pressure: ");
        uart_print_number((uint16_t)bmp.pressure);
        uart_print_string("\r\n");
        
        // Aggiorna LCD
        // lcd_print("T:22.5C H:65%");
        
        _delay_ms(2000);  // Aggiorna ogni 2 secondi
    }
    
    return 0;
}
```

## Programmazione Raspberry Pi in C

### Accesso GPIO con /dev/mem

**rpi_gpio.h:**
```c
#ifndef RPI_GPIO_H
#define RPI_GPIO_H

#include <stdint.h>

// BCM2835 Peripheral base
#define BCM2835_PERI_BASE   0x3F000000  // Pi 2/3
#define GPIO_BASE           (BCM2835_PERI_BASE + 0x200000)

// GPIO registers
#define GPFSEL0     0   // Function Select
#define GPSET0      7   // Pin Output Set
#define GPCLR0      10  // Pin Output Clear
#define GPLEV0      13  // Pin Level

// GPIO modes
#define GPIO_INPUT  0
#define GPIO_OUTPUT 1

int gpio_init(void);
void gpio_set_mode(int pin, int mode);
void gpio_write(int pin, int value);
int gpio_read(int pin);
void gpio_cleanup(void);

#endif
```

**rpi_gpio.c:**
```c
#include "rpi_gpio.h"
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <unistd.h>

#define PAGE_SIZE 4096
#define BLOCK_SIZE 4096

static volatile uint32_t *gpio;
static int mem_fd;

int gpio_init(void) {
    // Open /dev/mem
    if ((mem_fd = open("/dev/mem", O_RDWR | O_SYNC)) < 0) {
        perror("Failed to open /dev/mem");
        return -1;
    }
    
    // Map GPIO
    void *gpio_map = mmap(
        NULL,
        BLOCK_SIZE,
        PROT_READ | PROT_WRITE,
        MAP_SHARED,
        mem_fd,
        GPIO_BASE
    );
    
    if (gpio_map == MAP_FAILED) {
        perror("mmap failed");
        return -1;
    }
    
    gpio = (volatile uint32_t *)gpio_map;
    return 0;
}

void gpio_set_mode(int pin, int mode) {
    int reg = pin / 10;
    int shift = (pin % 10) * 3;
    
    if (mode == GPIO_OUTPUT) {
        gpio[GPFSEL0 + reg] = 
            (gpio[GPFSEL0 + reg] & ~(7 << shift)) | (1 << shift);
    } else {
        gpio[GPFSEL0 + reg] &= ~(7 << shift);
    }
}

void gpio_write(int pin, int value) {
    if (value) {
        gpio[GPSET0] = 1 << pin;
    } else {
        gpio[GPCLR0] = 1 << pin;
    }
}

int gpio_read(int pin) {
    return (gpio[GPLEV0] & (1 << pin)) ? 1 : 0;
}

void gpio_cleanup(void) {
    munmap((void *)gpio, BLOCK_SIZE);
    close(mem_fd);
}
```

### WiringPi Alternative (libreria semplificata)

**blink_rpi.c:**
```c
#include <stdio.h>
#include <unistd.h>
#include "rpi_gpio.h"

#define LED_PIN 17  // GPIO 17 (Physical pin 11)

int main(void) {
    printf("Raspberry Pi GPIO Blink\n");
    
    if (gpio_init() < 0) {
        fprintf(stderr, "Failed to initialize GPIO\n");
        return 1;
    }
    
    gpio_set_mode(LED_PIN, GPIO_OUTPUT);
    
    for (int i = 0; i < 10; i++) {
        gpio_write(LED_PIN, 1);
        printf("LED ON\n");
        sleep(1);
        
        gpio_write(LED_PIN, 0);
        printf("LED OFF\n");
        sleep(1);
    }
    
    gpio_cleanup();
    return 0;
}
```

### PWM Hardware su Raspberry Pi

**rpi_pwm.c:**
```c
#include <stdint.h>
#include <sys/mman.h>
#include <fcntl.h>

#define PWM_BASE    (BCM2835_PERI_BASE + 0x20C000)
#define CLOCK_BASE  (BCM2835_PERI_BASE + 0x101000)

#define PWM_CONTROL 0
#define PWM_STATUS  1
#define PWM0_RANGE  4
#define PWM0_DATA   5

typedef struct {
    volatile uint32_t *pwm;
    volatile uint32_t *clk;
} PWM;

PWM pwm_init(void) {
    int mem_fd = open("/dev/mem", O_RDWR | O_SYNC);
    
    void *pwm_map = mmap(NULL, 4096, PROT_READ | PROT_WRITE,
                         MAP_SHARED, mem_fd, PWM_BASE);
    void *clk_map = mmap(NULL, 4096, PROT_READ | PROT_WRITE,
                         MAP_SHARED, mem_fd, CLOCK_BASE);
    
    PWM p;
    p.pwm = (volatile uint32_t *)pwm_map;
    p.clk = (volatile uint32_t *)clk_map;
    
    // Configure PWM clock
    p.clk[40] = 0x5A000000 | (1 << 5);  // Stop clock
    usleep(10);
    p.clk[41] = 0x5A000000 | (2 << 12); // Source = oscillator
    p.clk[40] = 0x5A000010 | (1 << 4);  // Enable clock
    
    // Configure PWM
    p.pwm[PWM_CONTROL] = 0;
    p.pwm[PWM0_RANGE] = 1024;  // Range (resolution)
    p.pwm[PWM_CONTROL] = 0x81;  // Enable PWM, use M/S algorithm
    
    return p;
}

void pwm_set_duty(PWM *p, uint32_t duty) {
    p->pwm[PWM0_DATA] = duty;  // 0-1024
}
```

### Comunicazione SPI su Raspberry Pi

**rpi_spi.c:**
```c
#include <stdint.h>
#include <unistd.h>
#include <stdio.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/spi/spidev.h>

static int spi_fd;
static uint8_t spi_mode = SPI_MODE_0;
static uint8_t spi_bits = 8;
static uint32_t spi_speed = 1000000;  // 1 MHz

int spi_init(const char *device) {
    spi_fd = open(device, O_RDWR);
    if (spi_fd < 0) {
        perror("Can't open SPI device");
        return -1;
    }
    
    // Set SPI mode
    if (ioctl(spi_fd, SPI_IOC_WR_MODE, &spi_mode) < 0) {
        perror("Can't set SPI mode");
        return -1;
    }
    
    // Set bits per word
    if (ioctl(spi_fd, SPI_IOC_WR_BITS_PER_WORD, &spi_bits) < 0) {
        perror("Can't set bits per word");
        return -1;
    }
    
    // Set max speed
    if (ioctl(spi_fd, SPI_IOC_WR_MAX_SPEED_HZ, &spi_speed) < 0) {
        perror("Can't set max speed");
        return -1;
    }
    
    return 0;
}

int spi_transfer(uint8_t *tx, uint8_t *rx, int len) {
    struct spi_ioc_transfer tr = {
        .tx_buf = (unsigned long)tx,
        .rx_buf = (unsigned long)rx,
        .len = len,
        .speed_hz = spi_speed,
        .bits_per_word = spi_bits,
    };
    
    return ioctl(spi_fd, SPI_IOC_MESSAGE(1), &tr);
}

/* Esempio: Leggi ADC MCP3008 */
int read_adc_mcp3008(int channel) {
    uint8_t tx[3] = {1, (8 + channel) << 4, 0};
    uint8_t rx[3] = {0};
    
    spi_transfer(tx, rx, 3);
    
    return ((rx[1] & 3) << 8) + rx[2];  // 10-bit value
}
```

## Progetto Completo: Sistema di Controllo Temperatura

**temp_control_system.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <signal.h>
#include "rpi_gpio.h"

#define TEMP_SENSOR_PIN 4    // DHT22
#define HEATER_PIN 17        // Relay per riscaldatore
#define FAN_PIN 27           // Relay per ventola
#define LED_GREEN 22         // LED stato OK
#define LED_RED 23           // LED allarme

typedef struct {
    float current_temp;
    float target_temp;
    int heater_state;
    int fan_state;
    int running;
} SystemState;

SystemState state = {
    .current_temp = 20.0f,
    .target_temp = 22.0f,
    .heater_state = 0,
    .fan_state = 0,
    .running = 1
};

pthread_mutex_t state_mutex = PTHREAD_MUTEX_INITIALIZER;

void signal_handler(int sig) {
    state.running = 0;
}

/* Thread: Lettura sensore */
void* sensor_thread(void *arg) {
    while (state.running) {
        // Leggi temperatura (simulato)
        float temp = 20.0f + (rand() % 100) / 50.0f;
        
        pthread_mutex_lock(&state_mutex);
        state.current_temp = temp;
        pthread_mutex_unlock(&state_mutex);
        
        sleep(2);
    }
    return NULL;
}

/* Thread: Logica controllo */
void* control_thread(void *arg) {
    const float HYSTERESIS = 0.5f;
    
    while (state.running) {
        pthread_mutex_lock(&state_mutex);
        
        float diff = state.target_temp - state.current_temp;
        
        // Controllo riscaldatore
        if (diff > HYSTERESIS) {
            state.heater_state = 1;
            gpio_write(HEATER_PIN, 1);
        } else if (diff < -HYSTERESIS) {
            state.heater_state = 0;
            gpio_write(HEATER_PIN, 0);
        }
        
        // Controllo ventola
        if (state.current_temp > state.target_temp + 2.0f) {
            state.fan_state = 1;
            gpio_write(FAN_PIN, 1);
        } else if (state.current_temp < state.target_temp + 1.0f) {
            state.fan_state = 0;
            gpio_write(FAN_PIN, 0);
        }
        
        // LED status
        if (fabs(diff) < 0.5f) {
            gpio_write(LED_GREEN, 1);
            gpio_write(LED_RED, 0);
        } else {
            gpio_write(LED_GREEN, 0);
            gpio_write(LED_RED, diff > 2.0f ? 1 : 0);
        }
        
        pthread_mutex_unlock(&state_mutex);
        
        sleep(1);
    }
    return NULL;
}

/* Thread: Log e display */
void* display_thread(void *arg) {
    while (state.running) {
        pthread_mutex_lock(&state_mutex);
        
        printf("\r[Temp: %.1f°C | Target: %.1f°C | Heater: %s | Fan: %s]",
               state.current_temp,
               state.target_temp,
               state.heater_state ? "ON " : "OFF",
               state.fan_state ? "ON " : "OFF");
        fflush(stdout);
        
        pthread_mutex_unlock(&state_mutex);
        
        usleep(500000);
    }
    return NULL;
}

int main(void) {
    printf("Temperature Control System\n");
    printf("==========================\n\n");
    
    // Inizializza GPIO
    if (gpio_init() < 0) {
        fprintf(stderr, "Failed to initialize GPIO\n");
        return 1;
    }
    
    // Configura pin
    gpio_set_mode(HEATER_PIN, GPIO_OUTPUT);
    gpio_set_mode(FAN_PIN, GPIO_OUTPUT);
    gpio_set_mode(LED_GREEN, GPIO_OUTPUT);
    gpio_set_mode(LED_RED, GPIO_OUTPUT);
    
    // Signal handler
    signal(SIGINT, signal_handler);
    
    // Crea threads
    pthread_t sensor_t, control_t, display_t;
    
    pthread_create(&sensor_t, NULL, sensor_thread, NULL);
    pthread_create(&control_t, NULL, control_thread, NULL);
    pthread_create(&display_t, NULL, display_thread, NULL);
    
    // Attendi threads
    pthread_join(sensor_t, NULL);
    pthread_join(control_t, NULL);
    pthread_join(display_t, NULL);
    
    // Cleanup
    gpio_write(HEATER_PIN, 0);
    gpio_write(FAN_PIN, 0);
    gpio_write(LED_GREEN, 0);
    gpio_write(LED_RED, 0);
    
    gpio_cleanup();
    
    printf("\n\nSystem shutdown complete.\n");
    return 0;
}
```

## Ottimizzazioni Embedded

### Gestione Energia

```c
#include <avr/sleep.h>
#include <avr/power.h>

void enter_sleep_mode(void) {
    set_sleep_mode(SLEEP_MODE_PWR_DOWN);
    sleep_enable();
    
    // Disabilita moduli non necessari
    power_adc_disable();
    power_spi_disable();
    power_timer0_disable();
    power_timer1_disable();
    power_timer2_disable();
    power_twi_disable();
    
    sei();  // Abilita interrupts
    sleep_cpu();  // Enter sleep
    
    // Qui dopo wake-up
    sleep_disable();
    
    // Riabilita moduli
    power_all_enable();
}
```

### Watchdog Timer

```c
#include <avr/wdt.h>

void watchdog_init(void) {
    wdt_enable(WDTO_2S);  // Timeout 2 secondi
}

int main(void) {
    watchdog_init();
    
    while (1) {
        // Fai lavoro...
        
        wdt_reset();  // Reset watchdog (kick the dog)
        
        _delay_ms(100);
    }
}
```

## Esercizi Pratici

### Arduino:
1. **Datalogger SD Card** - Registra dati sensori su SD
2. **Display OLED I2C** - Mostra grafica e testo
3. **Motor controller** - Controlla motore DC con encoder
4. **RFID reader** - Sistema di accesso con tag RFID
5. **Sistema di irrigazione** - Automatico con sensore umidità

### Raspberry Pi:
1. **Web server con GPIO** - Controlla hardware via web
2. **Camera streaming** - Stream video con elaborazione
3. **Home automation hub** - Controlla luci e sensori
4. **Weather station** - Con database e grafici web
5. **Robot controller** - Controlla robot via WiFi

## Debugging Hardware

### Logic Analyzer (Software)

```c
/* Simple logic analyzer su Raspberry Pi */
#define SAMPLE_RATE 1000000  // 1 MHz
#define NUM_CHANNELS 8
#define BUFFER_SIZE 10000

void logic_analyzer_capture(int pins[], int num_pins) {
    uint8_t buffer[BUFFER_SIZE];
    
    for (int i = 0; i < BUFFER_SIZE; i++) {
        uint8_t sample = 0;
        for (int p = 0; p < num_pins; p++) {
            sample |= (gpio_read(pins[p]) << p);
        }
        buffer[i] = sample;
        
        // Timing preciso
        usleep(1);  // 1 μs
    }
    
    // Salva o analizza buffer
}
```

## Riferimenti

**Arduino:**
- AVR Datasheet
- Arduino Hardware Specification
- avr-libc Documentation

**Raspberry Pi:**
- BCM2835 ARM Peripherals Manual
- Linux Kernel GPIO Documentation
- WiringPi Documentation

---

[← Audio Synthesis](audio_synthesis.md) | [Torna a Low-Level](README.md)
