# Laboratório 2: ESP-IDF no hardware <img width="1600" height="900" alt="WhatsApp Image 2026-08-12 at 16 31 53" src="https://github.com/user-attachments/assets/ebdfa4b1-fd9e-4cc5-9bce-15ff7af534ef" />


## 1. Evidências Visuais
*   **Montagem Física:** <img width="1600" height="900" alt="WhatsApp Image 2026-08-12 at 16 31 53" src="https://github.com/user-attachments/assets/ebdfa4b1-fd9e-4cc5-9bce-15ff7af534ef" />
*   **Monitor Serial:** [Insira o link/imagem do print do terminal rodando o blink aqui]

## 2. Análise de Memória e Gravação

### Consumo de Memória (`idf.py size`)
| Memória | Uso (Bytes) | Descrição do Conteúdo |
| :--- | :--- | :--- |
| **DRAM** | `[Preencher]` | Dados (.data + .bss) e pilhas de execução. |
| **IRAM** | `[Preencher]` | Código executado diretamente da RAM (ex: ISRs). |
| **Flash** | `[Preencher]` | Código do programa (.text + .rodata). |

### Layout da Flash
Durante a gravação, foram identificados os seguintes endereços base:
1.  **0x1000**: Bootloader
2.  **0x8000**: Tabela de partições
3.  **0x10000**: Arquivo principal (App)

**Papel do endereço 0x10000:** 
Este offset na memória Flash é onde o executável do programa principal (o firmware da aplicação em si) é efetivamente armazenado. O bootloader, localizado em 0x1000, lê a tabela de partições e salta para este endereço 0x10000 para iniciar a execução do nosso código.

## 3. Análise Elétrica

| Medição | Onde | Valor esperado | Medido (Bancada) |
| :--- | :--- | :--- | :--- |
| **Alimentação do módulo** | pino 5V ↔ GND | 4,75–5,25 V | `[Preencher] V` |
| **Nível alto no GPIO** | GPIO 2 ↔ GND | ~3,3 V | `[Preencher] V` |
| **Queda no LED** | anodo ↔ catodo | ~1,8–2,1 V | `[Preencher] V` |

**Memória de Cálculo (Corrente do LED):**
Considerando os valores medidos e a resistência de 220 Ω, a corrente que flui pelo circuito é:

$$ I = \frac{V_{GPIO} - V_{LED}}{R} $$
$$ I = \frac{[V_{GPIO\_medido}] - [V_{LED\_medido}]}{220} $$
$$ I = [Resultado] \text{ mA} $$

*(Exemplo esperado se V_GPIO = 3.3V e V_LED = 2.0V: I ≈ 5.9 mA)*

## 4. Conceito de Atomicidade

A principal vantagem de utilizar os registradores dedicados `W1TS` (Write 1 To Set) e `W1TC` (Write 1 To Clear) é a garantia de **atomicidade via hardware**. Na abordagem clássica de "lê-modifica-escreve" com o operador `|=`, o processador precisa de vários ciclos de clock para ler o estado, alterar o bit e reescrever o dado, o que abre margem para **condições de corrida** caso uma interrupção altere o registrador no meio do processo. Com o `W1TS`, informamos em uma única instrução indivisível de escrita qual pino deve ser ativado, e o próprio hardware lógico altera apenas aquele pino, tornando o sistema imune a corrupções por interrupções concorrentes.

---

## 5. Desafio (Opcional)

**Objetivo:** Configurar a direção do GPIO 2 escrevendo no registrador `GPIO_ENABLE_W1TS_REG` sem utilizar o driver `gpio_set_direction`.

**Código implementado:**
```c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "soc/gpio_reg.h"
#include "driver/gpio.h"

#define BIT_LED (1u << 2)

void app_main(void)
{
    // Reseta o pino (recomendado para limpar configurações residuais)
    gpio_reset_pin(GPIO_NUM_2);
    
    // Configura a direção (Output) de forma atômica no registrador ENABLE
    *(volatile uint32_t *)GPIO_ENABLE_W1TS_REG = BIT_LED;

    volatile uint32_t *w1ts = (volatile uint32_t *)GPIO_OUT_W1TS_REG;
    volatile uint32_t *w1tc = (volatile uint32_t *)GPIO_OUT_W1TC_REG;

    while (1) {
        *w1ts = BIT_LED;
        vTaskDelay(pdMS_TO_TICKS(250));
        *w1tc = BIT_LED;
        vTaskDelay(pdMS_TO_TICKS(250));
    }
}
