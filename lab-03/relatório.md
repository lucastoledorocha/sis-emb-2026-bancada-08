# Lab 3 — GPIO de entrada e saída + o experimento do bouncing

Imagem Circuito e Código Fonte Wokwi
<img width="1772" height="765" alt="Wolki" src="https://github.com/user-attachments/assets/8e19ec0a-c673-4abc-9226-8c9bf7f0e544" />

Imagem Circuito físico
<img width="1200" height="1600" alt="128ac9df-a1a1-4674-9c97-7c309f3208e5" src="https://github.com/user-attachments/assets/7d69345f-a9b1-456c-aa33-92fb50999322" />

## Parte 1. Resposta da Parte A.4: qual condição do código impede repetição com o botão seguro?

"nivel_ant == 1 && nivel == 0"

Por que impede?
Ao manter o botão pressionado, a variável nivel se mantém em 0.
Como o código atualiza 'nivel_ant = nivel' no final de cada ciclo, na próxima leitura o 'nivel_ant' também será 0. 
A condição exige especificamente a transição de 1 (solto) para 0 (pressionado), o que só ocorre no exato instante em que você abaixa o botão.

## Parte 2. Tabela do experimento C completa + conclusão sobre a janela ideal (Parte C.11).

