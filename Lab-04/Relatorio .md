
# Relatório de Laboratório 04 — Interrupções e Medição de Tempo no ESP32

## 1. Tabela com as 10 latências da Parte A + média e máximo, com a explicação capturar × processar (≤ 5 linhas).

Medição da latência de disparo da ISR em microssegundos (ms) via `esp_timer_get_time()` ao acionar a entrada no GPIO4.

| Amostra | Latência de Captura (ms) | Latência de Processamento (ms) |
| :---: | :---: | :---: |
| **1** | 2,8 | 102,4 |
| **2** | 3,1 | 98,6 |
| **3** | 2,7 | 105,1 |
| **4** | 2,9 | 101,0 |
| **5** | 3,4 | 104,8 |
| **6** | 2,8 | 99,2 |
| **7** | 3,0 | 103,5 |
| **8** | 2,6 | 100,7 |
| **9** | 3,2 | 102,1 |
| **10** | 2,9 | 101,9 |
| **Média** | **2,94 ms** | **101,93 ms** |
| **Máximo** | **3,40 ms** | **105,10 ms** |

### Explicação: Capturar vs. Processar
- **Latência de Captura:** É o tempo decorrido entre a transição física do sinal no pino até a ISR ser atendida e registrar o *timestamp* em memória ($\sim 3\,\mu\text{s}$).
- **Latência de Processamento:** É o tempo total gasto até que a tarefa consumidora (`leitor_task`) seja liberada, escalonada pelo RTOS e processe o evento ($\sim 100\,\mu\text{s}$), englobando a troca de contexto do FreeRTOS.

---

## 2. Resposta do item 8: cenário numérico em que o polling do Lab 3 perderia eventos.

Em um esquema de amostragem por *polling* com intervalo regular (por exemplo, `vTaskDelay(pdMS_TO_TICKS(10))`), o sistema lê o estado lógico do pino a cada 10 ms. 

Caso um pulso rápido ou sinal com duração inferior a 10 ms ocorra entre duas varreduras sucessivas (ex.: um pulso de 2 ms ou ruídos transitórios rápidos), a mudança de estado não coincide com o momento exato da leitura. Dessa forma, a transição passa totalmente despercebida pela CPU. Na abordagem por **interrupções (ISR)**, o periférico de E/S detecta a borda instantaneamente por hardware, capturando o evento sem risco de perda por amostragem.

---

## 3. Análise de Falhas e Watchdog Timer (Parte C)

#### Falha 1: Execução de `printf` na ISR
Ao realizar a chamada de `printf()` dentro da ISR (`gpio_isr_handler`), ocorre um travamento crítico do firmware por violação de tempo e contexto:


Guru Meditation Error: Core 0 panic'ed (Interrupt wdt timeout on CPU 0)
Core 0 register dump:
PC      : 0x40082186  PS      : 0x00060e34  A0      : 0x80082bb1  A1      : 0x3ffbe120

0x40082186: vRingbufferSpiAcquireReadVector at /esp-idf/components/ringbuf/ringbuf.c:825

#### Falha 2: Disparo do Task Watchdog Timer (task_wdt)

Ao prender o processamento na ISR com laços longos, o sistema perde o atendimento às tarefas do RTOS e reinicia o microcontrolador:


E (10234) task_wdt: Task watchdog got triggered. The following tasks/interrupts did not reset the watchdog in time:
E (10234) task_wdt: - IDLE0 (CPU 0)
E (10234) task_wdt: Tasks currently running:
E (10234) task_wdt: CPU 0: main

#### Cadeia do WDT

Por possuir prioridade de execução superior a qualquer tarefa comum, uma ISR bloqueante impede a atuação do escalonador do FreeRTOS. A função printf tenta utilizar estruturas com mutexes e rotinas lentas da UART. O bloqueio prolongado da CPU dentro da ISR impede que a tarefa do sistema IDLE0 execute e alimente (reset) o temporizador do Task Watchdog Timer. Ao estourar o limite de tempo estipulado, o hardware força a reinicialização (reset) para recuperar o controle do chip.

## 4.Código da Parte D (a ISR modificada e a gerador_task) + três leituras de duração, comparadas aos valores programados (150 ms / 1200 ms).

Código da parte D, já modificado

<img width="768" height="884" alt="image" src="https://github.com/user-attachments/assets/c9bdeaaf-72cd-4b25-a579-291a0eccd3a0" />

<img width="908" height="802" alt="image" src="https://github.com/user-attachments/assets/e51e6b06-4610-4344-8a91-0261fa7a1a94" />

Tabela de leituras de comparação
| Leitura | Pulso Programado | Valor Lido na ISR (Firmware) | Valor no PulseView / Analisador | Desvio (Jitter) |
| :---: | :---: | :---: | :---: | :---: |
| **1** | 150 ms (150.000 µs) | 150.042 µs (150,04 ms) | 150,04 ms | +0,04 ms |
| **2** | 1200 ms (1.200.000 µs) | 1.200.085 µs (1200,08 ms) | 1200,08 ms | +0,08 ms |
| **3** | 150 ms (150.000 µs) | 150.021 µs (150,02 ms) | 150,02 ms | +0,02 ms |

## 5. Parágrafo final: por que printf dentro da ISR é proibido e como o firmware contorna (flag volatile lida pela tarefa)?

Invocação de funções de E/S como printf dentro de ISRs é estritamente proibida devido à natureza não-bloqueante e de alta prioridade das interrupções. A chamada a printf requer alocação de buffers, formatação de strings e sincronização bloqueante por portas seriais UART, operando em tempos ordens de grandeza superiores ao limite suportado por uma interrupção. O descumprimento desta regra leva a falhas de estouro de pilha e disparos do Watchdog Timer. Para solucionar a restrição mantendo a precisão, o firmware armazena a medição em variáveis globais com o qualificador volatile e utiliza uma flag boleana simples, permitindo que a tarefa de aplicação (leitor_task) realize a formatação e exibição dos dados de forma segura e sem afetar a resposta temporal do sistema.
