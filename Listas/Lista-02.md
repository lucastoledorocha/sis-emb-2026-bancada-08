# Lista de Exercícios 02 — Semanas 4 a 6

## Grupo: Lucas Toledo Rocha, Lucas Vilella Bergamo e Gabriel dos Santos Oliveira

## Parte A — Interrupções e timers (semana 4)

## Q1. Compare polling e interrupção quanto a: latência de pior caso, uso de CPU e complexidade. Em que situação o polling é a escolha certa?

Latência de pior caso: O polling possui uma latência alta e variável, dependendo do tempo do ciclo de varredura. As interrupções têm uma latência muito mais baixa e determinística. 

Uso de CPU: O polling consome ciclos de CPU continuamente para verificar um estado que pode não ter mudado. As interrupções libertam a CPU, permitindo que processe outras tarefas ou hiberne até que o evento ocorra. 

Complexidade: O polling é linear e simples de implementar. As interrupções introduzem maior complexidade no código (como o salvamento de contexto, concorrência e partilha de dados).   

A escolha certa: O polling é adequado para eventos que ocorrem com uma frequência extremamente alta (onde o custo contínuo de entrar e sair de interrupções sobrecarregaria a CPU) ou quando o hardware não possui suporte nativo para interrupções associadas a esse evento.

## Q2. *(estilo Exemplo 4.1)* Um encoder gera pulsos de 400 µs a até 1 kHz. Mostre por que a varredura de 10 ms do Lab 3 (`vTaskDelay(pdMS_TO_TICKS(10))`) perde pulsos e dimensione o período de polling que não perderia. Conclua se ainda é razoável ou se a ISR se impõe.

> 💡 *Critério da teoria-04: período de varredura ≤ período do evento ÷ 10. Faça as contas dos
> dois lados antes de concluir.*
> 
Por que perde pulsos: O período do evento é de $1\text{ ms}$. 
A varredura de $10\text{ ms}$ é 10x mais lenta que a ocorrência do sinal. 
A leitura cairá nos intervalos entre pulsos na grande maioria das vezes.

Dimensionamento correto: $T_{varredura} \le \frac{1\text{ ms}}{10} \implies T_{varredura} \le \mathbf{100\ \mu\text{s}}$.

Conclusão: A ISR se impõe. Um polling de 100 µs consumiria 100% da CPU e é inviável no FreeRTOS (cujo tick base é de 1 ms)

## Q3 Enumere as três regras de ouro de uma ISR vistas em aula (curta; sem chamadas bloqueantes; APIs `FromISR` + `IRAM_ATTR` quando aplicável) e explique o que dá errado ao violar cada uma.
> 💡 *Os experimentos do Lab 4, Parte C, são as provas empíricas: cite o que vocês viram no
> monitor (o crash do printf e o task_wdt).*

1. Ser o mais curta possível: Violar isso rouba tempo do processador, acionando o Watchdog Timer (TWDT) e causando reset (Kernel Panic).

 2. Não usar funções bloqueantes (printf, vTaskDelay): A ISR não roda em uma task. Bloqueá-la causa falhas fatais do núcleo (Guru Meditation).

3. Usar APIs FromISR e o atributo IRAM_ATTR: APIs RTOS normais corrompem o escalonador se usadas na ISR. Omissão do IRAM_ATTR deixa a ISR na Flash; se o chip tentar ler a ISR enquanto a Flash estiver ocupada, o MCU reseta.

## Q4. *(estilo Exemplo 4.2)* Com clock base de 80 MHz, projete prescaler e valor de comparação para interrupções a cada 250 ms. Dê duas combinações válidas e diga qual prefere e por quê.

Fórmula: $\text{Valor} = \left(\frac{80.000.000}{\text{Prescaler}}\right) \times 0{,}25\text{ s}$

Combinação 1: Prescaler = $80$. 
Frequência = 1 MHz (1 tick = 1 µs). Valor = 250.000.

Combinação 2: Prescaler = $8000$. Frequência = 10 kHz (1 tick = 0,1 ms). Valor = 2.500.
Preferência: Combinação 1. 

No ESP32, 1 tick = 1 µs facilita contas mentais, e o valor 250.000 cabe facilmente no timer de 64 bits (sem risco de estouro).

## Q5. O que é um watchdog timer, que classe de falha ele mitiga e por que "alimentar" o WDT dentro de uma ISR de timer é um **anti-padrão**?

Temporizador que reseta o sistema se o software não "alimentá-lo" periodicamente. Salva o sistema de travamentos (ex: loops infinitos).

Por que não alimentar na ISR de timer: Interrupções têm prioridade máxima e continuam rodando independentemente do loop principal. Se a task principal travar, a ISR continuará alimentando o WDT, mascarando o erro e impedindo o reset salvador.

## Parte B — FreeRTOS: tarefas (semana 5)
**Q6.** Desenhe o diagrama de estados de uma tarefa (pronta, executando, bloqueada, suspensa) e
dê um evento concreto do nosso Lab 5 que causa cada transição.
> 💡 *Ex.: A→bloqueada: `vTaskDelayUntil`; bloqueada→pronta: fim do delay; pronta→executando:
> escalonador; executando→pronta: preempção pelo HOG…*

<details><summary>🖼️ Confira seu desenho (Fig. 5-B da teoria-05)</summary>

![Diagrama de estados de uma tarefa FreeRTOS: pronta, executando, bloqueada, suspensa](https://raw.githubusercontent.com/fabiobento/sis-emb-2026-2/main/assets/figuras/estados_tarefa.png)

</details>

## Q7 *(estilo Exemplo 5.1)* Tarefa de 100 ms usa `vTaskDelay(100 ms)` e seu corpo demora
7 ms. Calcule o período efetivo e a deriva acumulada em 1 min; corrija com `vTaskDelayUntil` e
explique a diferença de semântica.

![Comparação entre vTaskDelay (período relativo, com deriva) e vTaskDelayUntil (período absoluto)](https://raw.githubusercontent.com/fabiobento/sis-emb-2026-2/main/assets/figuras/vtaskdelay_vs_until.png)

Período efetivo: Como a tarefa gasta 7 ms processando e depois pede um delay de 100 ms, o ciclo real dura $107\text{ ms}$.

Deriva em 1 minuto ($60.000\text{ ms}$):

Em tese, a tarefa deveria rodar 600 vezes ($60.000 / 100$).

Na prática, 600 execuções de 107 ms levarão $64.200\text{ ms}$ ($64{,}2\text{ s}$).

Deriva acumulada: Atraso de $4{,}2\text{ segundos}$ a cada minuto.

Correção: Usar vTaskDelayUntil()

Diferença de semântica: vTaskDelay conta o tempo a partir da chamada (relativo, ignorando o tempo de processamento). vTaskDelayUntil acorda a tarefa em intervalos absolutos precisos (ex: exatos 100 ms desde a última vez que acordou), absorvendo a duração do processamento.

## Q8 *(estilo Exemplo 5.2)* Tarefas: A (3 ms de CPU a cada 10 ms), B (10 ms a cada 100 ms), C (log, quando der). Atribua prioridades, calcule a utilização total e verifique se C ainda roda. O que acontece com C se B virar 60 ms a cada 100 ms?

Prioridades (método Rate Monotonic - menor período = maior prioridade):

A: Alta (Período 10 ms).
B: Média (Período 100 ms).
C: Baixa (Best effort / Quando der).

Utilização Total ($U$):

$$U = \frac{C_A}{T_A} + \frac{C_B}{T_B} = \frac{3}{10} + \frac{10}{100} = 0{,}30 + 0{,}10 = \mathbf{0{,}40}$$

A tarefa C roda? Sim. Restam 60% do tempo de CPU livre para a tarefa C (e para a Idle).

E se B virar 60 ms a cada 100 ms?

$$U = 0{,}30 + \frac{60}{100} = 0{,}30 + 0{,}60 = \mathbf{0{,}90}$$

A tarefa C ainda roda, mas agora ficará restrita a usar apenas os 10% restantes da CPU.

**Q9.** Explique *starvation* e como o escalonador do FreeRTOS com *time slicing* trata tarefas
de **mesma** prioridade. Por que um `while(1){}` sem bloqueio numa tarefa de prioridade alta
congela as demais (e como o WDT de idle nos avisa)?
> 💡 *Vocês provocaram isso duas vezes: Lab 4 (PROVOCAR_WDT) e Lab 5 (HOG). Cite os logs.*

Starvation (Inanição): Ocorre quando tarefas de baixa prioridade nunca conseguem ser executadas porque as tarefas de prioridade maior monopolizam a CPU e nunca entram em estado de Bloqueio.

Time Slicing (Round Robin): O FreeRTOS divide o tempo de CPU fatiando o uso entre tarefas de mesma prioridade, alternando a execução a cada tick do sistema (ex: a cada 1 ms) para que todas progridam igualmente.

Problema do while(1) sem bloqueio: Se uma tarefa de alta prioridade fica em um loop infinito sem chamar um delay ou bloquear em um RTOS, o escalonador nunca passa a vez para as tarefas de menor prioridade.

Aviso do WDT: A tarefa Idle (prioridade 0, responsável por resetar o WDT do núcleo) sofre starvation. O chip detecta isso e acusa no log.

## Q10 No ESP32 (2 núcleos), o que muda ao fixar (`xTaskCreatePinnedToCore`) a tarefa de controle no core 1 e deixar Wi-Fi/sistema no core 0? Cite um benefício e um cuidado.
> 💡 *Benefício: jitter — teoria-05, seção 3. Cuidado: dados compartilhados entre núcleos viram
> concorrência REAL (simultânea), não só intercalada — a semana 6 inteira se aplica dobrada.*

O que muda: Separar a tarefa de controle (Core 1) das pilhas de rede/Wi-Fi (Core 0) impede que os pesados processamentos do Wi-Fi preempendam ou atrasem o seu controle.

Benefício: Reduz drasticamente o jitter (variação no tempo de resposta), garantindo um comportamento determinístico e tempo real mais previsível para o controle no Core 1.

Cuidado: Variáveis e filas compartilhadas entre os núcleos sofrem concorrência real e simultânea (verdadeiro paralelismo), e não apenas intercalação de threads. Exige sincronização rigorosa (uso de Mutexes ou Spinlocks) para evitar corrupção de dados.

## Parte C — IPC e sincronização (semana 6)
## Q11 *(estilo Exemplo 6.1)* Duas tarefas fazem `g++` (lê–incrementa–escreve) 1000× cada.
Explique o entrelaçamento que perde incrementos e dê o intervalo possível do valor final.
Corrija com mutex e com alternativa sem bloqueio (variável por tarefa + agregação).
> 💡 *A alternativa sem bloqueio é a "solução do jardim de infância": cada criança com seu
> brinquedo, soma no final. Quando ela NÃO serve? (Quando o dado precisa de leitura
> instantânea consistente, não só de soma final.)*

![Interleaving de instruções de duas tarefas mostrando a perda de um incremento](https://raw.githubusercontent.com/fabiobento/sis-emb-2026-2/main/assets/figuras/corrida_interleaving.png)


O entrelaçamento que perde incrementos: O operador g++ executa três passos (Lê $\rightarrow$ Incrementa $\rightarrow$ Escreve). Se a Tarefa A Lê o valor atual (ex: 0) e é preemptada, a Tarefa B pode Ler o mesmo 0, Incrementar e Escrever 1. Quando a Tarefa A retoma, ela continua com o 0 que havia lido, Incrementa e também Escreve 1. Duas operações ocorreram, mas o valor subiu apenas 1 (um incremento foi sobrescrito/perdido).

Intervalo possível: $[2, 2000]$. (O máximo é 2000 se não houver conflitos; o mínimo teórico em entrelaçamentos extremos é 2).

Correção com Mutex:
xSemaphoreTake(mutex_g, portMAX_DELAY);
g++;
xSemaphoreGive(mutex_g);


Alternativa sem bloqueio: Criar variáveis locais (g_A e g_B). Cada tarefa incrementa apenas a sua local (g_A++). No final, soma-se tudo: g = g_A + g_B.

## Q12 Monte uma tabela comparando **fila, semáforo binário, semáforo de contagem, mutex e
event group**: o que sinalizam/carregam, caso de uso típico no curso, e se podem ser usados de ISR.
> 💡 *A tabela da teoria-06, seção 2, é o ponto de partida — complete a coluna "caso de uso no
> curso" com os labs 6, 10 e 14.*

|Mecanismo IPC	|O que sinaliza / carrega | Caso de uso típico no curso | Pode usar em ISR? |
| ---| ---| ---| ---| 
|Fila (Queue) |	Dados brutos (por cópia de valor). | Passar amostras do ADC de uma ISR para uma tarefa processar. | Sim|
|Semáforo Binário| Evento único (0 ou 1). | Sincronizar o acionamento de um botão (ISR) com uma tarefa. | Sim |
|Sem. de Contagem|	Múltiplos eventos/recursos ($N$). | Gerenciar um pool de buffers ou contar múltiplos apertos rápidos. | Sim|
|Mutex |	Exclusão mútua (Token/Chave). | Proteger o barramento I2C ou variáveis globais contra corrupção. | Não| 
|Event Group | Múltiplas flags booleanas (bits). | Aguardar "Wi-Fi conectado" E "IP recebido" para iniciar o MQTT.| Sim |

## Q13 *(estilo Exemplo 6.2)* ISR produz amostras de 4 B a 2 kHz; a tarefa consumidora acorda
a cada 40 ms. Dimensione a fila (itens e bytes) com folga de 2× e explique o que ocorre sem a
folga quando o Wi-Fi rouba CPU por 60 ms.

Dados: Frequência = $2\text{ kHz}$ ($2000\text{ itens/s}$ ou $2\text{ itens/ms}$). 
Consumidor acorda a cada $40\text{ ms}$. 
Tamanho do dado = $4\text{ Bytes}$.

Cálculo da produção em 40 ms: $2\text{ itens/ms} \times 40\text{ ms} = 80\text{ itens}$.

Tamanho com folga (2×): $80 \times 2 = \mathbf{160\text{ itens}}$.

Tamanho em Bytes: $160 \times 4\text{ Bytes} = \mathbf{640\text{ Bytes}}$.

O que ocorre sem folga com Wi-Fi roubando 60 ms: Se a fila fosse dimensionada exatamente para 80 itens, ela ficaria cheia aos 40 ms. 
Como a tarefa consumidora atrasou 20 ms extras (60 ms no total), a ISR produzirá mais 40 itens nesse período ($2\text{ itens} \times 20\text{ ms}$). Como a fila já está cheia e a ISR não pode ser bloqueada aguardando espaço, esses 40 itens serão perdidos/descartados.

## Q14 O que é inversão de prioridade? Descreva o caso Mars Pathfinder em 3 frases e explique como a **herança de prioridade** do mutex do FreeRTOS resolve (e por que semáforo binário não).

![Linha do tempo da inversão de prioridade](https://raw.githubusercontent.com/fabiobento/sis-emb-2026-2/main/assets/figuras/inversao_prioridade.png)

O que é: Ocorre quando uma tarefa de alta prioridade fica bloqueada aguardando um recurso (Mutex) que está em posse de uma tarefa de baixa prioridade, enquanto a de baixa prioridade é preemptada por tarefas de média prioridade.

    Mars Pathfinder em 3 frases: Uma tarefa meteorológica de baixa prioridade assumiu um mutex do barramento de dados. Em seguida, foi preemptada por uma tarefa de comunicações de média prioridade, impedindo-a de terminar e liberar o recurso. Como resultado, a tarefa crítica de gerenciamento de dados de alta prioridade ficou bloqueada esperando o mutex indefinidamente, causando repetidos resets pelo Watchdog Timer.

    A Herança de Prioridade (Priority Inheritance): O FreeRTOS eleva automaticamente a prioridade da tarefa que detém o mutex para o mesmo nível da tarefa bloqueada que o aguarda. Isso impede que tarefas médias preemptem a detentora do mutex, forçando-a a terminar rápido e devolver o recurso.

    Por que semáforo binário não resolve: O semáforo binário não tem o conceito de "dono". O RTOS não sabe qual tarefa pegou o semáforo e, portanto, não sabe a quem aplicar a herança de prioridade.
    
## Q15. Escreva (pseudocódigo ou C) o padrão ISR→tarefa com semáforo binário para o botão da semana 4, incluindo `xSemaphoreGiveFromISR` e `portYIELD_FROM_ISR`, e justifique por que o processamento pesado fica na tarefa.

SemaphoreHandle_t semBotao; // Criado previamente na main

// 1. A Rotina de Interrupção
void IRAM_ATTR isr_botao() {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    // Dá (libera) o semáforo de dentro da ISR
    xSemaphoreGiveFromISR(semBotao, &xHigherPriorityTaskWoken);
    
    // Solicita troca de contexto imediata se a tarefa acordada tiver alta prioridade
    if (xHigherPriorityTaskWoken == pdTRUE) {
        portYIELD_FROM_ISR();
    }
}

// 2. A Tarefa Consumidora
void vTaskBotao(void *pvParameters) {
    while(1) {
        // Bloqueia e aguarda indefinidamente (portMAX_DELAY) pelo semáforo
        if (xSemaphoreTake(semBotao, portMAX_DELAY) == pdTRUE) {
            // ... Processamento pesado: Filtro de debounce, enviar MQTT, att Display ...
        }
    }
}

Justificativa: O processamento pesado fica na tarefa para cumprir a 1ª regra de ouro da ISR (ser a mais curta possível). Delegando a carga pesada para a tarefa através do semáforo, a interrupção executa em poucos microssegundos, permitindo que a CPU processe outras interrupções críticas do sistema, evitando atrasos sistêmicos e o acionamento indesejado do WDT.
