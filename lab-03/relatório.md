# Lab 3 — GPIO de entrada e saída + o experimento do bouncing

Imagem Circuito e Código Fonte Wokwi
<img width="1772" height="765" alt="Wolki" src="https://github.com/user-attachments/assets/8e19ec0a-c673-4abc-9226-8c9bf7f0e544" />


## Parte 1. Resposta da Parte A.4: qual condição do código impede repetição com o botão seguro?

"nivel_ant == 1 && nivel == 0"

Por que impede?
Ao manter o botão pressionado, a variável nivel se mantém em 0.
Como o código atualiza 'nivel_ant = nivel' no final de cada ciclo, na próxima leitura o 'nivel_ant' também será 0. 
A condição exige especificamente a transição de 1 (solto) para 0 (pressionado), o que só ocorre no exato instante em que você abaixa o botão.

## Parte 2. Tabela do experimento C completa + conclusão sobre a janela ideal (Parte C.11).

| DEBOUNCE_MS | eventos por 10 pressionadas | eventos "fantasma"|
| :--- | :--- | :--- | 
| 0 | 36 | 26 | 
| 5 | 30 | 20 | 
| 20 | 26 | 16 | 
| 50 | 25 | 15 | 

A janela ideal de debounce é aquela que entregaria exatamente 10 eventos para 10 pressionadas, com 0 eventos "fantasma", de acordo com os testes, seria um valor acima de 50.

## Parte 3. Diff das mudanças da Parte D

As alterações feitas no firmware para adaptar o circuito ao uso do **pull-down externo** foram:

1. **Troca do pino** para evitar conflito com o pino de boot:
   

#### #define BTN   GPIO_NUM_4;


2.    Desativação dos resistores internos para garantir que apenas o resistor físico de 10 kΩ:


#### gpio_pullup_dis(BTN);       // Desabilita o pull-up interno
#### gpio_pulldown_dis(BTN);     // Desabilita o pull-down interno



3.    Inversão da lógica de detecção (estado de repouso e borda de acionamento):


#### int nivel_ant = 0;      
#### if (nivel_ant == 0 && nivel == 1 && agora >= t_ok)


## Parte 4. Foto da montagem final (pull-down) com o resistor de 10 kΩ visível.

Imagem Circuito físico

![Foto Circuito Fisico](https://github.com/lucastoledorocha/sis-emb-2026-bancada-08/blob/main/pictures/IMG-20260826-WA0113.jpg?raw=true)
