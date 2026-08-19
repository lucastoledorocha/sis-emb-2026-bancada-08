# Laboratório 2: ESP-IDF no hardware


## 1. Evidências Visuais
*   **Montagem Física:** <img width="1600" height="900" alt="WhatsApp Image 2026-08-12 at 16 31 53" src="https://github.com/user-attachments/assets/ebdfa4b1-fd9e-4cc5-9bce-15ff7af534ef" />


*   **Monitor Serial:** <img width="900" height="1600" alt="WhatsApp Image 2026-08-12 at 16 30 06" src="https://github.com/user-attachments/assets/85c21416-d939-49bc-bf57-a4cbea65619b" />

## 2. Análise de Memória e Gravação

### Consumo de Memória 

| Memória | Uso (Bytes) |
| --- | --- |
| **DRAM** | Não possui |
| **IRAM** | Não possui |
| **Flash** | 2mb |


Não possui DRAM e IRAM por conta do tipo de arquivo. 

### Layout da Flash
Durante a gravação, foram identificados os seguintes endereços base:
1.  **0x1000**: Bootloader
2.  **0x8000**: Tabela de partições
3.  **0x10000**: Arquivo principal (App)

**Papel do endereço 0x10000:** 
Este offset na memória Flash é onde o executável do programa principal é efetivamente armazenado.

## 3. Análise Elétrica

| Medição | Onde | Valor esperado | Medido (Bancada) |
| :--- | :--- | :--- | :--- |
| **Alimentação do módulo** | pino 5V e GND | 4,75–5,25 V | `5,02 V` |
| **Nível alto no GPIO** | GPI 2 e GND | ~3,3 V | `3,1 V` |
| **Queda no LED** | anodo e catodo | ~1,8–2,1 V | `1,94 V` |

**Memória de Cálculo (Corrente do LED):**
Considerando os valores medidos e a resistência de 220 Ω, a corrente que flui pelo circuito é:

$$ I = \frac{V_{GPIO} - V_{LED}}{R} $$
$$ I = \frac{3,1 - 1,94}{220} $$
$$ I = 5,2\text{ mA} $$

## 4. Conceito de Atomicidade

A principal vantagem de utilizar os registradores dedicados `W1TS` (Write 1 To Set) e `W1TC` (Write 1 To Clear) é a garantia de atomicidade via hardware. Na abordagem clássica de "lê-modifica-escreve" com o operador `|=`, o processador precisa de vários ciclos de clock para ler o estado, alterar o bit e reescrever o dado. Com o `W1TS`, informamos em uma única instrução indivisível de escrita qual pino deve ser ativado, e o próprio hardware lógico altera apenas aquele pino, tornando o sistema imune a corrupções por interrupções concorrentes.
