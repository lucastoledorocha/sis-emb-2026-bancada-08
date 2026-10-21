# Relatório - Lab 06: Filas, Semáforos e Mutex

## 2. Números da Parte B (Produtor-Consumidor com Fila)
* **Itens drenados por rajada (Item 14):** 30 itens
* **Perdas registradas com apagão de 700 ms (Item 15):** Em média 6 perdas por rajada (esperado: 70 - 64 = 6). 
* **Cálculo e novo tamanho da fila (Item 16):** Para suportar o apagão de 700 ms com folga de 2X a produção no período é de $70$ amostras. A folga exige $70 \times 2 = 140$. A próxima potência de 2 é **256**. O código deve ser ajustado para `xQueueCreate(256, sizeof(uint32_t));`
  
<img width="1600" height="900" alt="WhatsApp Image 2026-10-07 at 15 44 11" src="https://github.com/user-attachments/assets/b2d103d9-eb12-4dcb-90b7-38de8c1293ef" />
com 300ms

<img width="1600" height="900" alt="WhatsApp Image 2026-10-07 at 15 44 47" src="https://github.com/user-attachments/assets/3aa32817-d0e9-4a27-9cd4-187dcabf0036" />
com 700ms 


<img width="1600" height="900" alt="WhatsApp Image 2026-10-07 at 16 04 09(1)" src="https://github.com/user-attachments/assets/0c2556a9-39e6-4b60-807f-21477c80aa09" />
`xQueueCreate(256, sizeof(uint32_t));`


## 3. Tabela de Latências e Código da ISR/Tarefa
**Tabela de Latências:**
| método | latência média | latência máx |
|---|---|---|
| Lab 4: flag + polling da tarefa | 101,93 | 105,10 |
| Lab 6: semáforo + take bloqueante | 21 | 34 |


**Código da ISR e Tarefa:**
```c
static void IRAM_ATTR isr_botao(void *arg)
{
    int64_t agora = esp_timer_get_time();
    if (agora - s_ultimo_evento_us <= DEBOUNCE_US) {
        return;                               // bounce: ignora, não conta, não dá
    }
    s_ultimo_evento_us = agora;
    s_t_isr = agora;

    BaseType_t woken = pdFALSE;
    xSemaphoreGiveFromISR(sem, &woken);        // ~1 µs: só sinaliza "aconteceu"
    portYIELD_FROM_ISR(woken);                 // troca de contexto JÁ, se preciso
}

static void tarefa_botao(void *arg)
{
    uint32_t eventos = 0;
    while (1) {
        xSemaphoreTake(sem, portMAX_DELAY);    // dorme sem gastar CPU até o give
        int64_t latencia_us = esp_timer_get_time() - s_t_isr;
        printf("evento #%lu | latencia ISR->tarefa: %lld us\n",
               (unsigned long)(++eventos), latencia_us);
    }
}
```

## 4. Diferença entre Semáforo Binário e Contador (Item 23)
O semáforo binário satura em $1$, logo, múltiplos eventos (`give`) que ocorrem muito rápido antes de a tarefa conseguir consumi-los (`take`) são descartados (perdidos). O semáforo contador acumula os `give`s em sua contagem até o seu limite máximo (neste caso, 10), garantindo que a tarefa acorde o número exato de vezes correspondente às bordas reais registradas.


<img width="1200" height="1600" alt="WhatsApp Image 2026-10-07 at 16 01 51" src="https://github.com/user-attachments/assets/a96648d9-a2da-49ca-91e5-bce444391839" />

Define=1



<img width="1200" height="1600" alt="WhatsApp Image 2026-10-07 at 16 02 10" src="https://github.com/user-attachments/assets/4a173e2d-c97b-4b05-8e7f-735bc214813b" />

Define=0


## 5. Proteção de Barramento I2C
Para proteger o barramento I2C compartilhado entre tarefas, a primitiva correta é o **Mutex**. Não devemos usar um semáforo binário porque o Mutex implementa a "herança de prioridade" (*priority inheritance*), mecanismo essencial para evitar a Inversão de Prioridade Não Limitada que travou a sonda Mars Pathfinder.

## Parte B

https://wokwi.com/projects/477249459524264961

## Parte C

<img width="1600" height="900" alt="WhatsApp Image 2026-10-07 at 15 49 57" src="https://github.com/user-attachments/assets/55b24c62-694a-4e5f-b502-1f719191995d" />

https://wokwi.com/projects/477249459524264961
