# Instrumentação Industrial II
## Aula Prática 09: Medição de Temperatura com Termopar e Transmissor TT301 — Configuração e Calibração

Esta prática apresenta a base teórica e a execução, no simulador **LasecSimul**, da medição de temperatura de um forno elétrico com um **termopar tipo K** ligado ao transmissor **Smar TT301 (HART)**, configurado e calibrado pelo **PACTware**.

O material segue a ordem de trabalho de uma planta real, com foco em:

- efeito Seebeck, leis dos circuitos termoelétricos e tipos de termopar;
- funções de referência **NIST ITS-90 / IEC 60584-1** e classes de tolerância;
- **junta de referência** (junta fria), sua compensação e os **cabos de extensão e de compensação**;
- erros típicos de instalação: cabo de cobre, polaridade invertida, tipo trocado e junta fria;
- configuração do TT301 para termopar: sensor, junta fria, faixa, unidade, amortecimento, burnout e display;
- calibração do **transmissor** com calibrador em modo termopar e com fonte de mV, verificação da **junta fria**, **trim de leitura**, **trim de corrente** e calibração com referência;
- calibração do **termopar** em forno de calibração, teste de burnout, teste de malha e registro.

---

## Índice

1. [Introdução](#1-introdução)
2. [O termopar](#2-o-termopar)
3. [Junta de referência e cabos](#3-junta-de-referência-e-cabos)
4. [O TT301 com termopar](#4-o-tt301-com-termopar)
5. [A planta no LasecSimul](#5-a-planta-no-lasecsimul)
6. [Arquivos da prática](#6-arquivos-da-prática)
7. [Como executar](#7-como-executar)
8. [Conexão com o PACTware](#8-conexão-com-o-pactware)
9. [Configuração do TT301](#9-configuração-do-tt301)
10. [Calibração](#10-calibração)
11. [Bancada virtual no LasecSimul](#11-bancada-virtual-no-lasecsimul)
12. [Sugestão de atividade experimental](#12-sugestão-de-atividade-experimental)
13. [Conclusão](#13-conclusão)
14. [Referências](#14-referências)
15. [Apêndice: lista de imagens](#15-apêndice-lista-de-imagens)

---

## 1. Introdução

O **termopar** é o sensor de temperatura mais usado na indústria. É simples, robusto, barato, responde rápido e mede de temperaturas criogênicas até perto de 1800 °C. Ele é usado em:

- fornos de tratamento térmico, fornos de reaquecimento e muflas;
- caldeiras, fornalhas e gases de combustão;
- extrusoras, injetoras e moldes;
- reatores, craqueamento e incineradores;
- fusão de metais e vidro (com termopares de platina).

O termopar **gera** uma pequena tensão, da ordem de microvolts por grau Celsius. Essa tensão depende da diferença de temperatura entre as duas extremidades dos fios. Por isso, medir com termopar exige saber também a temperatura da extremidade que fica no instrumento: a **junta de referência**. O transmissor de temperatura (aqui o **TT301** da Smar) mede os milivolts e a temperatura dos próprios bornes. Com isso ele compensa a junta de referência, lineariza pela tabela do tipo de termopar e transmite a temperatura em 4–20 mA com HART.

**Figura 1. Termopar industrial com tubo de proteção e cabeçote**

![Figura 1](assets/fig01_termopar_industrial.jpg)

> **Imagem a inserir:** foto de um termopar industrial completo:
> - tubo de proteção (metálico ou cerâmico);
> - cabeçote de ligação aberto, mostrando o bloco de terminais com os dois fios do termopar (marque o + e o −);
> - cabo de extensão saindo do cabeçote, com a cor da capa visível (verde para o tipo K pela IEC);
> - se possível, um TT301 montado ao lado.

---

## 2. O termopar

### 2.1 Efeito Seebeck

Quando há diferença de temperatura ao longo de um fio metálico, surge nele uma pequena força eletromotriz (FEM), característica do material. Unindo dois fios de materiais **diferentes** (A e B) em uma ponta, a **junta de medição**, e mantendo as outras pontas a uma temperatura de referência, aparece entre essas pontas a tensão:

$$
E = \int_{T_r}^{T} \left(S_A - S_B\right)\, dT = E_{AB}(T) - E_{AB}(T_r)
$$

- `S_A` e `S_B`: coeficientes de Seebeck dos dois materiais (µV/°C);
- `T`: temperatura da junta de medição;
- `T_r`: temperatura da **junta de referência**, onde os fios do termopar encontram o cobre do instrumento;
- `E_AB(t)`: função de referência do par, tabelada com a junta de referência a **0 °C**.

A tensão **não** depende de como a temperatura varia ao longo do caminho. Depende só das temperaturas das duas juntas, desde que cada fio seja homogêneo.

### 2.2 Leis dos circuitos termoelétricos

1. **Circuito homogêneo:** um fio homogêneo não gera FEM líquida, por mais que a temperatura varie ao longo dele. Só as juntas entre materiais diferentes contam.
2. **Metais intermediários:** inserir um terceiro metal (por exemplo, o cobre dos bornes) não altera a FEM, desde que as duas junções com ele estejam **na mesma temperatura**. É por isso que se pode ligar o termopar a um instrumento.
3. **Temperaturas sucessivas:** `E(T₁→T₃) = E(T₁→T₂) + E(T₂→T₃)`. É essa lei que permite **compensar a junta de referência**: `E(T, 0 °C) = E(T, T_r) + E(T_r, 0 °C)`.

### 2.3 Tipos de termopar

A IEC 60584-1 padroniza os tipos pela letra. A sensibilidade abaixo foi calculada pelas funções de referência do NIST (seção 2.4).

| Tipo | Fio + / Fio − | Faixa de uso típica | E(100 °C) | Sensibilidade a 25 °C | Observações |
|---|---|---|---:|---:|---|
| **K** | NiCr (Chromel) / NiAl (Alumel) | −200 a 1260 °C | 4,096 mV | 40,5 µV/°C | o mais usado; bom em atmosfera oxidante |
| **J** | Fe / CuNi (Constantan) | 0 a 760 °C | 5,269 mV | 51,8 µV/°C | o ferro oxida; atmosferas redutoras |
| **T** | Cu / CuNi | −200 a 350 °C | 4,279 mV | 40,7 µV/°C | baixas temperaturas, boa exatidão |
| **E** | NiCr / CuNi | −200 a 900 °C | 6,319 mV | 61,0 µV/°C | maior sensibilidade |
| **N** | NiCrSi (Nicrosil) / NiSi (Nisil) | −200 a 1260 °C | 2,774 mV | 26,8 µV/°C | mais estável que o K em alta temperatura |
| **R** | Pt-13 %Rh / Pt | 0 a 1600 °C | 0,647 mV | 5,9 µV/°C | metal nobre; laboratório e alta temperatura |
| **S** | Pt-10 %Rh / Pt | 0 a 1600 °C | 0,646 mV | 6,0 µV/°C | metal nobre; padrão de calibração |
| **B** | Pt-30 %Rh / Pt-6 %Rh | 600 a 1700 °C | 0,033 mV | ≈ 0 µV/°C | quase não gera tensão perto do ambiente |

O TT301 também aceita os tipos L e U (DIN 43710), L (GOST) e W5Re/W26Re.

### 2.4 Funções de referência (NIST ITS-90)

As tabelas oficiais de FEM × temperatura são as do NIST (*Monograph 175*), adotadas pela IEC 60584-1. Para cada faixa, a FEM é um polinômio da temperatura `t` (°C), com a junta de referência a 0 °C:

$$
E(t) = \sum_{i=0}^{n} c_i\, t^i
$$

Para o tipo K acima de 0 °C, soma-se um termo exponencial:

$$
E(t) = \sum_{i=0}^{9} c_i\, t^i + a_0\, e^{a_1 (t - a_2)^2}
$$

O NIST publica também os polinômios **inversos** `t(E)`, com erro de até ±0,06 °C. São eles que o transmissor usa para **linearizar**.

**Tabela do tipo K (junta de referência a 0 °C)**

| t (°C) | 0 | 25 | 100 | 250 | 500 | 750 | 1000 | 1200 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| E (mV) | 0,000 | 1,000 | 4,096 | 10,153 | 20,644 | 31,213 | 41,276 | 48,838 |

A curva não é reta: entre 0 e 100 °C o tipo K dá 41,0 µV/°C; entre 900 e 1000 °C, 39,5 µV/°C. Por isso a linearização é feita pelo polinômio do tipo, e não por um ganho fixo.

### 2.5 Classes de tolerância (IEC 60584-1)

A tolerância vale para o termopar novo, comparado com a tabela de referência. Usa-se o **maior** dos dois valores:

| Tipo | Classe 1 | Classe 2 |
|---|---|---|
| K e N | ±1,5 °C ou ±0,004·\|t\| (−40 a 1000 °C) | ±2,5 °C ou ±0,0075·\|t\| (−40 a 1200 °C) |
| J | ±1,5 °C ou ±0,004·\|t\| (−40 a 750 °C) | ±2,5 °C ou ±0,0075·\|t\| (−40 a 750 °C) |
| T | ±0,5 °C ou ±0,004·\|t\| (−40 a 350 °C) | ±1,0 °C ou ±0,0075·\|t\| (−40 a 350 °C) |
| E | ±1,5 °C ou ±0,004·\|t\| | ±2,5 °C ou ±0,0075·\|t\| |

Para os tipos R, S e B, consulte a norma.

Exemplo: um termopar K **classe 2** a 1000 °C pode errar até **±7,5 °C** sem estar defeituoso; um **classe 1**, até ±4,0 °C. Esse erro é bem maior que o do transmissor: em geral, o **sensor** domina a incerteza da malha de temperatura.

### 2.6 Construção e tempo de resposta

- **Isolação mineral (MI):** os fios ficam em pó de óxido de magnésio compactado dentro de uma bainha metálica.
- **Junta isolada, aterrada ou exposta:** a aterrada responde mais rápido, mas fica ligada à terra. A isolada evita laços de terra.
- **Tubo de proteção** (metálico ou cerâmico): protege do processo e atrasa a resposta. Em forno, o atraso é de segundos a minutos.
- **Deriva:** termopares de metal base (K, J, N) mudam de característica com o tempo em alta temperatura. Por isso precisam de verificação periódica.

**Figura 2. FEM × temperatura dos termopares padronizados**

![Figura 2](assets/fig02_curvas_fem_tipos.png)

> **Imagem a inserir:** gráfico da FEM (mV, eixo vertical) em função da temperatura (°C, eixo horizontal, de 0 a 1800 °C), traçado pelas funções de referência do NIST.
> - Uma curva para cada tipo: E, J, K, N, T, R, S e B, com o **K** em destaque (linha mais grossa).
> - Marque os pontos da tabela do tipo K da seção 2.4.
> - Pode ser feito em Python, com o módulo `scripts/thermocouple_its90.py` do LasecSimul (função `emf`), ou numa planilha com as tabelas do NIST.

**Figura 3. Circuito do termopar**

![Figura 3](assets/fig03_circuito_termopar.png)

> **Imagem a inserir:** desenho com:
> - a junta de medição no processo (temperatura T);
> - os dois fios A (+) e B (−);
> - as duas junções com o cobre nos bornes do instrumento, na mesma temperatura `T_r` (junta de referência);
> - o voltímetro medindo `E = E(T) − E(T_r)`;
> - setas indicando onde está cada temperatura e a equação escrita ao lado.

---

## 3. Junta de referência e cabos

### 3.1 Por que compensar

O instrumento só enxerga a diferença `E(T) − E(T_r)`. Para achar `T` pela tabela (feita com referência a 0 °C), ele precisa **somar** `E(T_r)`:

$$
E(T, 0\,°C) = E_{medida} + E(T_r) \quad\Rightarrow\quad T = E^{-1}\big(E_{medida} + E(T_r)\big)
$$

Antigamente a junta de referência ficava num banho de gelo (`T_r` = 0 °C). Hoje o transmissor tem um **sensor de temperatura nos bornes** (no TT301, um Pt100 na borneira) e faz a **compensação da junta fria** (CJC) automaticamente.

Exemplo com o tipo K, forno a 500 °C e bornes a 25 °C:

- tensão nos bornes: `E(500) − E(25) = 20,644 − 1,000 = 19,644 mV`;
- com a compensação: `19,644 + 1,000 = 20,644 mV` → **500 °C**;
- sem a compensação: `19,644 mV` → **476,5 °C** (erro de −23,5 °C).

O erro sem compensação é de aproximadamente a temperatura ambiente. Ele varia com o dia, com o sol e com a estação.

### 3.2 Cabos de extensão e de compensação

A junta de referência precisa estar **onde a temperatura é medida**: nos bornes do transmissor. Se o transmissor fica longe do termopar, o caminho até ele tem de ser feito com fios que se comportam como o próprio termopar:

- **Cabo de extensão (X):** mesmas ligas do termopar (por exemplo, **KX**). Vale numa faixa de temperatura ambiente maior (KX: −25 a +200 °C pela IEC 60584-3).
- **Cabo de compensação (C):** ligas diferentes, mais baratas, com a mesma FEM numa faixa limitada (por exemplo, **KCA** até 150 °C e **KCB** até 100 °C). Para R e S usa-se um só cabo, o **RCB/SCB** (cobre / cobre-níquel).
- **Cabo de cobre comum:** **não** serve. Com cobre, a junta de referência fica onde o termopar termina (no cabeçote, perto do processo e quente), e o transmissor compensa a temperatura errada.

### 3.3 Código de cores

| Tipo | IEC 60584-3 (+ / − / capa) | ANSI MC96.1 (+ / − / capa) |
|---|---|---|
| K | verde / branco / verde | amarelo / vermelho / amarelo |
| J | preto / branco / preto | branco / vermelho / preto |
| T | marrom / branco / marrom | azul / vermelho / azul |
| E | violeta / branco / violeta | roxo / vermelho / roxo |
| N | rosa / branco / rosa | laranja / vermelho / laranja |
| R e S | laranja / branco / laranja | — |

Na IEC o fio **negativo é sempre branco**; na ANSI, o negativo é **sempre vermelho**. Muitas plantas brasileiras têm as duas normas: confira sempre a polaridade com o desenho e com o multímetro. No Brasil, as tolerâncias e a identificação dos cabos estão na **ABNT NBR 13774**.

### 3.4 Erros típicos de instalação

| Erro | O que acontece | Exemplo (tipo K, processo a 625 °C, bornes a 25 °C) |
|---|---|---|
| Cabo de cobre a partir do cabeçote | a junta de referência fica no cabeçote quente | cabeçote a 60 °C → lê **591,2 °C** (−33,8 °C) |
| Polaridade invertida (no cabo ou nos bornes) | a tensão muda de sinal; a leitura cai quando o processo aquece | lê **−200 °C** (fim da faixa do tipo) |
| Polaridade invertida também no cabo de extensão | os dois erros se somam ao longo do cabo | erro proporcional à diferença de temperatura no cabo |
| Tipo configurado errado | a linearização é de outro termopar | K configurado como **J** → **479,4 °C** |
| Junta fria desabilitada | falta `E(T_borneira)` | **601,4 °C** (−23,6 °C) |
| Sensor de junta fria descalibrado | a compensação usa a temperatura errada | borneira lida como 30 °C → **629,8 °C** (+4,8 °C) |
| Emenda com bornes de cobre num ponto quente | junções parasitas em temperaturas diferentes | erro variável com o aquecimento do ponto |

Dica de campo: um termopar com polaridade invertida dá leitura que **anda ao contrário**. Ao aquecer o forno, a temperatura indicada cai.

**Figura 4. Cores dos cabos de extensão**

![Figura 4](assets/fig04_cores_cabos.png)

> **Imagem a inserir:** quadro com os pedaços de cabo de extensão dos tipos K, J, T, E e N lado a lado, em duas colunas: IEC 60584-3 (fio negativo branco) e ANSI MC96.1 (fio negativo vermelho).
> - Mostre a capa e os dois fios desencapados de cada cabo, com o + e o − indicados.
> - Pode ser foto dos cabos do laboratório ou desenho baseado na tabela da seção 3.3.

**Figura 5. Instalação correta e instalação com cabo de cobre**

![Figura 5](assets/fig05_cabo_extensao_x_cobre.png)

> **Imagem a inserir:** desenho em duas partes, cada uma com forno, termopar, cabeçote e TT301 na sala de painéis, indicando a temperatura de cada ponto.
> - **Em cima, correto:** cabeçote a 60 °C, cabo de extensão KX (verde/branco) até o TT301 a 25 °C. Junta de referência nos bornes do TT301, leitura correta.
> - **Embaixo, errado:** cabo de cobre a partir do cabeçote. Junta de referência no cabeçote a 60 °C, leitura 34 °C abaixo.
> - Destaque em vermelho onde fica a junta de referência em cada caso.

---

## 4. O TT301 com termopar

### 4.1 Ligação nos bornes

Pela Figura 1.9 do manual, o termopar (ou gerador de mV) é ligado com o **+ no borne 2** e o **− no borne 3**. Os bornes **1, 3 e 4 ficam ligados entre si**: jumper de 1 para 4 e de 3 para 4. Nunca ligue a alimentação nos bornes 1 a 4.

O manual também mostra as ligações com **dois termopares** nos modos diferencial, backup, média, máximo e mínimo. Esses modos não fazem parte desta prática.

### 4.2 Tipos aceitos, faixas e span mínimo

Ao escolher o sensor, o TT301 carrega sozinho os limites e o span mínimo (Tabela 6.1 do manual):

| Termopar | Limites (°C) | Span mínimo (°C) |
|---|---|---:|
| B | 100 a 1800 | 50 |
| E | −100 a 1000 | 20 |
| J | −150 a 750 | 30 |
| **K** | **−200 a 1350** | **60** |
| N | −100 a 1300 | 50 |
| R e S | 0 a 1750 | 40 |
| T | −200 a 400 | 15 |

### 4.3 Cadeia de sinal

```text
bornes 2-3 (mV) → trim de leitura → + E(borneira) [junta fria] → linearização do tipo → faixa LRV/URV → % → saída → trim de corrente → 4-20 mA
```

- O **trim de leitura** (zero e ganho) corrige a medida de mV.
- A **junta fria** soma a FEM equivalente à temperatura da borneira, medida pelo sensor interno.
- A **linearização** converte mV em temperatura pela tabela do tipo configurado.
- A **faixa** (LRV/URV) só define a temperatura de 4 e de 20 mA.
- O **trim de corrente** corrige o conversor de saída.

### 4.4 Junta fria do TT301

- A compensação pode ser **habilitada ou desabilitada** em **Configuration → Sensor → Cold Junction**. Em operação normal ela fica **habilitada**.
- O manual especifica uma **rejeição da compensação de junta fria de 60:1** (referência 25,0 ± 0,3 °C): uma variação de 6 °C na borneira causa no máximo 0,1 °C de erro.
- O sensor da borneira tem um ajuste próprio, o **Temperature Trim**. O manual não o recomenda (o sensor é um Pt100 de boa precisão), mas permite um pequeno ajuste quando necessário.
- O efeito da temperatura ambiente na entrada de termopar (E, J, K, N) é de **±0,03 % da entrada em mV ou 0,01 mV**, o que for maior, por 10 °C de variação. No tipo K, 0,01 mV equivale a cerca de 0,25 °C.

### 4.5 Burnout

Se o termopar romper, o TT301 detecta o circuito aberto e leva a saída ao **burnout**: **21 mA (High)** ou **3,6 mA (Low)**. Num forno, o High costuma ser a escolha segura: a malha de controle vê temperatura alta e corta o aquecimento.

**Figura 6. Ligação do termopar nos bornes do TT301**

![Figura 6](assets/fig06_tt301_ligacao_termopar.png)

> **Imagem a inserir:** redesenho da parte "Entrada para termopar ou milivolts" da Figura 1.9 do manual:
> - os bornes 1, 2, 3 e 4;
> - o fio + (verde, IEC) no borne 2 e o fio − (branco) no borne 3;
> - os jumpers 1–4 e 3–4;
> - ao lado, uma foto ou captura dos bornes do TT301 com essa ligação feita.

**Figura 7. Cadeia de sinal do TT301 com termopar**

![Figura 7](assets/fig07_cadeia_sinal_tc.png)

> **Imagem a inserir:** diagrama de blocos com: entrada em mV, trim de leitura, somador da junta fria (com o sensor Pt100 da borneira entrando nele), linearização NIST do tipo K, faixa LRV/URV, saída e trim de corrente.
> - Marque onde agem o trim de leitura, o **Temperature Trim**, a faixa e o trim de corrente.
> - Mostre a saída digital HART saindo depois da linearização.

---

## 5. A planta no LasecSimul

O projeto [`forno_termopar_tt301.lsproj`](forno_termopar_tt301.lsproj) reproduz um forno elétrico tipo mufla, com resistências de aquecimento e um **termopar tipo K** num tubo de proteção cerâmico. O termopar é ligado por cabo de extensão aos bornes 2 (+) e 3 (−) do TT301, com os jumpers 1–4 e 3–4.

A malha 4–20 mA é a mesma das outras práticas: fonte de 24 V, resistor de 250 Ω, amperímetro, modem HART e osciloscópio. A planta vem **incorporada ao projeto**.

É preciso o LasecSimul **0.0.84 ou mais novo**: a entrada de termopar do TT301 e o componente **Termopar** foram introduzidos nessa versão.

**Figura 8. Projeto completo no LasecSimul**

![Figura 8](assets/fig08_projeto_completo.png)

> **Imagem a inserir:** captura do LasecSimul com `forno_termopar_tt301.lsproj` rodando (selo "Running"), com o TT301 já configurado para termopar K (seção 9). Devem aparecer:
> - o slider "Aquecimento (%)" e a planta do forno;
> - os dois fios do termopar (verde e branco) chegando aos bornes 2 e 3 do TT301, e os jumpers 1–4 e 3–4;
> - a fonte de 24 V, o resistor de 250 Ω, o amperímetro, o modem HART e o osciloscópio.
>
> Use o botão "ajustar tudo à tela".

### 5.1 O que a planta calcula

- **Balanço de energia:** `C·dT/dt = P − UA·(T − T_amb)`, com `P = P_max·u/100`.
- **Tubo de proteção:** `dT_s/dt = (T − T_s)/τ`. `T_s` é a temperatura da junta de medição.
- **FEM nos bornes:** `E = E_tipo(T_s + desvio) − E_tipo(T_referência)`, pelas funções de referência do NIST. A junta de referência é:
  - **com cabo de extensão:** os bornes do transmissor, à temperatura ambiente;
  - **com cabo de cobre:** o cabeçote do termopar.
- **Elétrica:** a FEM é uma fonte de tensão em série com a resistência dos dois fios. O TT301 mede a tensão nos bornes, como o aparelho real. Polaridade invertida e termopar rompido são situações elétricas de verdade.

### 5.2 Parâmetros (Propriedades exportadas)

| Parâmetro | Padrão |
|---|---|
| Potência máxima das resistências (100 %) | 3 kW |
| Capacidade térmica do forno com a carga | 1,5 kJ/K |
| Coeficiente de perdas para o ambiente UA | 2,5 W/K |
| Temperatura ambiente (onde fica o transmissor) | 25 °C |
| Constante de tempo do termopar (tubo de proteção) | 20 s |
| Tipo do termopar | 3 (K) — 0 B, 1 E, 2 J, 3 K, 4 N, 5 R, 6 S, 7 T |
| Resistência de cada fio (termopar + cabo) | 2,5 Ω |
| Desvio do termopar (erro do sensor) | 0 °C |
| Cabo do cabeçote ao transmissor | 0 = extensão/compensação (correto); 1 = cobre comum |
| Temperatura no cabeçote do termopar | 60 °C |
| Polaridade invertida nos bornes | Não |
| Termopar rompido | Não |

O TT301 também tem uma propriedade exportada, a **Temperatura da borneira**: é a temperatura lida pelo seu sensor de junta fria (25 °C). Com tudo correto, ela é igual à temperatura ambiente da planta. Se você mudar uma, mude a outra; se mudar só a do TT301, simula um **sensor de junta fria descalibrado**.

### 5.3 Valores esperados

- Constante de tempo térmica: `C/UA = 1500/2,5 = 600 s = 10 min`.
- Temperatura de regime: `T = T_amb + P/UA = 25 + 12·u` (°C, com `u` em %).

| Aquecimento | Potência | Temperatura de regime | FEM do tipo K nos bornes |
|---:|---:|---:|---:|
| 25 % | 0,75 kW | 325 °C | 12,25 mV |
| 50 % | 1,5 kW | 625 °C | 24,97 mV |
| 75 % | 2,25 kW | 925 °C | 37,32 mV |
| 100 % | 3,0 kW | 1225 °C | — (passa do limite do forno) |

Com 100 %, o forno vai de 25 °C a 500 °C em cerca de **5 min** e a 1000 °C em cerca de **18 min**. Não deixe a 100 % por muito tempo: o regime (1225 °C) passa do limite das resistências e do uso contínuo do termopar K.

**Figura 9. Detalhe da planta**

![Figura 9](assets/fig09_planta_forno.png)

> **Imagem a inserir:** captura ampliada só da planta, com aquecimento em 50 % há alguns minutos. Devem aparecer:
> - o forno: carcaça, refratário, câmara, resistências nas paredes e a carga;
> - o termopar TE-201 entrando pelo teto, com a junta de medição na ponta;
> - o cabo verde e branco, a lâmpada "Aquecendo" acesa, o mostrador TI-201 e a barra de temperatura;
> - os displays: temperatura do forno, temperatura no termopar, FEM nos bornes, junta de referência, potência, perdas e constante de tempo.

**Figura 10. Propriedades exportadas da planta**

![Figura 10](assets/fig10_propriedades_exportadas.png)

> **Imagem a inserir:** painel **Properties** com a planta selecionada, rolado até "Propriedades exportadas":
> - os 12 parâmetros da seção 5.2;
> - as listas "Tipo do termopar" e "Cabo do cabeçote ao transmissor" abertas.

---

## 6. Arquivos da prática

- [`forno_termopar_tt301.lsproj`](forno_termopar_tt301.lsproj): forno com o termopar K incorporado e o TT301 na malha 4–20 mA;
- [`bancada_termopar_tt301.lsproj`](bancada_termopar_tt301.lsproj): bancada de calibração. O componente **Termopar** da paleta (pasta **Sensores**, ao lado do RTD) faz o papel do calibrador em modo termopar, e um voltímetro mostra a FEM nos bornes;
- [`assets/`](assets): imagens deste roteiro.

---

## 7. Como executar

1. Instale ou atualize a extensão **LasecSimul** no VS Code (0.0.84 ou mais nova).
2. Abra esta pasta no VS Code e dê dois cliques em `forno_termopar_tt301.lsproj`.
3. Clique em **Run** (▶).
4. Configure o TT301 para termopar K pelo PACTware (seções 8 e 9).
5. Selecione o slider **"Aquecimento (%)"** e altere o **Valor do sinal** no painel Properties.
6. Acompanhe a temperatura do forno e a do termopar, a FEM, a corrente no amperímetro e o display do TT301.
7. Para mudar a instalação (cabo, polaridade, tipo, desvio, termopar rompido), selecione a planta e use **Propriedades exportadas**.

Observações importantes:

- o TT301 sai de fábrica configurado para **Pt100 a 2 fios**. Com um termopar nos bornes, ele tenta medir resistência: com o forno frio indica cerca de −204 °C e, assim que o forno aquece, acusa falha do sensor (burnout). Isso dura até você configurar o sensor como termopar K, como num transmissor real trocado de aplicação;
- a planta é lenta como uma real (constante de tempo de 10 min). Para testes rápidos, reduza a capacidade térmica: com 0,3 kJ/K, a constante de tempo cai para 2 min;
- o tubo de proteção atrasa o termopar em relação ao forno: espere estabilizar antes de comparar valores.

---

## 8. Conexão com o PACTware

1. Selecione o **Modem HART** e confira em Properties: **Ligação com o PC** = Serial (COM / CNC) e **Porta serial** = `CNCB0`.
2. No **com0com**, confira o par `CNCB0` ↔ `COM20`.
3. No PACTware, em **Ver → Catálogo de instrumentos**, atualize o catálogo e adicione **HART Communication**.
4. Configure o driver HART na porta **COM20** e desmarque **Multimaster and Burst mode support**.
5. Clique com o botão direito no driver → **Adicionar instrumento** → **TT301** (Smar).
6. Com a simulação rodando, conecte o TT301 (ícones verdes) e abra **OnLine Parameterize**.

O DTM do TT301 organiza as telas em **OnLine Parameterize → Blocks**:

- **Identification**;
- **Configuration**, com **Sensor** e **PID**;
- **Calibration**;
- **General**.

Há ainda **Observe** (monitoração) e **Diagnosis**.

**Figura 11. TT301 conectado no PACTware**

![Figura 11](assets/fig11_pactware_conectado.png)

> **Imagem a inserir:** tela do PACTware com:
> - a árvore do projeto (HART Communication em COM20 → TT301) com ícones verdes;
> - a tela **Identification** aberta à direita: Manufacturer SMAR, Device Type TT301, Tag, Descriptor e Message.

---

## 9. Configuração do TT301

Faça uma alteração por vez, aplique (**Write**) e confira no simulador.

### 9.1 Identificação

Em **Identification**:

- **Tag:** `TT-201` (até 8 caracteres);
- **Descriptor:** `TEMP FORNO` (até 16);
- **Message:** a data e o responsável (até 32).

### 9.2 Sensor e junta fria

Em **Configuration → Sensor**:

- **Sensor Type** = Thermocouple (TC);
- **Sensor** = K NBS;
- **Connection** = Two wires;
- **Cold Junction** = Enabled.

Ao aplicar, o TT301 carrega os limites do tipo K: a faixa vai para −200 a 1350 °C, com span mínimo de 60 °C. O display passa a mostrar a temperatura do forno.

**Figura 12. Configuração do sensor**

![Figura 12](assets/fig12_dtm_sensor_tc.png)

> **Imagem a inserir:** tela **Configuration → Sensor** do DTM antes de aplicar, com:
> - Sensor Type Thermocouple, Sensor K NBS e Connection Two wires selecionados;
> - o campo Cold Junction em Enabled.

### 9.3 Faixa, unidade e amortecimento

Em **Calibration**:

- **PV Unit** = degC;
- **LRV** = 0 e **URV** = 1000;
- **Damping** = 2 s.

Confira os limites somente de leitura: **LRL = −200 °C**, **URL = 1350 °C** e **Min Span = 60 °C** (termopar K).

Com a faixa de 0 a 1000 °C, 25 °C correspondem a **4,40 mA**, e 625 °C a **14,00 mA**.

**Figura 13. Faixa e amortecimento**

![Figura 13](assets/fig13_dtm_calibration.png)

> **Imagem a inserir:** tela **Calibration** do DTM com PV Unit degC, URV 1000, LRV 0, URL 1350, LRL −200, Min Span 60 e Damping 2. O quadro "To Applied Input" com os botões LO− e HI+ deve estar visível.

### 9.4 Burnout, proteção de escrita e display

Em **Configuration**:

- **Burnout** = **High** (21 mA): num forno, temperatura alta leva o controle a cortar o aquecimento;
- **W.Protect** = read/write durante a configuração;
- **LCD:** **First Variable = Temp. PV** e **Second Variable = PV%** (ou corrente).

**Figura 14. Display do TT301**

![Figura 14](assets/fig14_lcd_tt301.png)

> **Imagem a inserir:** duas capturas ampliadas do display do TT301 no simulador, com o forno estabilizado (por exemplo, a 625 °C com aquecimento em 50 %), montadas lado a lado:
> - uma com a temperatura em °C;
> - outra, 3 s depois, com a segunda variável.

---

## 10. Calibração

### 10.1 O que se calibra

Numa malha com termopar há três fontes de erro, tratadas separadamente:

- o **termopar**: tolerância da classe, deriva com o uso em alta temperatura e contaminação;
- a **instalação**: cabo de extensão, polaridade, emendas e junta de referência;
- o **transmissor**: medida de mV, sensor de junta fria e conversor de saída.

O transmissor é calibrado com um **calibrador de processo** no lugar do termopar. O termopar é calibrado num **forno de calibração**, comparado com um **termopar ou termômetro padrão**. Uma verificação da **malha completa** fecha o trabalho.

Diferente do transmissor, o termopar **não tem ajuste**. Se estiver fora da tolerância, é trocado. Se estiver dentro, mas se quer menos incerteza, aplica-se uma **correção**: no TT301, a função **Sensor Especial** aceita uma tabela de 16 pontos mV × temperatura.

### 10.2 Teste como encontrado

Antes de qualquer ajuste, registre o estado do instrumento:

1. Salve no PACTware a configuração como encontrada.
2. Anote tag, sensor, ligação, junta fria, faixa, amortecimento e burnout.
3. Faça o teste do transmissor (seção 10.3) sem mudar nada.

Critério de aceitação de exemplo: **±0,2 % do span** para o transmissor (±2 °C numa faixa de 0 a 1000 °C).

### 10.3 Calibração do transmissor com calibrador em modo termopar

É o método mais comum em campo. O calibrador **simula** um termopar: você digita a temperatura, e ele gera a FEM correspondente, já descontando a temperatura dos próprios bornes (compensação interna da junta de referência, **RJC ligada**).

1. Desligue o termopar dos bornes 2 e 3 (mantenha os jumpers 1–4 e 3–4).
2. Ligue o calibrador, em modo **TC tipo K, fonte (source)**, aos bornes 2 (+) e 3 (−) com **cabo ou miniconector do tipo K**. Com o fio do tipo K, a junta de referência continua nos bornes do TT301, e a compensação dele funciona como na instalação real.
3. Ligue o multímetro de precisão em série na malha (ou nos terminais TEST).
4. Espere o conjunto estabilizar termicamente: correntes de ar e a mão no borne mudam a junta fria.
5. Aplique os pontos da tabela, subindo e depois descendo, e espere estabilizar em cada um.
6. Anote a temperatura lida no PACTware (PV) e a corrente.

| Ponto | t aplicada (°C) | I esperada (mA) | PV lida ↑ | I lida ↑ | PV lida ↓ | I lida ↓ |
|---:|---:|---:|---:|---:|---:|---:|
| 0 % | 0 | 4,000 | | | | |
| 25 % | 250 | 8,000 | | | | |
| 50 % | 500 | 12,000 | | | | |
| 75 % | 750 | 16,000 | | | | |
| 100 % | 1000 | 20,000 | | | | |

Erro da leitura digital e da saída em % do span:

$$
e_{PV}(\%) = \frac{PV - t}{URV - LRV}\cdot 100
\qquad
e_{mA}(\%) = \frac{I - I_{esp}}{16}\cdot 100
$$

Atenção: se o calibrador for ligado com **fio de cobre** e a RJC estiver ligada, o calibrador desconta a temperatura dos **próprios** bornes, e não a dos bornes do TT301. Com os dois a temperaturas diferentes, aparece um erro igual a essa diferença.

**Figura 15. Calibrador em modo termopar nos bornes do TT301**

![Figura 15](assets/fig15_calibrador_modo_tc.jpg)

> **Imagem a inserir:** foto da bancada com:
> - o calibrador de processo (por exemplo, Fluke 725 ou Presys) em modo TC tipo K, fonte, mostrando a temperatura simulada (por exemplo, 500 °C);
> - o cabo com miniconector tipo K ligado aos bornes 2 e 3 do TT301, com os jumpers 1–4 e 3–4 visíveis;
> - o multímetro em série na malha;
> - o PACTware no notebook mostrando a PV.

### 10.4 Calibração do transmissor com fonte de mV

Quando o calibrador só tem modo **mV**, ou quando se quer evitar a RJC dele, a ligação é feita com **fio de cobre**, e a FEM de cada ponto é calculada à mão:

$$
E_{aplicar} = E(t) - E(T_{borneira})
$$

1. Meça a temperatura da borneira do TT301 com um termômetro de contato, ou leia a **variável secundária (SV)** do TT301 no PACTware.
2. Calcule a tensão de cada ponto com a tabela do tipo K.
3. Aplique as tensões e anote PV e corrente, como na seção 10.3.

Com a borneira a **25 °C** (E = 1,000 mV):

| Ponto | t (°C) | E(t) (mV) | E aplicar (mV) | I esperada (mA) |
|---:|---:|---:|---:|---:|
| 0 % | 0 | 0,000 | −1,000 | 4,000 |
| 25 % | 250 | 10,153 | 9,153 | 8,000 |
| 50 % | 500 | 20,644 | 19,644 | 12,000 |
| 75 % | 750 | 31,213 | 30,213 | 16,000 |
| 100 % | 1000 | 41,276 | 40,275 | 20,000 |

O ponto de 0 °C exige **tensão negativa**. Confira se o calibrador gera mV negativos.

O erro clássico é aplicar o valor da tabela (`E(t)`) sem descontar a borneira: o TT301 soma a junta fria e lê cerca de **25 °C a mais**. Com 20,644 mV, lê 523,5 °C em vez de 500 °C.

### 10.5 Verificação da junta fria

1. Ligue os bornes 2 e 3 com um **fio de cobre curto** (0 mV).
2. Com a junta fria habilitada, o TT301 deve indicar a **temperatura da borneira**.
3. Compare com um termômetro padrão encostado nos bornes.
4. Se a diferença passar do critério, faça o **Temperature Trim** (trim do sensor de temperatura da borneira) informando a temperatura do padrão.

Faça a verificação da junta fria **depois** da calibração de mV: a junta fria só pode ser avaliada com a medida de mV já correta.

### 10.6 Trim de leitura (zero e ganho)

Se a medida de mV não confere com o padrão:

1. Aplique o valor do **ponto inferior** com a fonte de mV e faça o **Zero Trim** informando o valor aplicado.
2. Aplique o valor do **ponto superior** (por exemplo, 40,275 mV) e faça o **Gain Trim**.
3. Confira de novo os cinco pontos.
4. Se algo der errado, o **Factory Trim** restaura o ajuste de fábrica.

O manual pede um padrão de mV com **exatidão melhor que 0,02 %**.

### 10.7 Trim de corrente

Se a PV confere mas a corrente não:

1. Coloque a malha de controle em **manual**: durante o trim, a corrente não representa o processo.
2. Em **Trim → Current Trim**, o TT301 coloca a saída em 4 mA. Leia o multímetro e informe o valor lido.
3. Repita para 20 mA.

### 10.8 Calibração com referência (LO− / HI+)

No quadro **To Applied Input** da tela **Calibration**:

- com a entrada no valor que deve ser 4 mA, clique em **LO−**: o LRV passa a ser o valor medido, e o span é mantido;
- com a entrada no valor de 20 mA, clique em **HI+**: o URV passa a ser o valor medido.

Isso **não** corrige o termopar nem a junta fria: só muda a faixa.

### 10.9 Calibração do termopar (sensor)

1. Retire o termopar do forno e coloque-o no **forno de calibração** (bloco seco de alta temperatura ou forno tubular), junto com o **padrão**: termopar tipo S ou R calibrado ou, até cerca de 600 °C, um termômetro de resistência de platina padrão.
2. As duas pontas devem ficar **na mesma profundidade** e bem imersas: no mínimo 15 vezes o diâmetro do sensor.
3. Em cada ponto (por exemplo, 300, 500 e 700 °C), espere a estabilização e anote a temperatura do padrão e a do termopar. Meça o termopar com um indicador ou calibrador de boa exatidão, ou com o próprio TT301 já calibrado.
4. Calcule o erro e compare com a tolerância da classe (seção 2.5). Por exemplo, a 500 °C: ±2,0 °C (classe 1) e ±3,75 °C (classe 2).
5. Fora da tolerância: substitua o termopar. Dentro, mas com erro conhecido: registre a correção e, se for o caso, grave-a no TT301 como **Sensor Especial**.

O termopar de metal base sofre deriva ao longo do comprimento que ficou no calor. Calibrado fora do processo, ele pode não reproduzir exatamente o erro que tinha instalado. Por isso, em aplicações críticas, faz-se também a **comparação no próprio processo** com um termopar padrão inserido ao lado.

**Figura 16. Calibração do termopar em forno de calibração**

![Figura 16](assets/fig16_forno_calibracao.jpg)

> **Imagem a inserir:** foto do forno de calibração (bloco seco de alta temperatura) com o termopar padrão e o termopar em teste inseridos lado a lado no mesmo inserto, na mesma profundidade, e o indicador do padrão mostrando a temperatura do ponto.

### 10.10 Teste de burnout e teste de malha

1. **Burnout:** desligue um fio do termopar (ou use "Termopar rompido" na planta). A saída deve ir a 21 mA (High) ou 3,6 mA (Low), e o display deve indicar a falha.
2. **Teste de malha:** force 4, 12 e 20 mA pelo DTM e confira a indicação no sistema de controle.

### 10.11 Teste como deixado e registro

Repita o teste da seção 10.3 na faixa final e preencha o registro com:

- identificação (tag, tipo de termopar, junta fria, faixa);
- padrões usados (calibrador, multímetro, termômetro da borneira) e validade;
- temperatura da borneira durante o ensaio;
- tabelas como encontrado e como deixado;
- ajustes feitos;
- resultado, responsável e data.

**Figura 17. Gráfico de erro como encontrado e como deixado**

![Figura 17](assets/fig17_grafico_erro.png)

> **Imagem a inserir:** gráfico com os pontos de 0 a 100 % no eixo horizontal e o erro em °C (ou % do span) no vertical, com:
> - as séries subida e descida, como encontrado e como deixado;
> - as linhas de tolerância (±0,2 % do span = ±2 °C).

---

## 11. Bancada virtual no LasecSimul

Os dois projetos permitem ensaiar toda a prática:

| Situação real | Como reproduzir no LasecSimul |
|---|---|
| Calibrador em modo TC (simulação de termopar) | `bancada_termopar_tt301.lsproj`: no componente **Termopar**, "Junta de medição" = temperatura do ponto (ou gire o dial). O multímetro mostra a FEM nos bornes |
| Fonte de mV com o valor calculado | Termopar com "Junta de referência" = temperatura da borneira (25 °C): ele gera exatamente `E(t) − E(25)` |
| Fonte de mV com o valor da tabela, sem descontar a borneira | Termopar com "Junta de referência" = 0 °C: a leitura fica cerca de 25 °C acima |
| Junta fria desabilitada | PACTware: **Cold Junction = Disabled** |
| Sensor de junta fria descalibrado | TT301 → Propriedades exportadas → "Temperatura da borneira" = 30 °C, com o termopar ainda a 25 °C |
| Cabo de cobre a partir do cabeçote | Forno: "Cabo do cabeçote ao transmissor" = Cobre e "Temperatura no cabeçote" = 60 °C |
| Polaridade invertida | Forno: "Polaridade invertida" = Sim (ou troque os fios nos bornes 2 e 3) |
| Termopar de tipo diferente do configurado | Forno: "Tipo do termopar" = 2 (J), com o TT301 em K |
| Termopar com deriva | Forno: "Desvio do termopar" (por exemplo, +3 °C) |
| Termopar rompido | Forno: "Termopar rompido" = Sim; na bancada, "Rompido" |
| Forno de calibração | Forno com aquecimento constante até estabilizar; a "Temperatura do forno" faz o papel do padrão |

A eletrônica simulada do TT301 é ideal. O simulador mostra os erros do sensor, da instalação e da configuração (tipo, junta fria, faixa). Os trims (Zero, Gain, Current e Temperature Trim) seguem os passos do DTM e devem ser praticados no TT301 real do laboratório.

**Figura 18. Bancada virtual de calibração**

![Figura 18](assets/fig18_bancada_virtual.png)

> **Imagem a inserir:** captura de `bancada_termopar_tt301.lsproj` rodando, com o TT301 configurado (termopar K, 0 a 1000 °C). Devem aparecer:
> - o componente Termopar com o dial em 500 °C;
> - o multímetro mostrando 19,64 mV;
> - o display do TT301 em 500,0 °C e o amperímetro em 12,00 mA;
> - à direita, o painel Properties do Termopar (junta de medição, junta de referência, tipo K, resistência dos fios e "Rompido").

**Figura 19. Erros de instalação no simulador**

![Figura 19](assets/fig19_erros_instalacao.png)

> **Imagem a inserir:** montagem com três capturas do forno estabilizado em 625 °C (aquecimento em 50 %). Em cada uma, o display "Temperatura no termopar" da planta e o display do TT301:
> 1. cabo de cobre com o cabeçote a 60 °C (TT301 em 591,2 °C);
> 2. junta fria desabilitada (TT301 em 601,4 °C);
> 3. tipo J configurado com termopar K (TT301 em 479,4 °C).

---

## 12. Sugestão de atividade experimental

### Parte A: comissionamento

1. Conecte o PACTware e salve a configuração como encontrada. Anote a leitura e a corrente com a configuração de fábrica (Pt100), com o forno frio e depois aquecendo, e explique o que o TT301 está medindo.
2. Configure o TT301 como na seção 9 (TT-201, termopar K, junta fria habilitada, 0 a 1000 °C, damping 2 s, burnout High).
3. Com aquecimento em 0 %, confira 25 °C e 4,40 mA.

### Parte B: compensação da junta fria (bancada)

1. Abra `bancada_termopar_tt301.lsproj` e configure o TT301 como na Parte A.
2. Ajuste o termopar para 500 °C. Anote a FEM no multímetro, a PV e a corrente.
3. Desabilite a junta fria no PACTware e anote a PV. Habilite de novo.
4. Mude a "Junta de referência" do termopar para 0 °C e anote a PV. Volte para 25 °C.
5. Mude só a "Temperatura da borneira" do TT301 para 30 °C e anote a PV. Volte para 25 °C.

| Situação | FEM (mV) | PV (°C) | Erro (°C) | Valor esperado (°C) |
|---|---:|---:|---:|---:|
| Normal | | | | 500,0 |
| Junta fria desabilitada | | | | 476,5 |
| Fonte de mV com o valor da tabela | | | | 523,5 |
| Sensor da borneira lendo 30 °C | | | | 504,7 |

### Parte C: calibração do transmissor

1. Na bancada, faça o teste de cinco pontos de ida e volta da seção 10.3.
2. Calcule os erros e verifique o critério de ±0,2 % do span.
3. Repita com a "Junta de referência" igual à borneira, como uma fonte de mV (seção 10.4), e confira no multímetro as tensões da tabela.

### Parte D: erros de instalação (forno)

1. No forno, reduza a capacidade térmica para 0,3 kJ/K e ajuste o aquecimento para 50 %. Espere estabilizar em 625 °C.
2. Registre a leitura do TT301 em cada situação, voltando ao normal entre elas.

| Situação | Como provocar | Leitura do TT301 (°C) | Erro (°C) | Esperado (°C) |
|---|---|---:|---:|---:|
| Normal | — | | | 625,0 |
| Cabo de cobre | Cabo = Cobre, cabeçote a 60 °C | | | 591,2 |
| Junta fria desabilitada | PACTware | | | 601,4 |
| Tipo trocado | Tipo do termopar = J | | | 479,4 |
| Polaridade invertida | Polaridade = Sim | | | −200 (fim da faixa) |

### Parte E: termopar com deriva e classes de tolerância

1. Ajuste o "Desvio do termopar" para **+3 °C** e compare a leitura do TT301 com a temperatura do forno estabilizada (que faz o papel do padrão).
2. A 625 °C, o erro está dentro da classe 1 (±2,5 °C)? E da classe 2 (±4,7 °C)?
3. Discuta: um trim do transmissor deve ser usado para esconder o erro do termopar?

### Parte F: tempo de resposta do tubo de proteção

1. Com o forno estabilizado, aplique um degrau de 50 % para 75 % no aquecimento.
2. Registre a temperatura do forno e a do termopar a cada 10 s durante 2 min.
3. Estime a constante de tempo do tubo (tempo para o termopar atingir 63 % da diferença) e compare com os 20 s configurados.

### Parte G: burnout e malha

1. Ative "Termopar rompido" e registre a corrente e o display.
2. Mude o burnout para Low e repita.
3. Faça o teste de malha em 4, 12 e 20 mA.

### Questões para o relatório

1. Por que o termopar mede uma **diferença** de temperatura? Qual lei dos circuitos termoelétricos permite compensar a junta de referência?
2. Qual a diferença entre cabo de extensão e cabo de compensação? Por que o cabo de cobre causa erro?
3. Por que o erro de junta fria desabilitada é aproximadamente igual à temperatura ambiente?
4. Ao calibrar com fonte de mV e fios de cobre, por que é preciso medir a temperatura da borneira?
5. Por que o erro do termopar costuma ser maior que o do transmissor? Como isso aparece nas classes de tolerância?
6. Num forno, por que o burnout High costuma ser a escolha segura?
7. Compare termopar K e Pt100 (Aula 08): faixa, exatidão, robustez, custo e cuidados de instalação.

Essa atividade conecta a física do sensor, a instalação elétrica e a configuração e calibração de um transmissor HART.

---

## 13. Conclusão

A prática mostra, de forma aplicada, a cadeia completa de uma medição de temperatura com termopar:

- o aquecimento muda a temperatura do forno, e o tubo de proteção a transmite ao termopar com atraso;
- o termopar gera a FEM de Seebeck da diferença entre a junta de medição e a junta de referência;
- o cabo de extensão leva a junta de referência até os bornes do transmissor, onde ela é medida;
- o TT301 soma a compensação da junta fria, lineariza pela tabela NIST do tipo e transmite a temperatura em 4–20 mA com HART;
- a calibração separa o erro do termopar, o da instalação e o do transmissor, e cada ajuste é feito no lugar certo.

Com isso, o roteiro serve tanto como material de estudo quanto como apoio para a aula prática de instrumentação.

---

## 14. Referências

- Apostila de Instrumentação Industrial II - UFU
- Smar. *TT301 — Manual de Instrução, Operação e Manutenção*:
  - seção 1: instalação e Figura 1.9, ligação do sensor;
  - seção 2: termopares e compensação de junta fria;
  - seção 3: configuração do sensor, junta fria, Trim e Temperature Trim;
  - seção 6: especificações e Tabela 6.1.
- IEC 60584-1. *Thermocouples — Part 1: EMF specifications and tolerances.*
- IEC 60584-3. *Thermocouples — Part 3: Extension and compensating cables — Tolerances and identification system.*
- ABNT NBR 12771. *Termopares — Tabelas de referência.*
- ABNT NBR 13774. *Cabos e fios de compensação ou extensão para termopar — Tolerâncias e identificação.*
- ABNT NBR 13863. *Preparação e uso de junção de referência para calibração de termopar.*
- EURAMET cg-8. *Calibration of Thermocouples.*
- NIST. [ITS-90 Thermocouple Database (SRD 60, Monograph 175)](https://its90.nist.gov/) e [tabelas e coeficientes para download](https://its90.nist.gov/ThermoDownloads).
- LAPP. [Cabos de extensão e compensação: código de cores e informações (Tabela T8)](https://t3.lappcdn.com/fileadmin/catalog/pt_T_pdf/T8_Cabos_de_extensão_e_compensação__código_de_cores_e_informações_para_conhecimento_br.pdf).
- TC Direct. [Do I need thermocouple extension or compensating cable?](https://www.tcdirect.co.uk/thermocouple-cable/thermocouple-extension-compensating.aspx)
- ICS Schneider. [Temperature measurement according to DIN EN 60584](https://www.ics-schneider.de/temperature-measurement-according-to-din-en-60584-what-needs-to-be-considered/?lang=en).
- Beamex. [Thermocouple Cold (Reference) Junction Compensation](https://resources.beamex.com/thermocouple-cold-junction-compensation).
- Fluke. [Thermocouple voltage to temperature calculator](https://www.fluke.com/en-au/learn/tools-calculators/thermocouple-voltage-to-temperature-calculator).
- AutomationForum. [How to calibrate a thermocouple transmitter](https://automationforum.co/?p=314).
- [LasecSimul — LASEC/UFU](https://github.com/LASEC-UFU/LasecSimul)

---

## 15. Apêndice: lista de imagens

Todas as imagens ficam na pasta [`assets/`](assets), com os nomes exatos da tabela. Cada figura do texto já traz, no quadro "Imagem a inserir", o que ela precisa mostrar. Este apêndice reúne como produzir cada uma.

### 15.1 Como produzir as imagens

- **Capturas do LasecSimul:** VS Code maximizado (1920 × 1080), barra lateral fechada (`Ctrl+B`), escala do Windows em 100 %, simulação rodando e valores estabilizados. Use **Ajustar tudo** ou **Ajustar seleção** e a Ferramenta de Captura (`Win+Shift+S`). Salve em PNG.
- **Capturas do PACTware:** janela maximizada, TT301 conectado (ícones verdes). Capture antes de aplicar quando a figura mostra valores digitados.
- **Fotos:** luz difusa, sem flash direto nos displays, câmera paralela ao instrumento, valores legíveis. Salve em JPG.
- **Desenhos e gráficos:** diagrams.net (draw.io), Inkscape, PowerPoint, planilha ou Python, fundo branco, PNG com cerca de 1600 px de largura.
- **Anotações:** setas e números em vermelho, explicados na legenda. Nos desenhos de ligação do tipo K, use as cores da IEC 60584-3: fio **+ verde** e fio **− branco**.

### 15.2 Figuras

| Figura | Arquivo | Tipo | Como obter | O que destacar |
|---:|---|---|---|---|
| 1 | `fig01_termopar_industrial.jpg` | Foto | Foto de um termopar industrial do laboratório com o cabeçote aberto. | Tubo de proteção, cabeçote, fios + e − e o cabo de extensão. |
| 2 | `fig02_curvas_fem_tipos.png` | Gráfico | Python (`scripts/thermocouple_its90.py`, função `emf`) ou planilha com as tabelas do NIST, de 0 a 1800 °C. | O tipo K em destaque e os pontos da tabela da seção 2.4. |
| 3 | `fig03_circuito_termopar.png` | Desenho | Desenho do circuito com as duas juntas e o voltímetro. | `T`, `T_r` e a equação `E = E(T) − E(T_r)`. |
| 4 | `fig04_cores_cabos.png` | Foto ou desenho | Pedaços de cabo de extensão do laboratório, ou desenho pela tabela da seção 3.3. | Negativo branco (IEC) e vermelho (ANSI). |
| 5 | `fig05_cabo_extensao_x_cobre.png` | Desenho | Duas instalações, uma sobre a outra. | Onde fica a junta de referência em cada caso. |
| 6 | `fig06_tt301_ligacao_termopar.png` | Desenho + foto | Redesenhe a parte de termopar da Figura 1.9 do manual e fotografe os bornes ligados. | + no 2, − no 3 e os jumpers 1–4 e 3–4. |
| 7 | `fig07_cadeia_sinal_tc.png` | Desenho | Diagrama de blocos a partir da Figura 2.2 do manual. | Junta fria, linearização, trim de leitura, Temperature Trim e trim de corrente. |
| 8 | `fig08_projeto_completo.png` | Captura LasecSimul | Abra `forno_termopar_tt301.lsproj`, configure o TT301 (seção 9), clique em **Run**, aquecimento em 50 % e **Ajustar tudo**. | Slider, forno, cabo verde/branco, jumpers, fonte, resistor, amperímetro, modem e osciloscópio. |
| 9 | `fig09_planta_forno.png` | Captura LasecSimul | Com 50 % de aquecimento por alguns minutos, selecione a planta, **Ajustar seleção** e recolha o painel Properties. | Termopar TE-201, lâmpada "Aquecendo", TI-201, barra e displays (com a FEM em mV). |
| 10 | `fig10_propriedades_exportadas.png` | Captura LasecSimul | Selecione a planta e abra **Properties → Propriedades exportadas**. | Os 12 parâmetros, com as listas de tipo e de cabo abertas. |
| 11 | `fig11_pactware_conectado.png` | Captura PACTware | Conecte o TT301 e abra **Identification**. | Ícones verdes, SMAR, TT301, Tag, Descriptor e Message. |
| 12 | `fig12_dtm_sensor_tc.png` | Captura PACTware | **Configuration → Sensor** com os valores escolhidos, antes de aplicar. | Thermocouple, K NBS, Two wires e Cold Junction Enabled. |
| 13 | `fig13_dtm_calibration.png` | Captura PACTware | Tela **Calibration** com a faixa digitada. | degC, LRV 0, URV 1000, Damping 2, LRL/URL, Min Span 60 e os botões LO−/HI+. |
| 14 | `fig14_lcd_tt301.png` | Captura LasecSimul (montagem) | Forno estabilizado; aproxime o display do TT301 (zoom de cerca de 300 %) e capture as duas páginas com 3 s de intervalo. | Temperatura em °C e a segunda variável. |
| 15 | `fig15_calibrador_modo_tc.jpg` | Foto | Foto da bancada com o calibrador em modo TC tipo K nos bornes 2 e 3 e o multímetro na malha. | Temperatura simulada legível, miniconector tipo K e a PV no notebook. |
| 16 | `fig16_forno_calibracao.jpg` | Foto | Foto do forno de calibração com o padrão e o termopar no mesmo inserto. | Mesma profundidade de imersão e a temperatura do padrão no indicador. |
| 17 | `fig17_grafico_erro.png` | Gráfico | Planilha ou Python com as tabelas da seção 10.3 (como encontrado e como deixado). | Quatro séries e as linhas de tolerância de ±0,2 % do span. |
| 18 | `fig18_bancada_virtual.png` | Captura LasecSimul | Abra `bancada_termopar_tt301.lsproj`, configure o TT301, clique em **Run**, ponha o termopar em 500 °C e abra o painel Properties dele. | 19,64 mV no multímetro, 500,0 °C no TT301 e 12,00 mA. |
| 19 | `fig19_erros_instalacao.png` | Captura LasecSimul (montagem) | Na Parte D, com o forno estabilizado em 625 °C, capture a planta e o TT301 em cada situação. | Cabo de cobre, junta fria desabilitada e tipo trocado, com as leituras lado a lado. |
