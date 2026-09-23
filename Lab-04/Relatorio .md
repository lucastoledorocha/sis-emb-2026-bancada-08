
# Relatório de Laboratório 04 — Interrupções e Medição de Tempo no ESP32

**Disciplina:** Sistemas Embarcados  
**Data:** Setembro de 2026  

---

## 1. Medição de Latência de Interrupção (Parte A)

Medição da latência de disparo da ISR em microssegundos ($\mu$s) via `esp_timer_get_time()` ao acionar a entrada no GPIO4.

| Amostra | Latência de Captura ($\mu$s) | Latência de Processamento ($\mu$s) |
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
| **Média** | **2,94 $\mu$s** | **101,93 $\mu$s** |
| **Máximo** | **3,40 $\mu$s** | **105,10 $\mu$s** |

### Explicação: Capturar vs. Processar
- **Latência de Captura:** É o tempo decorrido entre a transição física do sinal no pino até a ISR ser atendida e registrar o *timestamp* em memória ($\sim 3\,\mu\text{s}$).
- **Latência de Processamento:** É o tempo total gasto até que a tarefa consumidora (`leitor_task`) seja liberada, escalonada pelo RTOS e processe o evento ($\sim 100\,\mu\text{s}$), englobando a troca de contexto do FreeRTOS.

---

## 2. Perda de Eventos em Polling (Item 8)

Em um esquema de amostragem por *polling* com intervalo regular (por exemplo, `vTaskDelay(pdMS_TO_TICKS(10))`), o sistema lê o estado lógico do pino a cada 10 ms. 

Caso um pulso rápido ou sinal com duração inferior a 10 ms ocorra entre duas varreduras sucessivas (ex.: um pulso de 2 ms ou ruídos transitórios rápidos), a mudança de estado não coincide com o momento exato da leitura. Dessa forma, a transição passa totalmente despercebida pela CPU. Na abordagem por **interrupções (ISR)**, o periférico de E/S detecta a borda instantaneamente por hardware, capturando o evento sem risco de perda por amostragem.

---

## 3. Análise de Falhas e Watchdog Timer (Parte C)

### Falha 1: Execução de `printf` na ISR
Ao realizar a chamada de `printf()` dentro da ISR (`gpio_isr_handler`), ocorre um travamento crítico do firmware por violação de tempo e contexto:

```text
Guru Meditation Error: Core 0 panic'ed (Interrupt wdt timeout on CPU 0)
Core 0 register dump:
PC      : 0x40082186  PS      : 0x00060e34  A0      : 0x80082bb1  A1      : 0x3ffbe120
...
0x40082186: vRingbufferSpiAcquireReadVector at /esp-idf/components/ringbuf/ringbuf.c:825
