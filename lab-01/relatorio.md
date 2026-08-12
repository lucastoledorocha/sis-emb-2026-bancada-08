
# sis-emb-2026-bancada-08

Link https://wokwi.com/projects/472153508039227393	
 
! [Imagem Circuito Esp32][([Captura de tela 2026-08-12 102907.png](https://raw.githubusercontent.com/lucastoledorocha/sis-emb-2026-bancada-08/refs/heads/main/Captura%20de%20tela%202026-08-12%20102907.png?token=GHSAT0AAAAAAEFRBNI7TRIFO6VOD625HE7Q2T4QFIQ)](https://raw.githubusercontent.com/lucastoledorocha/sis-emb-2026-bancada-08/refs/heads/main/Captura%20de%20tela%202026-08-12%20102907.png?token=GHSAT0AAAAAAEFRBNI7TRIFO6VOD625HE7Q2T4QFIQ))


# Experimento 1.
|Período (ms)	|Piscar foi perceptível? |  
| ---| ---|
|400 |	sim |
|300 |	sim |
|200 |	sim |
|150 |	sim |
|100 |			Sim, porém com maior dificuldade |
|50 |	não |


# Experimento 2:  

        while (1) {
        gpio_set_level(PINO_LED, 1);
        printf("LED = 1\n");
        vTaskDelay(pdMS_TO_TICKS(900));
        gpio_set_level(PINO_LED, 0);
        printf("LED = 0\n");
        vTaskDelay(pdMS_TO_TICKS(100));
    }


## Mudanças ##

Adicionado outro vTaskdelay como o recomendado
Printf gerando uma resposta fixa (0 ou 1)

# Experimento 3. #

O vTaskDelay() coloca a tarefa atual em um estado para baixo consumo (Sleep), o que realiza uma grande eficiência energética. Já um laço 'for' vazio causa mantém a CPU ocupada trabalhando nesse laço. Conforme argumentado no Exemplo 1.1 da teoria, essa diferença de eficiência energética é crítica: um nó que operaria por 10 meses com o atraso inteligente esgotaria sua bateria em apenas 25 horas com um laço vazio (no caso esquecer o rádio ligado).

## EXTRA ##:

#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/gpio.h"

#define PINO_LED GPIO_NUM_2

void simbolo(int duracao_ms) {
    gpio_set_level(PINO_LED, 1);             
    vTaskDelay(pdMS_TO_TICKS(duracao_ms));   
    
    gpio_set_level(PINO_LED, 0);             
    vTaskDelay(pdMS_TO_TICKS(200));          
}

void app_main(void) {
    gpio_reset_pin(PINO_LED);
    gpio_set_direction(PINO_LED, GPIO_MODE_OUTPUT);

    while (1) {
        printf("S (···)\n");
        simbolo(200); 
        simbolo(200); 
        simbolo(200); 
        
        vTaskDelay(pdMS_TO_TICKS(600)); 

        printf("O (---)\n");
        simbolo(600); 
        simbolo(600); 
        simbolo(600); 
        
        
        vTaskDelay(pdMS_TO_TICKS(600));

        printf("S (···)\n");
        simbolo(200); 
        simbolo(200); 
        simbolo(200); 

   
        printf("---\n");
        vTaskDelay(pdMS_TO_TICKS(1200));
    }
}

Criada função “void simbolo(int duracao_ms)” conforme o recomendado
configurado uma pausa padrão entre símbolos de 200ms
Letra com o código morse em parênteses para facilitar a visualização 

