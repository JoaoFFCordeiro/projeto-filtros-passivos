# Projeto de Filtros Passivos para Crossover de Duas Vias

**Autor:** João Francisco Farias Cordeiro  
**Disciplina:** Circuitos de Corrente Alternada - CC44CP  
**Professor:** Lucas Bernardo Zilch  
**Data:** Julho de 2026

---

## Apresentação do Problema

Em sistemas de áudio, o *crossover* passivo é o circuito responsável por direcionar as faixas de frequência apropriadas para cada alto-falante. O woofer (grave) deve reproduzir apenas as frequências baixas, enquanto o tweeter (agudo) deve receber apenas as altas, garantindo a máxima fidelidade e uma transição suave entre eles.

Fui contratado como engenheiro para projetar esse crossover. O desafio é dimensionar os filtros com precisão teórica, mas respeitar a disponibilidade de componentes comerciais, avaliando o impacto prático dessas substituições no desempenho do sistema de áudio.

---

## Objetivos e Especificações

- **Objetivo geral:** Projetar um crossover passivo de duas vias que atenda aos requisitos de áudio com máxima fidelidade e transição suave.
- **Especificações técnicas:**
  - Frequência de corte (\(f_c\)): **2 kHz**
  - Impedância da carga (\(R\)): **8 Ω** (considerada puramente resistiva)
  - Filtros: **Butterworth de 2ª ordem** (máxima planicidade na banda de passagem)
  - **Filtro Passa-Baixas (LPF):** para o woofer.
  - **Filtro Passa-Altas (HPF):** para o tweeter.
  - Utilizar exclusivamente os valores comerciais de indutores e capacitores fornecidos nas tabelas do enunciado.
  - Desenvolver uma ferramenta computacional (MATLAB) para realizar os cálculos, a seleção dos componentes e a geração dos gráficos de Bode comparativos.

---

## Funções de Transferência e Fórmulas de Projeto

### Filtro Passa-Baixas (LPF) – 2ª Ordem Butterworth

A função de transferência normalizada é dada por:

$$
H_{LP}(s) = \frac{\omega_c^2}{s^2 + \sqrt{2}\omega_c s + \omega_c^2}
$$

Para uma carga resistiva \(R\) e frequência de corte \(f_c\) (\(\omega_c = 2\pi f_c\)), os valores dos componentes são:

$$
L = \frac{\sqrt{2} \cdot R}{\omega_c}
$$
$$
C = \frac{1}{\sqrt{2} \cdot \omega_c \cdot R}
$$

### Filtro Passa-Altas (HPF) – 2ª Ordem Butterworth

A função de transferência normalizada é dada por:

$$
H_{HP}(s) = \frac{s^2}{s^2 + \sqrt{2}\omega_c s + \omega_c^2}
$$

Os valores dos componentes para o HPF são:

$$
L = \frac{R}{\sqrt{2} \cdot \omega_c}
$$
$$
C = \frac{\sqrt{2}}{\omega_c \cdot R}
$$

---

## Lógica do Programa (MATLAB)

O código foi desenvolvido em MATLAB e segue a seguinte lógica estruturada:

1. **Parâmetros de entrada:** Define a frequência de corte (\(f_c = 2000\,Hz\)) e a impedância da carga (\(R = 8\,\Omega\)).
2. **Cálculo dos valores ideais:** Utiliza as fórmulas de projeto apresentadas para calcular \(L\) e \(C\) ideais para ambos os filtros.
3. **Seleção de componentes reais:**
   - O programa possui vetores com todos os valores comerciais disponíveis nas Tabelas 1 e 2 do enunciado.
   - Através de uma função auxiliar (`encontrar_mais_proximo`), ele calcula a diferença absoluta entre o valor ideal e cada valor comercial, selecionando aquele com a menor diferença.
4. **Cálculo da resposta em frequência:**
   - Para cada filtro (LPF e HPF), calcula a magnitude (em dB) da função de transferência usando os valores ideais e, em seguida, usando os valores comerciais selecionados.
   - A frequência é varrida em escala logarítmica de 10 Hz a 100 kHz.
5. **Geração dos gráficos:** Plota os gráficos de Bode (magnitude) sobrepostos (Ideal vs. Real) para o LPF e HPF, salvando a figura em `bode_ideal_vs_real.png`.
6. **Análise crítica automática:** O programa encontra a frequência de corte real (ponto de -3 dB) para ambos os filtros com os componentes comerciais e calcula os erros percentuais, exibindo-os no console.

---
## Análise dos Resultados

### Valores Calculados (Ideais vs. Comerciais)

A execução do programa resultou nos seguintes valores:

| Filtro | Componente | Valor Ideal | Valor Comercial | Variação |
|--------|------------|-------------|-----------------|----------|
| **LPF** (Woofer) | Indutor (\(L\)) | 0,900 mH | **0,82 mH** | -8,9% |
| **LPF** (Woofer) | Capacitor (\(C\)) | 7,03 µF | **6,8 µF** | -3,3% |
| **HPF** (Tweeter) | Indutor (\(L\)) | 0,450 mH | **0,47 mH** | +4,4% |
| **HPF** (Tweeter) | Capacitor (\(C\)) | 14,06 µF | **15 µF** | +6,7% |

### Gráfico de Bode Comparativo

A figura abaixo mostra a resposta em magnitude (em dB) dos filtros LPF e HPF. A linha contínua representa a resposta com os componentes ideais (Butterworth), enquanto a linha tracejada representa a resposta utilizando os componentes comerciais selecionados. A linha vertical pontilhada indica a frequência de corte projetada (2 kHz).

![Gráfico de Bode Comparativo](bode_ideal_vs_real.png)

### Análise Crítica

#### Quantificação das Diferenças

- **Deslocamento da frequência de corte:**
  - **LPF Real:** A frequência de corte (ponto de -3 dB) deslocou-se para aproximadamente **2.130 Hz**, resultando em um erro de **+6,5%** em relação ao ideal.
  - **HPF Real:** A frequência de corte deslocou-se para aproximadamente **1.896 Hz**, resultando em um erro de **-5,2%** em relação ao ideal.

- **Impacto na resposta em frequência:** Embora os desvios nos componentes individuais sejam relativamente pequenos (menores que 9%), o deslocamento das frequências de corte causa uma **assimetria na região de transição (crossover)**. Idealmente, ambos os filtros deveriam se cruzar em -3 dB exatamente em 2 kHz. Com os componentes reais, há uma pequena sobreposição (ambos reproduzindo a mesma faixa) ou uma lacuna (nenhum reproduzindo), dependendo da direção do erro.

#### Impacto Prático no Sistema de Áudio

1. **Fidelidade e Coerência:**
   - A mudança na frequência de corte pode fazer com que o woofer reproduza levemente os médios-agudos (no caso do LPF) ou que o tweeter perca parte dos médios-graves.
   - Isso pode resultar em uma ligeira "coloração" do som, onde instrumentos com harmônicos na região de 2 kHz (como voz feminina, violinos e pratos de bateria) podem ter seu timbre alterado.

2. **Audibilidade:**
   - Desvios de até 7% na frequência de corte são considerados **perceptíveis** por ouvintes treinados em sistemas de alta fidelidade, especialmente em testes com varredura senoidal. Para o ouvinte comum em um ambiente doméstico, essa diferença pode ser sutil, mas pode afetar a percepção da "imagem sonora" (*soundstage*).
   - O fator mais crítico é a **diferença de fase** introduzida entre os dois filtros, que pode causar cancelamentos na região do crossover, prejudicando a resposta de transientes.
