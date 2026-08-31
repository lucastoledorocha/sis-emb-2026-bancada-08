## Grupo: Lucas Toledo Rocha, Lucas Vilella Bergamo e Gabriel dos Santos Oliveira

### Parte A — Introdução (semana 1)

## Q1. Defina sistema embarcado e cite as cinco restrições de projeto discutidas em aula (tempo real, energia, custo, memória, confiabilidade), dando um exemplo de produto em que cada uma é a restrição dominante.

##### Definição de Sistema Embarcado: É um sistema baseado em microprocessador ou microcontrolador projetado para executar uma função dedicada e específica dentro de um sistema maior, frequentemente sujeito a restrições operacionais e recursos computacionais limitados (diferente de um computador de uso geral).

Cinco Restrições de Projeto e Exemplos:

##### Tempo Real: O sistema deve responder a eventos em limites de tempo extremamente curto (prazos/deadlines).
###### Exemplo: Freios ABS e Airbag automotivo (o atraso na atuação causa falha catastrófica).

##### Consumo de Energia (Power/Energy): Operação sob suprimento limitado (bateria/harvesting) por longos períodos.
###### Exemplo: Marca-passo cardíaco implantável

##### Custo Unitário: O custo de produção em larga escala dita a viabilidade econômica do produto.
###### Exemplo: Controle remoto de TV.

##### Memória: Espaço restrito de Flash (código) e SRAM (dados), exigindo código ultra-otimizado.
###### Exemplo: Cartão de memória

##### Confiabilidade: O sistema não pode travar ou falhar de forma não segura durante toda a sua vida útil.
###### Exemplo: Controlador de voo aeroespacial.

## Q2. Classifique como MCU, MPU/SoC ou FPGA a melhor tecnologia para: (a) controle de airbag; (b) roteador Wi-Fi doméstico; (c) protótipo de codec de vídeo proprietário; (d) sensor de umidade a bateria por 2 anos. Justifique com as restrições da Q1.

|Item	|Aplicação | Tecnologia | Justificativa |
| ---| ---| ---| ---| 
|a) |	Controle de airbag | MCU | Tempo real rígido e Confiabilidade: Exige determinismo temporal em nível de microssegundos, necessário também um grande nível de segurança, por conta disso, confiabilidade. |
|b) |	Roteador Wi-Fi doméstico | MPU / SoC |  Capacidade de processamento de rede e Memória: Exige pilha de protocolos pesada, roteamento de pacotes gigabit, sendo necessário um grande processamento de rede e memória |
|c) |	Protótipo de codec de vídeo proprietário | FPGA | Taxa de transferência (Throughput), Paralelismo e Flexibilidade: Processamento massivamente paralelo de fluxos de vídeo em hardware customizado antes de investir em um ASIC definitivo. |
|d) |	Sensor de umidade a bateria (2 anos) | MCU | Energia e Custo: Modos de baixo consumo profundo e custo unitário reduzido | 


## Q3. (estilo Exemplo 1.1) Um rastreador GPS consome 120 mA por 3 s a cada transmissão (1 transmissão/10 min) e 40 µA dormindo. Calcule a corrente média e a autonomia com bateria de 1200 mAh. O que domina o consumo? Que mudança de software dobraria a autonomia?

Dados fornecidos:
Período total (T):10 min = 600 s <br>
Tempo ativo ($t_{\text{ativo}}$): 3 s com corrente ($T_{\text{ativo}}$) = 120 mA  <br>
Tempo dormindo ($t_{\text{sleep}}$): 600 s - 3 s = 597 s , com corrente ($I_{\text{sleep}}$) = 40µA = 0,04 mA <br>
Capacidade da bateria (C): 1200 mAh <br>

Carga por período:

Q = I . T

$Q_{\text{ativo}}$ = 120 mA x 3s = 360 mA.s <br>
$Q_{\text{sleep}}$ = 0,04 mA x 597s = 23,88 mA.s <br>
$Q_{\text{total}}$ = 360 + 23,66 = 383,88 mA.s <br>


Corrente média:

$I_{\text{médio}}$ = $Q_{\text{total}}$/T = 383,88 mA.s / 600 s = 0,6398 mA

Autonomia Bateria:

A = C/ $I_{\text{médio}}$ = 1200 mAh /0,6396 mA = 1785,58 h -> 78,15 dias

O que domina o consumo?Verificando a porcentagem de cada carga, a carga quando ativa é totalmente dominante, a mesma é referente a 93,7% do gasto da carga total (360/383,88 x 100).

Mudança de software para dobrar a autonomia: Dobrar o tempo de dormida aumentando o intervalo de transmissão. Passar de 1 transmissão a cada 10 minutos para 1 transmissão a cada 20 minutos cortaria o impacto do modo ativo quase pela metade, praticamente dobrando a autonomia da bateria.

verificando:

Período total (T):20 min = 1200 s <br>
Tempo ativo ($t_{\text{ativo}}$): 3 s com corrente ($T_{\text{ativo}}$) = 120 mA  <br>
Tempo dormindo ($t_{\text{sleep}}$): 1200 s - 3 s = 1197 s , com corrente ($I_{\text{sleep}}$) = 40µA = 0,04 mA <br>
Capacidade da bateria (C): 1200 mAh <br>

Carga por período:

Q = I . T

$Q_{\text{ativo}}$ = 120 mA x 3s = 360 mA.s <br>
$Q_{\text{sleep}}$ = 0,04 mA x 1197 = 47,88 mA.s <br>
$Q_{\text{total}}$ = 360 + 47,88 = 407,88 mA.s <br>


Corrente média:

$I_{\text{médio}}$ = $Q_{\text{total}}$/T = 407,88 mA.s / 1200 s = 0,3399 mA

Autonomia Bateria:

A = C/ $I_{\text{médio}}$ = 1200 mAh /0,3399 mA = 3530,45 h -> 147 dias

Aumentou de 78 dias para 147 dias


## Q4. Explique a diferença entre tempo real rígido (hard) e brando (soft), com um exemplo de cada num automóvel. Em qual categoria está o nosso projeto de malha de luminosidade (semana 13)?


tempo real rígido: O cumprimento do prazo (deadline) é absoluto. Se o sistema se atrasar, ocorre uma falha crítica ou catastrófica.

Exemplo: Acionamento do Airbag. Um atraso de milissegundos pode ocasionar na lesão de algum usuário do veículo.

tempo real brando: Cumprir o prazo é importante para o desempenho, mas atrasos ocasionais não causam falhas graves.

Exemplo no automóvel: Uso do rádio, como passar faixas ou aumentar o volume

O projeto de malha de luminosidade (semana 13) Encaixa como tempo real Brando. Se o microcontrolador atrasar 50 milissegundos para ajustar o brilho de um LED com base na luz ambiente, o usuário humano possivelmente nem perceberá, e nenhuma consequência crítica ocorrerá.

## Q5. No fluxo de projeto visto em aula (requisitos → particionamento HW/SW → protótipo → validação), explique o papel do simulador (Wokwi) e cite duas limitações dele em relação ao hardware real.

Papel do Simulador: Ele atua na fase entre o testes antes de efetuar o hardware físico. Permite validar a lógica do código em C, testar a máquina de estados, verificar o uso dos registradores (GPIO) sem necessidade de montagens demoradas em protoboard.

Duas limitações em relação ao hardware real:

Idealização de componentes físicos: O Wokwi não considera erros e questões que podem ocorrer com componentes físicos, como por exemplo ruído de contato,  quedas de tensão e correntes de fuga.

Ausência de interferência externa: O simulador não sofre com ruído eletromagnético (EMI), problemas de aterramento ou variações de temperatura que frequentemente afetam questões em hardware físico.

### Parte B — Arquitetura (semana 2)

## Q6. Desenhe (à mão) os diagramas Von Neumann e Harvard e explique por que MCUs adotam majoritariamente Harvard (ou Harvard modificada). Onde o ESP32 se encaixa?

<img width="900" height="1600" alt="image" src="https://github.com/user-attachments/assets/ceb08bc6-7299-4b0e-8730-1eead6ce0d7e" />

Diagrama de Van Neumann

<img width="900" height="1600" alt="image" src="https://github.com/user-attachments/assets/a175d21c-7612-464a-b1d6-620f337675c7" />

Diagrama de Havard


##### Por que os MCUs adotam Harvard (ou Modificada)?

Microcontroladores (MCUs) são frequentemente usados em sistemas embarcados que exigem respostas em tempo real, determinismo e alto desempenho. A arquitetura Harvard permite que a CPU execute grande parte das instruções em um único ciclo, tornando a leitura de sensores, o processamento de sinais e o controle de periféricos muito mais rápidos e previsíveis do que em um sistema Von Neumann tradicional.

##### Onde o ESP32 se encaixa (Harvard Modificada)?

O ESP32 adota uma arquitetura Modificada. Ele extrai o melhor dos dois mundos:
Na CPU (Estilo Harvard): Os núcleos do ESP32 possuem barramentos e memórias   cache estritamente separados para instruções e dados, Isso garante o acesso rápido e simultâneo durante a execução do código.

Na Memória Física (Estilo Von Neumann): Esses barramentos paralelos muitas vezes convergem para um espaço de endereçamento unificado na memória interna, permitindo um orçamento flexível e otimizado

## Q7. (estilo Exemplo 2.1) Um laço de controle precisa executar 12 000 instruções por iteração a 1 kHz. Que fração da CPU de um núcleo de 240 MHz (1 instr/ciclo) isso ocupa? E num AVR de 16 MHz? O que isso diz sobre a escolha Arduino × ESP32 para PDS?

Demanda do laço de controle:

12.000 instruções/iteração x 1.000 iterações/s (1 kHz) = 12.000.000 instruções/s.

A demanda total do processo é de 12 MIPS (Milhões de Instruções Por Segundo).

Fração de uso no ESP32 (núcleo a 240 MHz):

Capacidade máxima teórica de um núcleo: 240 MIPS.

Fração da CPU = 12 MIPS / 240 MIPS = 0,05.

Ocupa 5% do processamento de um núcleo.

Fração de uso no Arduino AVR (16 MHz):

Capacidade máxima teórica: 16 MIPS.

Fração da CPU = 12 MIPS / 16 MIPS = 0,75.

Ocupa 75% do processamento

No ESP32, o mesmo laço consome apenas 5% de um único núcleo, evidenciando que ele é a escolha correta para PDS, pois deixa ampla margem de processamento livre para RTOS, Wi-Fi e algoritmos avançado.

## Q8. Diferencie Flash, SRAM e memória RTC do ESP32 quanto a: volatilidade, velocidade, uso típico (código, dados, deep sleep). Por que variáveis globais "vivem" na SRAM mas seu valor inicial vem da Flash?

