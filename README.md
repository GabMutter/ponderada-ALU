# ALU — Unidade Lógica e Aritmética em Digital

Projeto de implementação de uma **ALU (Arithmetic Logic Unit)** de 8 bits desenvolvida do zero no simulador [Digital](https://github.com/hneemann/Digital), com hierarquia de componentes construída progressivamente a partir de portas lógicas básicas.

---

## Sumário

- [Visão Geral](#visão-geral)
- [Ferramentas](#ferramentas)
- [Hierarquia de Componentes](#hierarquia-de-componentes)
- [Componentes](#componentes)
  - [Soma](#soma)
  - [Subtração](#subtração)
  - [Multiplicação](#multiplicação)
  - [Operações Lógicas](#operações-lógicas)
  - [Deslocamento de Bits](#deslocamento-de-bits)
  - [ALU Principal](#alu-principal)
- [Operações Suportadas pela ALU](#operações-suportadas-pela-alu)
- [Como Executar](#como-executar)
- [Estrutura de Arquivos](#estrutura-de-arquivos)

---

## Visão Geral

Este projeto implementa uma ALU de 8 bits capaz de realizar operações aritméticas e lógicas. A construção seguiu uma abordagem **bottom-up**: começando pelos somadores de 1 bit, encadeando-os em somadores de 4 e 8 bits, e assim por diante, até compor a ALU completa com multiplexador de seleção de operação.

A ALU recebe dois operandos de 8 bits (**A** e **B**), um sinal de **Select** para escolha da operação, e um clock (**CLK**) para os registradores internos. O resultado é disponibilizado nas saídas **S** (resultado), **AC** (acumulador) e **MQ** (quociente/produto parcial).

---

## Ferramentas

| Ferramenta | Versão | Uso |
|---|---|---|
| [Digital](https://github.com/hneemann/Digital) | Qualquer recente | Simulação dos circuitos `.dig` |
| Java | 11+ | Necessário para rodar o Digital |

---

## Hierarquia de Componentes

O projeto foi construído de forma hierárquica, onde cada componente reutiliza os anteriores:

```
ALU
├── soma-8bit
│   └── soma-4bit (x2)
│       └── soma-1bit (x4)
├── subtração-8bits
│   └── subtração-4bits (x2)
│       └── subtração-1bit (x4)
├── multiplicador-8bit
│   ├── somador_16bits
│   │   └── soma-8bit (x2)
│   └── shift_left (16 bits)
├── NAND (8 bits)
├── xor_8bits
├── shiftLeft-8bits
└── shiftRight-8bits
```

---

## Componentes

### Soma

#### `soma-1bit.dig` — Somador Completo de 1 bit

Implementa um **full adder** usando portas XOR e AND.

| Entrada | Bits | Descrição |
|---|---|---|
| A | 1 | Operando A |
| B | 1 | Operando B |
| Cin | 1 | Carry de entrada |

| Saída | Bits | Descrição |
|---|---|---|
| S | 1 | Soma |
| Cout | 1 | Carry de saída |

---

#### `soma-4bit.dig` — Somador de 4 bits

Encadeia **4 instâncias** do `soma-1bit` com propagação de carry (ripple carry adder).

| Entrada | Bits | Descrição |
|---|---|---|
| A | 4 | Operando A |
| B | 4 | Operando B |
| Cin | 1 | Carry de entrada |

| Saída | Bits | Descrição |
|---|---|---|
| S | 4 | Soma |
| Cout | 1 | Carry de saída |

---

#### `soma-8bit.dig` — Somador de 8 bits

Encadeia **2 instâncias** do `soma-4bit`.

| Entrada | Bits | Descrição |
|---|---|---|
| A | 8 | Operando A |
| B | 8 | Operando B |
| Cin | 1 | Carry de entrada |

| Saída | Bits | Descrição |
|---|---|---|
| S | 8 | Soma |
| Cout | 1 | Carry de saída |

---

#### `somador_16bits.dig` — Somador de 16 bits

Encadeia **2 instâncias** do `soma-8bit`. Utilizado internamente pelo multiplicador.

| Entrada | Bits | Descrição |
|---|---|---|
| A | 16 | Operando A |
| B | 16 | Operando B |
| Cin | 1 | Carry de entrada |

| Saída | Bits | Descrição |
|---|---|---|
| S | 16 | Soma |
| Cout | 1 | Carry de saída |

---

### Subtração

#### `subtração-1bit.dig` — Subtrator Completo de 1 bit

Implementa um **full subtractor** com borrow de entrada e saída.

| Entrada | Bits | Descrição |
|---|---|---|
| A | 1 | Minuendo |
| B | 1 | Subtraendo |
| Cin | 1 | Borrow de entrada |

| Saída | Bits | Descrição |
|---|---|---|
| S | 1 | Diferença |
| Cout | 1 | Borrow de saída |

---

#### `subtração-4bits.dig` — Subtrator de 4 bits

Encadeia **4 instâncias** do `subtração-1bit`.

---

#### `subtração-8bits.dig` — Subtrator de 8 bits

Encadeia **2 instâncias** do `subtração-4bits`.

| Entrada | Bits | Descrição |
|---|---|---|
| A | 8 | Minuendo |
| B | 8 | Subtraendo |
| Cin | 1 | Borrow de entrada |

| Saída | Bits | Descrição |
|---|---|---|
| S | 8 | Diferença |
| Cout | 1 | Borrow de saída |

---

### Multiplicação

#### `multiplicador-2bits.dig` — Multiplicador de 2 bits
 
Multiplicador de 2×2 bits que produz um resultado de **4 bits**, construído com portas AND e somadores parciais (`soma-1bit`). O princípio é o mesmo da multiplicação decimal tradicional: multiplica-se cada bit do multiplicador pelos bits do multiplicando, gerando **produtos parciais** que são somados ao final.
 
##### Como funciona
 
Na multiplicação binária, cada bit do multiplicador age como um "seletor": se for `1`, o multiplicando é copiado; se for `0`, o resultado é zero. Isso é exatamente o que uma porta **AND** faz — e é por isso que os produtos parciais são gerados com ANDs bit a bit.
 
Considere `A = A1 A0` e `B = B1 B0`:
 
```
        A1  A0       ← multiplicando
      × B1  B0       ← multiplicador
      ─────────
    B0·A1  B0·A0     ← produto parcial 0  (linha do B0)
 B1·A1  B1·A0  0     ← produto parcial 1  (linha do B1, deslocado 1 posição à esquerda)
 ────────────────
   S3   S2   S1  S0  ← resultado final (4 bits)
```
 
O **primeiro produto parcial** (linha de B0) já vai direto para os bits menos significativos do resultado — não há nada para somar ainda. O **segundo produto parcial** (linha de B1) está deslocado uma posição para a esquerda, então seus bits se sobrepõem com os do primeiro. É aí que entram os `soma-1bit`: eles somam os produtos que se sobrepõem, propagando o carry quando necessário.
 
##### Passo a passo com exemplo: `3 × 3` (ou seja, `11 × 11` em binário)
 
```
        1  1       (A = 3)
      × 1  1       (B = 3)
      ───────
        1  1       ← produtos parciais da linha B0=1: AND(1,1)=1, AND(1,1)=1... wait
```
 
Expandindo com os ANDs:
```
  B0·A1 = 1·1 = 1
  B0·A0 = 1·1 = 1
  B1·A1 = 1·1 = 1
  B1·A0 = 1·1 = 1
```
 
Montando as linhas:
```
  Posição:  3    2    1    0
            ─    ─    ─    ─
  PP0:           1    1         (B0·A1 na pos 1, B0·A0 na pos 0)
  PP1:      1    1              (B1·A1 na pos 2, B1·A0 na pos 1)
```
 
Somando coluna por coluna da direita para a esquerda:
 
| Passo | Operação | Resultado | Carry |
|---|---|---|---|
| Posição 0 | B0·A0 = 1 | **S0 = 1** | — |
| Posição 1 | B0·A1 + B1·A0 = 1+1 | **S1 = 0** | Cout = 1 |
| Posição 2 | B1·A1 + Cout anterior = 1+1 | **S2 = 0** | Cout = 1 |
| Posição 3 | Cout anterior = 1 | **S3 = 1** | — |
 
**Resultado: `S3 S2 S1 S0` = `1001` = 9 ✓** (pois 3 × 3 = 9)
 
##### Tabela verdade completa (todas as combinações de A e B de 0 a 3)
 
| A (dec) | B (dec) | A (bin) | B (bin) | Resultado (bin) | Resultado (dec) |
|---|---|---|---|---|---|
| 0 | 0 | 00 | 00 | 0000 | 0 |
| 0 | 1 | 00 | 01 | 0000 | 0 |
| 0 | 2 | 00 | 10 | 0000 | 0 |
| 0 | 3 | 00 | 11 | 0000 | 0 |
| 1 | 0 | 01 | 00 | 0000 | 0 |
| 1 | 1 | 01 | 01 | 0001 | 1 |
| 1 | 2 | 01 | 10 | 0010 | 2 |
| 1 | 3 | 01 | 11 | 0011 | 3 |
| 2 | 0 | 10 | 00 | 0000 | 0 |
| 2 | 1 | 10 | 01 | 0010 | 2 |
| 2 | 2 | 10 | 10 | 0100 | 4 |
| 2 | 3 | 10 | 11 | 0110 | 6 |
| 3 | 0 | 11 | 00 | 0000 | 0 |
| 3 | 1 | 11 | 01 | 0011 | 3 |
| 3 | 2 | 11 | 10 | 0110 | 6 |
| 3 | 3 | 11 | 11 | 1001 | 9 |
 
---

#### `multiplicador-8bit.dig` — Multiplicador de 8 bits

Implementa a multiplicação de dois operandos de 8 bits pelo método de **somas parciais com deslocamento**, produzindo um resultado de 16 bits. Utiliza internamente o `somador_16bits` e o `shift_left`.

| Entrada | Bits | Descrição |
|---|---|---|
| A | 8 | Multiplicando |
| B | 8 | Multiplicador |

| Saída | Bits | Descrição |
|---|---|---|
| S | 16 | Produto |

---

### Operações Lógicas

#### `NAND.dig` — Porta NAND de 8 bits

Realiza a operação NAND bit a bit entre dois operandos de 8 bits (AND seguido de NOT).

| Entrada | Bits |
|---|---|
| A | 8 |
| B | 8 |

| Saída | Bits |
|---|---|
| S | 8 |

---

#### `xor_8bits.dig` — XOR de 8 bits

Realiza a operação XOR bit a bit entre dois operandos de 8 bits.

| Entrada | Bits |
|---|---|
| A | 8 |
| B | 8 |

| Saída | Bits |
|---|---|
| S | 8 |

---

### Deslocamento de Bits

#### `shiftLeft-8bits.dig` — Shift Left de 8 bits

Desloca os 8 bits do operando **uma posição à esquerda**, inserindo 0 no bit menos significativo.

| Entrada | Bits |
|---|---|
| in | 8 |

| Saída | Bits |
|---|---|
| out | 8 |

---

#### `shiftRight-8bits.dig` — Shift Right de 8 bits

Desloca os 8 bits do operando **uma posição à direita**, inserindo 0 no bit mais significativo.

| Entrada | Bits |
|---|---|
| in | 8 |

| Saída | Bits |
|---|---|
| out | 8 |

---

#### `shift_left.dig` — Shift Left de 16 bits

Versão de 16 bits do shift left. Usada pelo multiplicador de 8 bits para deslocar produtos parciais.

| Entrada | Bits |
|---|---|
| in | 16 |

| Saída | Bits |
|---|---|
| out | 16 |

---

### ALU Principal

#### `ALU.dig` — Unidade Lógica e Aritmética

Integra todos os módulos acima com um **multiplexador** controlado pelo sinal `Select` para escolher qual operação será executada. Possui registradores internos (**AC** e **MQ**) sincronizados pelo clock.

| Entrada | Bits | Descrição |
|---|---|---|
| A | 8 | Operando A |
| B | 8 | Operando B |
| Select | 3 | Seleção de operação |
| CLK | 1 | Clock |

| Saída | Bits | Descrição |
|---|---|---|
| S | 8 | Resultado da operação |
| AC | 8 | Registrador acumulador |
| MQ | 8 | Registrador quociente/produto |

---

## Operações Suportadas pela ALU

| Select | Operação | Módulo usado |
|---|---|---|
| 000 | Soma (A + B) | `soma-8bit` |
| 001 | Subtração (A − B) | `subtração-8bits` |
| 010 | Multiplicação (A × B) | `multiplicador-8bit` |
| 011 | NAND (A NAND B) | `NAND` |
| 100 | XOR (A ⊕ B) | `xor_8bits` |
| 101 | Shift Left de A | `shiftLeft-8bits` |
| 110 | Shift Right de A | `shiftRight-8bits` |
| 111 | Divisão (A ÷ B) | Componente nativo Digital |

> **Nota:** Os valores de `Select` acima são uma referência baseada na estrutura do multiplexador. Verifique o circuito `ALU.dig` no Digital para confirmar o mapeamento exato de cada seletor.

---

## Estrutura de Arquivos

```
.
├── ALU.dig                  # Circuito principal da ALU
├── soma-1bit.dig            # Full adder de 1 bit
├── soma-4bit.dig            # Somador de 4 bits (4× soma-1bit)
├── soma-8bit.dig            # Somador de 8 bits (2× soma-4bit)
├── somador_16bits.dig       # Somador de 16 bits (2× soma-8bit)
├── subtração-1bit.dig       # Full subtractor de 1 bit
├── subtração-4bits.dig      # Subtrator de 4 bits
├── subtração-8bits.dig      # Subtrator de 8 bits
├── multiplicador-2bits.dig  # Multiplicador de 2 bits
├── multiplicador-8bit.dig   # Multiplicador de 8 bits
├── NAND.dig                 # NAND de 8 bits
├── xor_8bits.dig            # XOR de 8 bits
├── shiftLeft-8bits.dig      # Shift left de 8 bits
├── shiftRight-8bits.dig     # Shift right de 8 bits
├── shift_left.dig           # Shift left de 16 bits
└── README.md
```
