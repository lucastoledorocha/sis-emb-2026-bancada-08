# Lista de Exercícios 01 — Semanas 1 a 3

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

