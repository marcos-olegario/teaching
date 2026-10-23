# How to use CONEX

> **Note (English):** This is an introductory study guide for the air shower simulation code **CONEX**, written in **Portuguese** for undergraduate students who are starting to work with extensive air shower (EAS) simulations. It covers installation, running simple simulations, and reading the ROOT output files. It does not replace the official documentation distributed with the code (the `README`/documentation file inside the CONEX package). When in doubt, the official documentation is the reference.

---

## Sumário

1. [O que é o CONEX](#1-o-que-é-o-conex)
2. [Instalação](#2-instalação)
3. [Executando simulações simples](#3-executando-simulações-simples)
4. [Entendendo os arquivos de saída](#4-entendendo-os-arquivos-de-saída)
5. [Referências](#5-referências)

---

## 1. O que é o CONEX

O CONEX é um código **híbrido e unidimensional** para simulação de chuveiros atmosféricos extensos. Ele combina:

- **Monte Carlo (MC)** para as interações de mais alta energia (onde as flutuações importam);
- **solução numérica de equações de cascata (CE)** para as partículas secundárias de mais baixa energia (o que deixa a simulação muito rápida).

Para uma dada massa, energia e ângulo zenital do primário, ele calcula os **perfis longitudinais** (número de partículas carregadas, múons, elétrons, fótons, hádrons e depósito de energia) e faz um **ajuste de Gaisser-Hillas** em cada chuveiro. Por ser unidimensional, o CONEX **não** fornece distribuições laterais nem partículas no solo como o CORSIKA — ele é ideal para estudar grandezas como $X_\text{max}$, $N_\text{max}$ e o número de múons em função da profundidade.

Esta versão do tutorial foi escrita com base no **CONEX v7.80** (saída versão 2.5x).

---

## 2. Instalação

A instalação tem três etapas: (1) instalar o **ROOT**, (2) obter os dados do **CORSIKA (`corsika-data`)** e (3) compilar o **CONEX**.

Para instalar e executar o CONEX, é necessário que você tenha um pouco de familiaridade com o **terminal** do Linux e os **principais comandos** (como baixar e atualizar arquivos/programas, criar pastas e navegar entre diretórios).

### 2.1 Dependências do sistema

O CONEX é escrito em Fortran e C++, então precisamos de compiladores e de algumas ferramentas. No Ubuntu:

```bash
sudo apt update
sudo apt install build-essential gfortran make git bzip2 libboost-iostreams-dev
```

### 2.2 Instalando o ROOT

Há dois caminhos. **Recomendo o Conda** para quem está começando, porque isola tudo em um ambiente.

O **Conda** é um gerenciador de programas que cria "ambientes": caixinhas isoladas onde instalamos programas sem bagunçar o resto do sistema. Vamos usá-lo para instalar o **ROOT**, o software do CERN que o CONEX usa para gravar os resultados.
#### Opção A — via Conda (recomendado)

Acesse o site https://www.anaconda.com/download para entender como instalar.

1. Instale o Anaconda (ou o Miniforge/Miniconda, que são mais leves):

   ```bash
   wget https://repo.anaconda.com/archive/Anaconda3-2023.07-1-Linux-x86_64.sh
   bash Anaconda3-2023.07-1-Linux-x86_64.sh
   ```

2. Se o instalador não configurar o `PATH` sozinho, adicione ao final do `~/.bashrc`:

   ```bash
   export PATH="/home/SEU_USUARIO/anaconda3/bin:$PATH"
   ```

   e recarregue com `source ~/.bashrc`.

#### 2.2.1 Criando um ambiente com o ROOT

 1) Crie um ambiente chamado (por exemplo) `rootenv` com o ROOT dentro:

```bash
conda create -n rootenv root -c conda-forge
```

Ele vai listar muitos pacotes e perguntar `Proceed ([y]/n)?`. Digite `y` e `Enter`. Isso pode demorar alguns minutos.

2) Ative o ambiente:

```bash
conda activate rootenv
```

O prompt muda de `(base)` para `(rootenv)`.

3) Teste o ROOT:

```bash
root --version
```

Deve aparecer a versão do ROOT (por exemplo, `ROOT Version: 6.xx/xx`).

#### Opção B — binário pré-compilado

Também é possível instalar o ROOT a partir de um arquivo pré-compilado (*binário*). Esse caminho exige escolher a versão certa para o seu Ubuntu e compilador, por isso não o seguimos aqui. Quem quiser pode seguir as instruções do site oficial: <https://root.cern/install/>. 

Esse caminho pode ser útil para usos do CONEX em clusters, por exemplo.

### 2.3 Obtendo o código do CONEX

Agora que temos um ambiente onde usar o CONEX, precisamos do programa.
Como este é um programa científico "fechado", você precisa que alguem te envie ele, seja seu orientador, colaboração em que faz parte ou através do CORSIKA (https://www.iap.kit.edu/corsika/).

Uma vez com o programa (uma pasta compactada,  algo como `conex_XXX.tar.gz`) em mãos, descompacte o pacote do CONEX em uma pasta de trabalho, por exemplo `~/conex`:

```bash
mkdir -p ~/conex
tar -xzvf conex_<versao>.tar.gz -C ~/conex
cd ~/conex
```

### 2.4 Dados do CORSIKA (`CORSIKA_DATA`)

Alguns modelos de interação hadrônica (QGSJET-II e QGSJET-III) precisam de tabelas grandes que não vêm com o CONEX. Elas ficam em um repositório chamado `corsika-data` (seu orientador ou grupo podem ter outro repositório de onde baixar essa pasta):

<https://gitlab.iap.kit.edu/AirShowerPhysics/corsika-data>

Baixe esse repositório (é grande; siga as instruções do próprio repositório) e anote o caminho — ele será a variável `CORSIKA_DATA`.

### 2.5 Configurar as variáveis de ambiente

O CONEX usa quatro variáveis:

| Variável | O que indica |
|---|---|
| `CONEX_ROOT` | pasta com os arquivos de configuração do CONEX |
| `ROOT_OUT` | pasta onde os resultados serão salvos |
| `URQMD13_TAB` | arquivo de tabela do modelo UrQMD (criado automaticamente na primeira vez) |
| `CORSIKA_DATA` | pasta do `corsika-data` |
Vamos guardar essas variáveis **dentro do ambiente `rootenv`**, assim elas são ligadas sozinhas quando você der `conda activate rootenv`.

1) Confirme que o ambiente está ativo (o prompt deve começar com `(rootenv)`). Se não estiver:

```bash
conda activate rootenv
```

2) Crie a pasta onde os resultados serão salvos:

```bash
mkdir -p ~/conex/run
```

3) Registre as variáveis (copie o bloco inteiro de uma vez; ele é um único comando):

```bash
conda env config vars set \
  CONEX_ROOT=$HOME/conex/cfg \
  ROOT_OUT=$HOME/conex/run \
  URQMD13_TAB=$HOME/conex/tables.dat \
  CORSIKA_DATA=$HOME/corsika-data
```

``$HOME`` é a pasta onde o CONEX foi descompactado.

4) Desative e ative de novo o ambiente para as variáveis valerem:

```bash
conda deactivate
conda activate conex
```

5) Confira:

```bash
conda env config vars list
```

Devem aparecer as quatro variáveis com caminhos começando por `/home/SEU_USUARIO/`.

### 2.6 Compilando o CONEX

"Compilar" é transformar o código-fonte (texto) em um programa que o computador consegue executar.

Dentro da pasta do CONEX, com o ambiente ativado:

```bash
make all            # compila as bibliotecas de todos os modelos
```

Vão aparecer **muitas** linhas de texto, possivelmente com vários `warning` (avisos). Isso é normal. Pode levar alguns minutos.

Também é possível compilar só um modelo com `make qgsjet`, `make epos`, `make qgsjetII` ou `make sibyll`. Ao final, o executável fica em `bin/` (nesta versão, `bin/conex2r`).

### ==Teste:==

1) Peça a ajuda do programa:

```bash
bin/conex2r -h
```

Deve aparecer a lista de opções do CONEX.

2) Rode um chuveiro de teste (1 próton de $10^{17}$ eV, vertical, modelo SIBYLL):

```bash
bin/conex2r -p 100 -e 17 -E 17 -z 0 -Z 0 -m 5 -n 1 -x teste
```

3) Veja se o arquivo de resultado apareceu:

```bash
ls ~/conex/run
```

Deve existir um arquivo terminado em `.root`, algo como `teste_sibyll_XXXXXXXXXX_100.root`.

**Se o arquivo apareceu, a instalação está completa!** O que significam essas opções e como ler o arquivo `.root` são os assuntos das próximas partes do guia. 🎉 🙌  🎆 🎊

### 2.7 Organização dos diretórios

Depois de compilado, a pasta do CONEX fica, de forma simplificada, assim:

```
conex/
├── bin/        # executável conex2r
├── cfg/        # arquivos de configuração (conex_sibyll.param, conex_epos.param, ...)
├── lib/        # bibliotecas compiladas (uma por modelo hadrônico)
├── macros/     # macros de exemplo para ler a saída (ex.: plotProfile.C)
├── src/        # código-fonte (inclui conexConfig.h)
├── tabs/       # tabelas dos modelos (EPOS, SIBYLL, DPMJET, QGSJET, UrQMD)
├── Makefile
└── run/        # (criada por você durante o processo) para guardar as saídas
```

### 2.8 Toda vez que for usar o CONEX

A instalação só é feita uma vez. Depois, sempre que abrir um terminal novo para usar o CONEX, digite:

```bash
conda activate rootenv
cd ~/conex
```

Se esquecer o `conda activate rootenv`, o CONEX não vai encontrar o ROOT nem as variáveis de ambiente.

---

## 3. Executando simulações simples

### 3.1 Primeiro exemplo

Diferente do CORSIKA, que lê um arquivo de entrada (*steering card*), o CONEX recebe as configurações principais diretamente por **flags** na linha de comando. Exemplo:

```bash
mkdir -p run
bin/conex2r -p 100 -z 0 -Z 0 -e 19 -E 19 -m 5 -n 100 -x simulation
```

O que cada flag faz:

| Flag            | Valor        | Significado                                      |
| --------------- | ------------ | ------------------------------------------------ |
| `-p 100`        | próton       | partícula primária (código = A×100 para núcleos) |
| `-z 0 -Z 0`     | 0°           | intervalo do ângulo zenital → chuveiro vertical  |
| `-e 19 -E 19`   | $10^{19}$ eV | intervalo de energia em $\log_{10}(E/\text{eV})$ |
| `-m 5`          | SIBYLL 2.3e  | modelo hadrônico de altas energias               |
| `-n 100`        | 100          | número de chuveiros                              |
| `-x simulation` | —            | prefixo do arquivo de saída                      |

### 3.2 Lista de flags

| Flag | Descrição | Padrão | Observações |
|---|---|---|---|
| `-p` | partícula primária | `100` | `100` = próton, `5600` = ferro, `0` = fóton (A×100); também aceita códigos PDG para partículas "padrão" (ex.: `11` elétron, `22` fóton). Neutrinos e taus **não** são suportados. |
| `-e` | $\log_{10}(E_\text{min}/\text{eV})$ | `16.5` | |
| `-E` | $\log_{10}(E_\text{max}/\text{eV})$ | `21` | se `-e` = `-E`, a energia é fixa |
| `-a` | índice espectral $\alpha$ ($dN/dE \propto E^{-\alpha}$) | `3` | só importa se $E_\text{min} \neq E_\text{max}$ (ver abaixo) |
| `-z` | ângulo zenital mínimo (graus) | `60` | |
| `-Z` | ângulo zenital máximo (graus) | `60` | > 90° = chuveiro ascendente |
| `-i` | ângulo azimutal (graus) | `0` | convenção Auger: 0 = Leste; < 0 sorteia uniforme em [0, 360] |
| `-m` | modelo de altas energias | `4` | ver tabela abaixo |
| `-n` | número de chuveiros | `1` | |
| `-s` | semente aleatória | `0` | ≤ 0: gerada automaticamente; > 0: valor fixo |
| `-S` | auto-salvar a cada n chuveiros | `10` | |
| `-x` | prefixo do arquivo de saída | `conex` | |
| `-o` / `-O` | parâmetro de impacto mín./máx. [m] | `0` | |
| `-H` | `1` = sem cascata eletromagnética | `0` | |
| `-K` | nível de detalhe da árvore `LeadingInteractions` | `0` | número de interações salvas (< 100) |

**Modelos hadrônicos (`-m`):**

| Código | Modelo |
|---|---|
| 2 | QGSJET01 |
| 4 | EPOS LHC-R (padrão) |
| 5 | SIBYLL 2.3e |
| 6 | QGSJETII-04 |
| 9 | DPMJETIII.2017-1 |
| 10 | QGSJETIII-01 |

O modelo de **baixas energias** padrão é o UrQMD 1.3. A transição entre baixa e alta energia é em 80 GeV (30 GeV para o EPOS LHC-R).

#### Detalhes que costumam confundir

- **Espectro de energia (`-a`)**: se `a > 1`, as energias seguem $E^{-a}$ entre $E_\text{min}$ e $E_\text{max}$; se `0 ≤ a ≤ 1`, seguem $1/E$; se `a < 0`, a energia vai em passos fixos: $\log_{10}E = \log_{10}E_\text{min} + n\,|a|$.
- **Ângulo zenital**: se `-z` ≠ `-Z`, o ângulo é sorteado de um fluxo isotrópico em superfície plana ($dN/d\cos\theta \propto \cos\theta$). Para chuveiros horizontais, use sempre `-z` = `-Z`.
- **Semente e sobrescrita**: a semente entra no nome do arquivo. Rodar duas vezes com a **mesma semente** e os mesmos parâmetros sobrescreve o arquivo anterior. A reprodutibilidade exata com a mesma semente só é garantida com o QGSJET.

### 3.3 Arquivos de configuração (`cfg/*.param`)

Além das flags, alguns parâmetros físicos ficam nos arquivos de configuração em `cfg/` (um por modelo: `conex_sibyll.param`, `conex_epos.param`, `conex_qgsjetII.param`, etc.). Os principais:

- **Cortes de energia** (*cuts*): energia mínima para hádrons/múons (não menos que 0.3 GeV) e para partículas eletromagnéticas (não menos que 1 MeV).
- **Limiar MC → CE** (*threshold*): define a partir de que fração da energia a simulação passa de Monte Carlo para equações de cascata. O padrão atual é 0.005. **Não mude** a menos que saiba o que está fazendo (1 = só equações de cascata, sem flutuações; 0 = só MC, muito lento).
- **`ixmax`**: se for 0, o ajuste de Gaisser-Hillas não é feito (economiza tempo).
- **`xmaxp`**: profundidade inclinada máxima da simulação. Se não especificada, simula até o solo.

### 3.4 Mais exemplos

```bash
# 500 chuveiros de ferro, 10^18 eV, 45 graus, EPOS LHC-R
bin/conex2r -p 5600 -e 18 -E 18 -z 45 -Z 45 -m 4 -n 500 -x ferro_e18

# Prótons com espectro E^-1 entre 10^17 e 10^20 eV, ângulos entre 0 e 60 graus
bin/conex2r -p 100 -e 17 -E 20 -a 1 -z 0 -Z 60 -m 6 -n 1000 -x proton_espectro

# Energias discretas: 10^17, 10^17.5, ..., 10^19 eV
bin/conex2r -p 100 -e 17 -E 19 -a -0.5 -z 0 -Z 0 -m 5 -n 200 -x proton_passos
```

---

## 4. Entendendo os arquivos de saída

### 4.1 Nome do arquivo

O nome é gerado automaticamente:

```
<prefixo>_<modelo>_<semente>_<partícula>.root
```

Por exemplo, o padrão é algo como `conex_eposlhcr_??????????_100.root`. O arquivo é escrito em `ROOT_OUT`.

### 4.2 Estrutura do arquivo `.root`

O arquivo contém estruturas do tipo **`TTree`**. As duas principais são:

- **`Header`**: as condições gerais da simulação (uma entrada por arquivo);
- **`Shower`**: os dados de cada chuveiro simulado (uma entrada por chuveiro);
- **`LeadingInteractions`**: só aparece se a simulação foi feita com `-K > 0`.

Cada variável dentro de uma `TTree` é chamada de **branch**, e é ela que lemos diretamente.

> ❓ **"Por que aparecem cópias das árvores?"** Ao listar o conteúdo, você pode ver algo como `Header;1`, `Shower;1`, `Shower;2`. O número após o `;` é o **ciclo** do ROOT: como o CONEX salva automaticamente a cada `-S` chuveiros, versões anteriores da árvore ficam guardadas. O ciclo de **maior número** é a versão completa, e é ele que o ROOT/uproot lê por padrão quando você pede só `"Shower"`.

```
arquivo.root
├── Header   (1 entrada)   → configuração da simulação
├── Shower   (n entradas)  → um chuveiro por entrada
│   ├── parâmetros escalares: lgE, zenith, Xfirst, Xmax, Nmax, ...
│   └── perfis (arrays de tamanho nX): X, N, dEdX, Mu, Electrons, ...
└── LeadingInteractions (só com -K > 0)
```

### 4.3 Branches da árvore `Header`

| Branch | Significado |
|---|---|
| `Seed1` | semente aleatória |
| `Particle` | ID da partícula primária |
| `Alpha` | índice espectral |
| `lgEmin`, `lgEmax` | $\log_{10}$ da energia mín./máx. (eV) |
| `zMin`, `zMax` | ângulo zenital mín./máx. (graus) |
| `Version`, `OutputVersion` | versão do CONEX e do formato de saída |
| `HEModel`, `LEModel` | modelos de alta e baixa energia (`LEModel`: 3 = GHEISHA, 8 = UrQMD 1.3.1) |
| `HiLowEgy` | energia de transição entre modelos (GeV) |
| `hadCut`, `emCut` | cortes de energia usados no perfil de partículas carregadas |
| `haCut`, `muCut`, `elCut`, `gaCut` | cortes dos perfis de hádrons, múons, elétrons e fótons |
| `hadThr`, `muThr`, `emThr` | limiares MC → CE |
| `lambdaProton`, `lambdaPion`, `lambdaHelium`, `lambdaNitrogen`, `lambdaIron` | comprimento de interação com o ar (g/cm²) em função da energia `lambdaLgE` |

> Mais informações estão na documentação do programa

### 4.4 Branches da árvore `Shower` — parâmetros escalares

São as grandezas mais usadas em análises:

| Branch | Significado | Unidade |
|---|---|---|
| `lgE` | $\log_{10}$ da energia do primário | eV |
| `zenith`, `azimuth` | ângulos do chuveiro | graus |
| `Xfirst` | profundidade da primeira interação | g/cm² |
| `Hfirst` | altitude da primeira interação | m |
| `XfirstIn` | inelasticidade da primeira interação | [0, 1] |
| **`Xmax`** | profundidade do máximo (ajuste Gaisser-Hillas) | g/cm² |
| **`Nmax`** | nº de partículas carregadas no máximo (ajuste GH) | — |
| `X0`, `p1`, `p2`, `p3` | demais parâmetros do ajuste GH | — |
| `chi2` | $\chi^2$/ndf/$\sqrt{N_\text{max}}$ do ajuste (valores pequenos são esperados) | — |
| `Xmx`, `Nmx` | máximo de um ajuste **quadrático** do perfil $N(X)$ | g/cm², — |
| `XmxdEdX`, `dEdXmx` | máximo de um ajuste quadrático do perfil $dE/dX$ | g/cm², GeV/(g/cm²) |
| `cpuTime` | tempo de CPU do chuveiro | s |

A função de Gaisser-Hillas ajustada é:

$$
N(X) = N_\text{max}\left(\frac{X-X_0}{X_\text{max}-X_0}\right)^{\frac{X_\text{max}-X_0}{\lambda(X)}}\exp\left(\frac{X_\text{max}-X}{\lambda(X)}\right), \qquad \lambda(X) = p_1 + p_2 X + p_3 X^2
$$

> **`Xmax` × `Xmx`:** `Xmax` vem do ajuste de Gaisser-Hillas (o mais usado em comparação com dados); `Xmx` vem de um ajuste quadrático em torno do pico. Os valores são próximos, mas não idênticos — diga sempre qual você usou. Para comparar com dados de detectores de fluorescência (ex.: Observatório Pierre Auger), em geral se usa o máximo do perfil de **depósito de energia** (`XmxdEdX`).

### 4.5 Branches da árvore `Shower` — perfis longitudinais

Os perfis são **arrays** de tamanho `nX` (um array por chuveiro):

| Branch | Significado |
|---|---|
| `nX` | número de pontos dos arrays |
| `X` | profundidade inclinada (g/cm²) |
| `H` | altitude (m) |
| `D` | distância ao longo do eixo (m) |
| `N` | nº de partículas carregadas acima dos cortes `hadCut`/`emCut` cruzando cada plano X |
| `dEdX` | depósito de energia (GeV/(g/cm²)) **no meio** de cada bin |
| `Mu` | nº de múons acima de `muCut` cruzando cada plano X |
| `dMu` | nº de múons produzidos em cada bin |
| `Electrons` | nº de e⁺ + e⁻ acima de `elCut` |
| `Gamma` | nº de fótons acima de `gaCut` |
| `Hadrons` | nº de hádrons acima de `haCut` |
| `EGround` | energia no X máximo: `[0]` = e/m, `[1]` = hádrons, `[2]` = múons |

> ⚠️ **Atenção com `dEdX`:** o valor `dEdX[i]` corresponde ao ponto $(X_i + X_{i+1})/2$, e **não** a `X[i]`. Ao fazer gráficos, use os pontos médios dos bins.

### 4.6 Árvore `LeadingInteractions` (opcional)

Se você rodar com `-K n`, são salvos os secundários das `n` primeiras interações da partícula líder (a mais energética). Branches úteis: `nInt`, `kinel` (inelasticidade), `pEnergy` (energia do pai), `mult` (multiplicidade), `depth` (profundidade), `Type` e `Energy` dos secundários. É útil para estudar como a física das primeiras interações afeta o chuveiro.

### 4.9 Dicas para ler a documentação

A documentação oficial do CONEX é um arquivo de texto simples. Para encontrar o significado de uma branch:

1. Procure o nome exato da branch (ex.: `"Xmax"`) — elas aparecem entre aspas na seção que descreve o arquivo ROOT.
2. Fique atento às **unidades** (g/cm², m, GeV, eV em escala log) e aos **cortes de energia** que definem cada perfil — o "número de partículas" sempre significa "acima de algum corte".
3. As notas numeradas (`[1]`, `[2]`, ...) explicam o comportamento das flags; leia-as antes de rodar algo fora do padrão.

---

## 5. Referências

Se usar o CONEX em trabalhos, cite:

- T. Bergmann *et al.*, *One-dimensional hybrid approach to extensive air shower simulation*, Astropart. Phys. **26** (2007) 420–432, [astro-ph/0606564](https://arxiv.org/abs/astro-ph/0606564).
- T. Pierog *et al.*, *First results of fast one-dimensional hybrid simulation of EAS using CONEX*, Nucl. Phys. Proc. Suppl. **151** (2006) 159–162, [astro-ph/0411260](https://arxiv.org/abs/astro-ph/0411260).

Outros links úteis:

- ROOT: <https://root.cern/>
- uproot: <https://uproot.readthedocs.io/>
- corsika-data: <https://gitlab.iap.kit.edu/AirShowerPhysics/corsika-data>
