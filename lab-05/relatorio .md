## 1. Explicação da Parte A.7
As tarefas coexistem pacatamente porque passam a maior parte do tempo bloqueadas, resultando numa CPU ociosa em cerca de 99,9% do tempo. O escalonamento por prioridade apenas intervém para decidir quem ganha acesso à CPU em caso de disputa simultânea, cenário que raramente ocorre quando as tarefas estão em repouso na maioria do seu período.

## 2. Saída da Parte B.9 e B.10

O monitor serial deixará de imprimir as execuções das tarefas A, B e C e, passados cerca de 5 segundos, apresentará um erro do *Task Watchdog Timer* indicando *starvation*. O erro será semelhante a:
```text
E (5000) task_wdt: Task watchdog got triggered. The following tasks did not reset the watchdog in time:
E (5000) task_wdt:  - IDLE0 (CPU 0)
```

**Resposta (Parte B.10):**
Ao igualar a prioridade da tarefa `HOG` à da tarefa C (prioridade 1), as tarefas A (prioridade 5) e B (prioridade 3) voltam a ser executadas normalmente, pois detêm maior prioridade. A tarefa C passa a alternar a execução com a `HOG` através de *time slicing* a cada *tick* de 10 ms do escalonador, o que faz com que a tarefa C execute apenas em metade do tempo disponível e veja o seu período duplicar.

## 3. Tabela da Parte C e Ativações Perdidas

| Configuração | período médio (ms) | período máx (ms) |
|---|---|---|
| vTaskDelay + corpo 50 ms | ~ 250.0 ms | ~ 250.0 ms |
| vTaskDelayUntil + corpo 50 ms | ~ 200.0 ms | ~ 200.0 ms |

**Cálculo de ativações perdidas (C.12):**
Para um período ideal de 200 ms, seriam esperadas 300 ativações por minuto (60.000 ms / 200 ms). Com a deriva gerando um período de 250 ms, ocorrem apenas 240 ativações (60.000 ms / 250 ms). Logo, perdem-se **60 ativações**, o que representa uma degradação de 20% na taxa de execução.

## 4. Medições de Pilha (Parte D)
O valor devolvido pela função representa a folga da pilha em palavras de 32 bits (onde 1 palavra equivale a 4 bytes no ESP32).
Ao declarar um `char buf[1500]`, o consumo de memória local aumenta em 1500 bytes, o que reduz drasticamente a margem do *high water mark* em cerca de 375 palavras (1500 bytes / 4). Se este acréscimo consumir a folga restante, a aplicação sofre um *stack overflow*.

**Recomendação de Tamanho:**
Seguindo o Exemplo 5.3, o tamanho ideal é o consumo máximo registado somado a uma margem de segurança de 50%. Se, por exemplo, o consumo de pico da tarefa com o *buffer* for de 1800 bytes, o tamanho recomendado no `xTaskCreate` deverá ser de cerca de 2700 bytes (1800 + 900).

## 5. Evidência da Parte E (Dual-core)

```text
[A] core=0  periodo=500.0 ms
[B] core=0  periodo=500.0 ms
[C] core=0  periodo=500.0 ms
[HOG] core=1
```


Na arquitetura do ESP32, o núcleo 0 já é responsável por gerir as tarefas de Wi-Fi, Bluetooth e os serviços de base do sistema. Isolar as cargas pesadas e blocantes no núcleo 1 garante que os serviços críticos não sofrem *starvation*, evitando que o *watchdog* reinicie o sistema.
