# Sintesi Audio e Musica Elettronica in C

## Introduzione alla Sintesi Audio

La sintesi audio in C rappresenta un'applicazione perfetta della programmazione a basso livello: richiede precisione matematica, comprensione del DSP (Digital Signal Processing), e ottimizzazione per performance real-time.

### Fondamenti di Audio Digitale

**Concetti base:**
- **Sample Rate**: Frequenza di campionamento (es. 44100 Hz = CD quality)
- **Bit Depth**: Risoluzione dell'ampiezza (es. 16-bit, 24-bit)
- **Channels**: Mono (1), Stereo (2), Surround (5.1, 7.1)
- **Buffer Size**: Dimensione del buffer audio (trade-off latenza/performance)

```c
#include <stdint.h>
#include <math.h>

/* Configurazione audio standard */
#define SAMPLE_RATE 44100
#define CHANNELS 2
#define BITS_PER_SAMPLE 16
#define BUFFER_SIZE 512

typedef int16_t sample_t;  // 16-bit signed audio sample
```

## Forme d'Onda Base

### Oscillatori Fondamentali

**oscillator.h:**
```c
#ifndef OSCILLATOR_H
#define OSCILLATOR_H

#include <stdint.h>
#include <math.h>

#define SAMPLE_RATE 44100.0f
#define TWO_PI (2.0f * M_PI)

typedef enum {
    WAVE_SINE,
    WAVE_SQUARE,
    WAVE_SAW,
    WAVE_TRIANGLE,
    WAVE_NOISE
} WaveType;

typedef struct {
    WaveType type;
    float frequency;
    float phase;
    float amplitude;
    float phase_increment;
} Oscillator;

void osc_init(Oscillator *osc, WaveType type, float freq, float amp);
float osc_generate(Oscillator *osc);
void osc_set_frequency(Oscillator *osc, float freq);

#endif
```

**oscillator.c:**
```c
#include "oscillator.h"
#include <stdlib.h>

void osc_init(Oscillator *osc, WaveType type, float freq, float amp) {
    osc->type = type;
    osc->frequency = freq;
    osc->amplitude = amp;
    osc->phase = 0.0f;
    osc->phase_increment = (freq * TWO_PI) / SAMPLE_RATE;
}

void osc_set_frequency(Oscillator *osc, float freq) {
    osc->frequency = freq;
    osc->phase_increment = (freq * TWO_PI) / SAMPLE_RATE;
}

float osc_generate(Oscillator *osc) {
    float output = 0.0f;
    
    switch (osc->type) {
        case WAVE_SINE:
            output = sinf(osc->phase);
            break;
            
        case WAVE_SQUARE:
            output = (osc->phase < M_PI) ? 1.0f : -1.0f;
            break;
            
        case WAVE_SAW:
            output = (2.0f * osc->phase / TWO_PI) - 1.0f;
            break;
            
        case WAVE_TRIANGLE:
            if (osc->phase < M_PI) {
                output = -1.0f + (2.0f * osc->phase / M_PI);
            } else {
                output = 3.0f - (2.0f * osc->phase / M_PI);
            }
            break;
            
        case WAVE_NOISE:
            output = ((float)rand() / RAND_MAX) * 2.0f - 1.0f;
            break;
    }
    
    osc->phase += osc->phase_increment;
    if (osc->phase >= TWO_PI) {
        osc->phase -= TWO_PI;
    }
    
    return output * osc->amplitude;
}
```

## Inviluppo ADSR (Attack, Decay, Sustain, Release)

**envelope.h:**
```c
#ifndef ENVELOPE_H
#define ENVELOPE_H

typedef enum {
    ENV_IDLE,
    ENV_ATTACK,
    ENV_DECAY,
    ENV_SUSTAIN,
    ENV_RELEASE
} EnvelopeState;

typedef struct {
    EnvelopeState state;
    
    float attack_time;   // secondi
    float decay_time;    // secondi
    float sustain_level; // 0.0 - 1.0
    float release_time;  // secondi
    
    float current_level;
    float attack_rate;
    float decay_rate;
    float release_rate;
} Envelope;

void env_init(Envelope *env, float attack, float decay, 
              float sustain, float release);
void env_note_on(Envelope *env);
void env_note_off(Envelope *env);
float env_process(Envelope *env);

#endif
```

**envelope.c:**
```c
#include "envelope.h"

void env_init(Envelope *env, float attack, float decay, 
              float sustain, float release) {
    env->state = ENV_IDLE;
    env->attack_time = attack;
    env->decay_time = decay;
    env->sustain_level = sustain;
    env->release_time = release;
    env->current_level = 0.0f;
    
    // Calcola rate per sample
    env->attack_rate = 1.0f / (attack * SAMPLE_RATE);
    env->decay_rate = (1.0f - sustain) / (decay * SAMPLE_RATE);
    env->release_rate = sustain / (release * SAMPLE_RATE);
}

void env_note_on(Envelope *env) {
    env->state = ENV_ATTACK;
}

void env_note_off(Envelope *env) {
    env->state = ENV_RELEASE;
}

float env_process(Envelope *env) {
    switch (env->state) {
        case ENV_IDLE:
            env->current_level = 0.0f;
            break;
            
        case ENV_ATTACK:
            env->current_level += env->attack_rate;
            if (env->current_level >= 1.0f) {
                env->current_level = 1.0f;
                env->state = ENV_DECAY;
            }
            break;
            
        case ENV_DECAY:
            env->current_level -= env->decay_rate;
            if (env->current_level <= env->sustain_level) {
                env->current_level = env->sustain_level;
                env->state = ENV_SUSTAIN;
            }
            break;
            
        case ENV_SUSTAIN:
            env->current_level = env->sustain_level;
            break;
            
        case ENV_RELEASE:
            env->current_level -= env->release_rate;
            if (env->current_level <= 0.0f) {
                env->current_level = 0.0f;
                env->state = ENV_IDLE;
            }
            break;
    }
    
    return env->current_level;
}
```

## Filtri Audio

### Filtro Low-Pass (Risonante)

**filter.h:**
```c
#ifndef FILTER_H
#define FILTER_H

typedef struct {
    float cutoff;      // 0.0 - 1.0 (normalized frequency)
    float resonance;   // 0.0 - 1.0
    
    float buf0, buf1;  // Buffer per feedback
} LowPassFilter;

void filter_init(LowPassFilter *f, float cutoff, float resonance);
float filter_process(LowPassFilter *f, float input);
void filter_set_cutoff(LowPassFilter *f, float cutoff);

#endif
```

**filter.c:**
```c
#include "filter.h"
#include <math.h>

void filter_init(LowPassFilter *f, float cutoff, float resonance) {
    f->cutoff = cutoff;
    f->resonance = resonance;
    f->buf0 = 0.0f;
    f->buf1 = 0.0f;
}

void filter_set_cutoff(LowPassFilter *f, float cutoff) {
    f->cutoff = cutoff;
}

float filter_process(LowPassFilter *f, float input) {
    // Moog-style ladder filter (semplificato)
    float fb = f->resonance + f->resonance / (1.0f - f->cutoff);
    
    input -= f->buf1 * fb;
    input *= 0.35013f * (f->cutoff * f->cutoff) * 
             (f->cutoff * f->cutoff);
    
    f->buf0 += (input - f->buf0) * f->cutoff;
    f->buf1 += (f->buf0 - f->buf1) * f->cutoff;
    
    return f->buf1;
}
```

## Sintetizzatore Completo

**synth.h:**
```c
#ifndef SYNTH_H
#define SYNTH_H

#include "oscillator.h"
#include "envelope.h"
#include "filter.h"

#define MAX_VOICES 8

typedef struct {
    int active;
    int note;
    
    Oscillator osc1;
    Oscillator osc2;
    Envelope env;
    LowPassFilter filter;
    
    float osc_mix;  // 0.0 = osc1, 1.0 = osc2
} Voice;

typedef struct {
    Voice voices[MAX_VOICES];
    int num_voices;
    
    float master_volume;
} Synthesizer;

void synth_init(Synthesizer *synth);
void synth_note_on(Synthesizer *synth, int note, float velocity);
void synth_note_off(Synthesizer *synth, int note);
void synth_process(Synthesizer *synth, float *output, int num_samples);

#endif
```

**synth.c:**
```c
#include "synth.h"
#include <string.h>

/* Conversione MIDI note to frequency */
static float note_to_freq(int note) {
    return 440.0f * powf(2.0f, (note - 69) / 12.0f);
}

void synth_init(Synthesizer *synth) {
    synth->num_voices = MAX_VOICES;
    synth->master_volume = 0.5f;
    
    for (int i = 0; i < MAX_VOICES; i++) {
        Voice *v = &synth->voices[i];
        v->active = 0;
        v->note = -1;
        v->osc_mix = 0.5f;
        
        osc_init(&v->osc1, WAVE_SAW, 440.0f, 0.5f);
        osc_init(&v->osc2, WAVE_SQUARE, 440.0f, 0.5f);
        env_init(&v->env, 0.01f, 0.1f, 0.7f, 0.3f);
        filter_init(&v->filter, 0.8f, 0.3f);
    }
}

void synth_note_on(Synthesizer *synth, int note, float velocity) {
    // Trova voice libera
    Voice *v = NULL;
    for (int i = 0; i < MAX_VOICES; i++) {
        if (!synth->voices[i].active) {
            v = &synth->voices[i];
            break;
        }
    }
    
    if (!v) return;  // Nessuna voice disponibile
    
    v->active = 1;
    v->note = note;
    
    float freq = note_to_freq(note);
    osc_set_frequency(&v->osc1, freq);
    osc_set_frequency(&v->osc2, freq * 1.01f);  // Slight detune
    
    env_note_on(&v->env);
}

void synth_note_off(Synthesizer *synth, int note) {
    for (int i = 0; i < MAX_VOICES; i++) {
        Voice *v = &synth->voices[i];
        if (v->active && v->note == note) {
            env_note_off(&v->env);
        }
    }
}

void synth_process(Synthesizer *synth, float *output, int num_samples) {
    memset(output, 0, num_samples * sizeof(float));
    
    for (int i = 0; i < MAX_VOICES; i++) {
        Voice *v = &synth->voices[i];
        if (!v->active) continue;
        
        for (int s = 0; s < num_samples; s++) {
            float osc1_out = osc_generate(&v->osc1);
            float osc2_out = osc_generate(&v->osc2);
            
            // Mix oscillatori
            float mixed = osc1_out * (1.0f - v->osc_mix) + 
                         osc2_out * v->osc_mix;
            
            // Applica filtro
            float filtered = filter_process(&v->filter, mixed);
            
            // Applica inviluppo
            float env_level = env_process(&v->env);
            output[s] += filtered * env_level;
            
            // Disattiva voice se inviluppo finito
            if (env_level == 0.0f && v->env.state == ENV_IDLE) {
                v->active = 0;
            }
        }
    }
    
    // Master volume
    for (int s = 0; s < num_samples; s++) {
        output[s] *= synth->master_volume / MAX_VOICES;
        
        // Soft clipping
        if (output[s] > 1.0f) output[s] = 1.0f;
        if (output[s] < -1.0f) output[s] = -1.0f;
    }
}
```

## Effetti Audio

### Delay/Echo

**delay.c:**
```c
#include <stdlib.h>
#include <string.h>

typedef struct {
    float *buffer;
    int buffer_size;
    int write_pos;
    float feedback;
    float mix;
} Delay;

void delay_init(Delay *d, float time_seconds, float feedback, float mix) {
    d->buffer_size = (int)(SAMPLE_RATE * time_seconds);
    d->buffer = calloc(d->buffer_size, sizeof(float));
    d->write_pos = 0;
    d->feedback = feedback;
    d->mix = mix;
}

float delay_process(Delay *d, float input) {
    float delayed = d->buffer[d->write_pos];
    d->buffer[d->write_pos] = input + delayed * d->feedback;
    
    d->write_pos = (d->write_pos + 1) % d->buffer_size;
    
    return input * (1.0f - d->mix) + delayed * d->mix;
}

void delay_free(Delay *d) {
    free(d->buffer);
}
```

### Reverb (Semplice)

**reverb.c:**
```c
#define NUM_COMBS 4
#define NUM_ALLPASS 2

typedef struct {
    float *buffer;
    int size;
    int pos;
    float feedback;
} CombFilter;

typedef struct {
    float *buffer;
    int size;
    int pos;
} AllpassFilter;

typedef struct {
    CombFilter combs[NUM_COMBS];
    AllpassFilter allpass[NUM_ALLPASS];
    float mix;
} Reverb;

void comb_init(CombFilter *c, int size, float feedback) {
    c->buffer = calloc(size, sizeof(float));
    c->size = size;
    c->pos = 0;
    c->feedback = feedback;
}

float comb_process(CombFilter *c, float input) {
    float output = c->buffer[c->pos];
    c->buffer[c->pos] = input + output * c->feedback;
    c->pos = (c->pos + 1) % c->size;
    return output;
}

void reverb_init(Reverb *r) {
    // Comb filters con tempi diversi
    comb_init(&r->combs[0], 1116, 0.84f);
    comb_init(&r->combs[1], 1188, 0.82f);
    comb_init(&r->combs[2], 1277, 0.80f);
    comb_init(&r->combs[3], 1356, 0.78f);
    
    r->mix = 0.3f;
}

float reverb_process(Reverb *r, float input) {
    float output = 0.0f;
    
    // Parallelo comb filters
    for (int i = 0; i < NUM_COMBS; i++) {
        output += comb_process(&r->combs[i], input);
    }
    output /= NUM_COMBS;
    
    // Mix wet/dry
    return input * (1.0f - r->mix) + output * r->mix;
}
```

## Sequencer MIDI

**sequencer.h:**
```c
#ifndef SEQUENCER_H
#define SEQUENCER_H

#include <stdint.h>

#define MAX_NOTES 64
#define PPQN 96  // Pulses Per Quarter Note

typedef struct {
    int note;
    int velocity;
    int start_time;  // in pulses
    int duration;    // in pulses
} Note;

typedef struct {
    Note notes[MAX_NOTES];
    int num_notes;
    int current_pulse;
    int tempo;  // BPM
} Sequencer;

void seq_init(Sequencer *seq, int tempo);
void seq_add_note(Sequencer *seq, int note, int velocity, 
                  int start, int duration);
void seq_step(Sequencer *seq, Synthesizer *synth);

#endif
```

**sequencer.c:**
```c
#include "sequencer.h"

void seq_init(Sequencer *seq, int tempo) {
    seq->num_notes = 0;
    seq->current_pulse = 0;
    seq->tempo = tempo;
}

void seq_add_note(Sequencer *seq, int note, int velocity,
                  int start, int duration) {
    if (seq->num_notes >= MAX_NOTES) return;
    
    Note *n = &seq->notes[seq->num_notes++];
    n->note = note;
    n->velocity = velocity;
    n->start_time = start;
    n->duration = duration;
}

void seq_step(Sequencer *seq, Synthesizer *synth) {
    for (int i = 0; i < seq->num_notes; i++) {
        Note *n = &seq->notes[i];
        
        // Note on
        if (n->start_time == seq->current_pulse) {
            synth_note_on(synth, n->note, n->velocity / 127.0f);
        }
        
        // Note off
        if (n->start_time + n->duration == seq->current_pulse) {
            synth_note_off(synth, n->note);
        }
    }
    
    seq->current_pulse++;
}
```

## Esempio Completo: Mini DAW

**mini_daw.c:**
```c
#include <stdio.h>
#include <stdlib.h>
#include "synth.h"
#include "sequencer.h"

/* Esempio di pattern musicale */
void create_pattern(Sequencer *seq) {
    // Scala C maggiore: C D E F G A B C
    int scale[] = {60, 62, 64, 65, 67, 69, 71, 72};
    
    // Arpeggio
    for (int i = 0; i < 8; i++) {
        seq_add_note(seq, scale[i], 100, i * PPQN/4, PPQN/8);
    }
    
    // Accordo
    seq_add_note(seq, 60, 80, 8 * PPQN/4, PPQN);  // C
    seq_add_note(seq, 64, 80, 8 * PPQN/4, PPQN);  // E
    seq_add_note(seq, 67, 80, 8 * PPQN/4, PPQN);  // G
}

/* Salva su WAV file */
void save_wav(const char *filename, float *data, int num_samples) {
    FILE *f = fopen(filename, "wb");
    if (!f) return;
    
    int byte_rate = SAMPLE_RATE * sizeof(int16_t);
    int data_size = num_samples * sizeof(int16_t);
    
    // WAV header
    fwrite("RIFF", 1, 4, f);
    int chunk_size = 36 + data_size;
    fwrite(&chunk_size, 4, 1, f);
    fwrite("WAVE", 1, 4, f);
    
    // fmt chunk
    fwrite("fmt ", 1, 4, f);
    int fmt_size = 16;
    fwrite(&fmt_size, 4, 1, f);
    int16_t format = 1;  // PCM
    fwrite(&format, 2, 1, f);
    int16_t channels = 1;  // Mono
    fwrite(&channels, 2, 1, f);
    int sample_rate = SAMPLE_RATE;
    fwrite(&sample_rate, 4, 1, f);
    fwrite(&byte_rate, 4, 1, f);
    int16_t block_align = sizeof(int16_t);
    fwrite(&block_align, 2, 1, f);
    int16_t bits = 16;
    fwrite(&bits, 2, 1, f);
    
    // data chunk
    fwrite("data", 1, 4, f);
    fwrite(&data_size, 4, 1, f);
    
    // Convert float to int16 and write
    for (int i = 0; i < num_samples; i++) {
        int16_t sample = (int16_t)(data[i] * 32767.0f);
        fwrite(&sample, sizeof(int16_t), 1, f);
    }
    
    fclose(f);
}

int main(void) {
    printf("=== Mini DAW - Sintesi Audio in C ===\n\n");
    
    Synthesizer synth;
    Sequencer seq;
    
    synth_init(&synth);
    seq_init(&seq, 120);  // 120 BPM
    
    printf("Creazione pattern...\n");
    create_pattern(&seq);
    
    printf("Sintesi audio...\n");
    
    // 4 secondi di audio
    int duration_samples = SAMPLE_RATE * 4;
    float *audio_buffer = malloc(duration_samples * sizeof(float));
    
    int samples_per_pulse = (SAMPLE_RATE * 60) / (seq.tempo * PPQN);
    int processed = 0;
    
    while (processed < duration_samples) {
        seq_step(&seq, &synth);
        
        int to_process = samples_per_pulse;
        if (processed + to_process > duration_samples) {
            to_process = duration_samples - processed;
        }
        
        synth_process(&synth, audio_buffer + processed, to_process);
        processed += to_process;
    }
    
    printf("Salvataggio WAV...\n");
    save_wav("output.wav", audio_buffer, duration_samples);
    
    printf("File 'output.wav' creato con successo!\n");
    printf("\nComponenti implementati:\n");
    printf("- Oscillatori (sine, saw, square, triangle)\n");
    printf("- Inviluppo ADSR\n");
    printf("- Filtro low-pass risonante\n");
    printf("- Synth polifonico (8 voci)\n");
    printf("- Sequencer MIDI\n");
    
    free(audio_buffer);
    return 0;
}
```

## Sintesi Modulare

### Patch Bay Virtuale

```c
typedef enum {
    MODULE_OSC,
    MODULE_FILTER,
    MODULE_ENV,
    MODULE_LFO,
    MODULE_VCA,
    MODULE_MIXER
} ModuleType;

typedef struct Module {
    ModuleType type;
    void *data;
    float *inputs[4];
    float outputs[4];
    
    void (*process)(struct Module *self);
} Module;

typedef struct {
    Module *source;
    int source_output;
    Module *dest;
    int dest_input;
} Patch;

/* Patch: LFO modula frequenza OSC */
void create_patch_example(Module *lfo, Module *osc) {
    osc->inputs[0] = &lfo->outputs[0];  // LFO out -> OSC freq in
}
```

## Algoritmi di Sintesi Avanzati

### FM Synthesis (Frequency Modulation)

```c
typedef struct {
    Oscillator carrier;
    Oscillator modulator;
    float mod_index;  // Intensità modulazione
} FMSynth;

float fm_generate(FMSynth *fm) {
    float mod_out = osc_generate(&fm->modulator);
    
    // Modula frequenza del carrier
    float original_freq = fm->carrier.frequency;
    fm->carrier.frequency = original_freq * (1.0f + mod_out * fm->mod_index);
    
    float output = osc_generate(&fm->carrier);
    
    fm->carrier.frequency = original_freq;  // Ripristina
    return output;
}
```

### Wavetable Synthesis

```c
#define WAVETABLE_SIZE 2048

typedef struct {
    float table[WAVETABLE_SIZE];
    float position;
    float increment;
} WavetableOsc;

void wavetable_init(WavetableOsc *w, float *waveform) {
    for (int i = 0; i < WAVETABLE_SIZE; i++) {
        w->table[i] = waveform[i];
    }
    w->position = 0.0f;
}

float wavetable_generate(WavetableOsc *w) {
    int index = (int)w->position;
    float frac = w->position - index;
    
    // Interpolazione lineare
    float sample1 = w->table[index];
    float sample2 = w->table[(index + 1) % WAVETABLE_SIZE];
    float output = sample1 + frac * (sample2 - sample1);
    
    w->position += w->increment;
    if (w->position >= WAVETABLE_SIZE) {
        w->position -= WAVETABLE_SIZE;
    }
    
    return output;
}
```

## Analisi Spettrale (FFT Base)

```c
#include <complex.h>

void simple_fft(float complex *data, int n) {
    if (n <= 1) return;
    
    // Divide
    float complex *even = malloc((n/2) * sizeof(float complex));
    float complex *odd = malloc((n/2) * sizeof(float complex));
    
    for (int i = 0; i < n/2; i++) {
        even[i] = data[i*2];
        odd[i] = data[i*2 + 1];
    }
    
    // Conquer
    simple_fft(even, n/2);
    simple_fft(odd, n/2);
    
    // Combine
    for (int k = 0; k < n/2; k++) {
        float complex t = cexpf(-2.0f * I * M_PI * k / n) * odd[k];
        data[k] = even[k] + t;
        data[k + n/2] = even[k] - t;
    }
    
    free(even);
    free(odd);
}
```

## Ottimizzazioni Audio Real-Time

### SIMD per Audio Processing

```c
#include <immintrin.h>  // AVX/SSE

/* Processamento 8 sample alla volta con AVX */
void process_audio_simd(float *input, float *output, int num_samples,
                        float gain) {
    __m256 gain_vec = _mm256_set1_ps(gain);
    
    int i;
    for (i = 0; i < num_samples - 7; i += 8) {
        __m256 data = _mm256_loadu_ps(&input[i]);
        data = _mm256_mul_ps(data, gain_vec);
        _mm256_storeu_ps(&output[i], data);
    }
    
    // Processa rimanenti
    for (; i < num_samples; i++) {
        output[i] = input[i] * gain;
    }
}
```

## Esercizi

1. **Implementa un arpeggiatore** con diverse modalità (up, down, random)
2. **Crea un campionatore** che legge file WAV e li riproduce
3. **Implementa un vocoder** (almeno versione base)
4. **Crea un drum machine** con sample di batteria
5. **Implementa un compressore audio** con attack, release, threshold
6. **Crea un pitch shifter** usando time-stretching
7. **Implementa un chorus effect** con multiple delay lines

## Riferimenti

- **Libri:**
  - "The Computer Music Tutorial" - Curtis Roads
  - "Designing Sound" - Andy Farnell
  - "The Audio Programming Book" - MIT Press

- **Librerie da studiare:**
  - PortAudio - Audio I/O cross-platform
  - RtAudio - Real-time audio
  - JUCE - Framework audio completo
  - Maximilian - DSP library

---

[← Retro Gaming](retro_gaming.md) | [Arduino & Raspberry →](hardware_programming.md)
