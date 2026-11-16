# 🎲 Software de Sorteio Público — ESEBA/UFU (v3.0)

Sistema web desenvolvido para a realização dos sorteios públicos anuais de ingresso na **Escola de Educação Básica da Universidade Federal de Uberlândia (ESEBA-UFU)**.

O software unifica em uma única interface web todas as etapas do processo seletivo: importação da base de candidatos, validação de integridade via hash MD5, ranqueamento aleatório auditável (por semente), distribuição automática de vagas imediatas, remanejamento de vagas ociosas e geração de listas de espera e relatórios oficiais (PDF e CSV).

## 📋 Índice

* [Sobre o Projeto](#-sobre-o-projeto)

* [Tecnologias e Bibliotecas Utilizadas](#-tecnologias-e-bibliotecas-utilizadas)

* [Estrutura de Diretórios](#-estrutura-de-diretórios)

* [Formato da Planilha de Entrada (CSV)](#-formato-da-planilha-de-entrada-csv)

* [Regras de Negócio e Funcionamento](#-regras-de-negócio-e-funcionamento)

* [Como Executar](#-como-executar)

* [Histórico de Versões (Changelog)](#-histórico-de-versões-changelog)

* [Equipe e Créditos](#-equipe-e-créditos)

## 🎯 Sobre o Projeto

Anteriormente, o processo de sorteio da ESEBA dependia de dois sistemas distintos (uma aplicação web para simular o globo de sorteio e um software em Java para distribuição de vagas e relatórios), exigindo a manipulação de múltiplas planilhas e a transposição manual de dados.

Este sistema resolve esse gargalo ao centralizar todo o fluxo em uma aplicação web *client-side*, processando uma **planilha única** com todos os inscritos, garantindo transparência durante as transmissões ao vivo e assegurando total reprodutibilidade dos resultados através de sementes (*seeds*) criptográficas/temporais.

## 🛠 Tecnologias e Bibliotecas Utilizadas

O projeto foi construído utilizando tecnologias web padrão, dispensando a necessidade de servidores *back-end* complexos para execução:

* **HTML5 & CSS3** — Estruturação semântica e estilização.

* **JavaScript (ES6+)** — Lógica principal de leitura, filtragem, ranqueamento e regras de distribuição.

* [**Bootstrap**](https://getbootstrap.com/) — Responsividade e componentes de interface.

* [**seedrandom.js**](https://github.com/davidbau/seedrandom) — Geração de números pseudoaleatórios baseados em semente (*seed*) auditável.

* [**spark-md5.js**](https://github.com/satazor/js-spark-md5) — Cálculo de hash MD5 do arquivo CSV importado para comprovação de integridade.

* [**jsPDF**](https://github.com/parallax/jsPDF) — Geração e exportação automática dos relatórios oficiais em formato `.pdf`.

* [**anime.js**](https://animejs.com/) — Animações visuais durante a execução do sorteio.

## 📁 Estrutura de Diretórios

```
├── Sorteio_v3.html       # Arquivo principal (ponto de entrada da aplicação)
├── css/                  # Folhas de estilo (Bootstrap e customizações visuais)
├── images/               # Recursos visuais (brasão, logos da ESEBA/UFU, banners do ano)
├── js/                   # Scripts de lógica e bibliotecas
│   ├── sorteio.js        # Lógica principal: leitura CSV, MD5, ranqueamento e regras de vagas
│   ├── salvapdf.js       # Formatação e exportação do relatório oficial em PDF
│   ├── animation.js      # Controle das animações de sorteio
│   ├── seedrandom.js     # Gerador de números aleatórios por semente
│   ├── spark-md5.js      # Cálculo de hash MD5
│   └── bootstrap.js      # Scripts de interface do Bootstrap
└── planilhas/            # Diretório sugerido para armazenamento das planilhas CSV

```

## 📊 Formato da Planilha de Entrada (CSV)

Para que o sistema processe os candidatos corretamente, o arquivo de entrada deve estar no formato **`.csv`** utilizando **ponto e vírgula (`;`)** como separador de colunas.

Cada linha representa um candidato (ou dupla/grupo de gêmeos) e deve conter as seguintes colunas na ordem exata:

| Coluna | Tipo | Descrição / Valores Aceitos | 
| ----- | ----- | ----- | 
| `id` | `String` | Identificador único do candidato. | 
| `sorteio` | `Inteiro` | Número/ordem de inscrição do candidato. | 
| `nome` | `String` | Nome completo do candidato (ou nomes, caso haja gêmeos). | 
| `gemeo` | `Inteiro` | Indica inscrição de gêmeos (`0` = Não; `1` = Sim). | 
| `irmao_1` | `String` | *(Campo legado em desuso)* Nome do primeiro irmão. | 
| `irmao_2` | `String` | *(Campo legado em desuso)* Nome do segundo irmão. | 
| `nascimento` | `Date` | Data de nascimento do candidato (`DD/MM/AAAA`). | 
| `deficiente` | `Inteiro` | Indica Pessoa com Deficiência — PCD (`0` = Não; `1` = Sim). | 
| `mae` | `String` | CPF da mãe ou responsável legal (parcialmente mascarado no PDF final). | 
| `serie` | `String` | Descrição textual da série/período pretendido. | 
| `cod_serie` | `Inteiro` | Código numérico da série/período (ver tabela abaixo). | 
| `pse` | `Inteiro` | Código da modalidade de concorrência (ver tabela abaixo). | 

### Códigos de Série (`cod_serie`)

* **`11`**: 1º Período da Educação Infantil

* **`22`**: 2º Período da Educação Infantil

* **`1` a `9`**: 1º ao 9º Ano do Ensino Fundamental

### Códigos de Modalidade (`pse`)

* **`0`**: Ampla Concorrência (AC) e Pessoa com Deficiência (PCD)

* **`1`**: Modalidade M1 (Perfil Socioeconômico - PSE)

* **`2`**: Modalidade M2 (PPI / PSE)

* **`3`**: Modalidade M3 (PPI)

* **`4`**: Modalidade M4 (PCD / PSE)

* **`5`**: Modalidade M5 (PCD / PPI)

* **`6`**: Modalidade M6 (PCD)

* **`7`**: Modalidade M7 (Modalidade extra / Refugiados)

* **`8`**: Modalidade M8 (Modalidade extra configurável)

* **`99`**: Modalidade Ampla Concorrência

## ⚙️ Regras de Negócio e Funcionamento

1. **Integridade dos Dados (MD5):** Ao carregar o arquivo `.csv`, o sistema calcula e exibe imediatamente o **Hash MD5** da planilha, permitindo que a comissão e o público auditem que o arquivo não sofreu alterações.

2. **Semente Auditável (*Seed*):** O sorteio pode ser executado utilizando o *timestamp* exato do clique (gerado automaticamente) ou através de uma **semente inserida manualmente** (ideal para auditorias e testes de reprodutibilidade).

3. **Ordem de Precedência (Lei nº 14.723/2023):** As vagas da **Ampla Concorrência (M0 / AC)** são sorteadas primeiro. Candidatos cotistas (M1 a M7) sorteados na Ampla Concorrência ocupam essa vaga e são automaticamente removidos da disputa por vagas imediatas em suas modalidades específicas.

4. **Remanejamento de Vagas Ociosas:** Caso uma modalidade não possua candidatos suficientes para preencher suas vagas imediatas, o sistema redistribui automaticamente as vagas remanescentes seguindo a cascata:

   * `M2` $\rightarrow$ `M3` $\rightarrow$ `M7`

   * `M4` e `M5` $\rightarrow$ `M6` $\rightarrow$ `Ampla Concorrência (AC)`

5. **Tratamento de Gêmeos:** O algoritmo contempla regras específicas para alocação conjunta de candidatos sinalizados com `gemeo = 1`, inclusive em cenários limítrofes (últimas vagas disponíveis).

6. **Listas de Espera Simultâneas:** Candidatos não contemplados nas vagas imediatas podem figurar simultaneamente tanto na lista de espera da **Ampla Concorrência** quanto na lista de espera de sua **modalidade específica**.

7. **Exportação de Resultados:** Ao final, o painel técnico exibe a semente utilizada, o horário exato (UTC-03:00) e disponibiliza botões para download do **Relatório Oficial em PDF** (pronto para publicação) e da **Planilha Final em CSV** (para uso da secretaria).

## 🚀 Como Executar

Por ser uma aplicação *front-end* pura, não há necessidade de instalação de dependências ou compilação:

1. Clone ou baixe este repositório para o computador local:

   ```bash
   git clone https://github.com/seu-usuario/sorteio-eseba-ufu.git
   ```

2. Acesse a pasta raiz do projeto e abra o arquivo **`Sorteio_v3.html`** em qualquer navegador moderno (Google Chrome, Mozilla Firefox, Microsoft Edge).

3. Na interface:

   * Defina o **Nome do Sorteio** (ex: `Eseba/2025`).

   * Configure a quantidade de vagas imediatas e de lista de espera para cada série/modalidade (ou habilite modalidades extras M7/M8, se necessário).

   * Clique em **"Escolher arquivo"** e selecione a planilha `.csv` padronizada.

   * *(Opcional)* Confira a lista de candidatos importados ou clique em **"Ocultar Candidatos"** para manter a tela limpa durante a transmissão.

   * *(Opcional)* Marque **"Inserir semente manualmente"** caso deseje reproduzir um sorteio anterior.

   * Clique em **"Sortear"** e baixe os arquivos de resultado (`.pdf` e `.csv`).

## 📌 Histórico de Versões (Changelog)

### **v3.0 (Junho/2025)**

* **Ranqueamento Único Global:** Substituição dos sorteios independentes por turma por um ranqueamento global único de toda a planilha a partir de uma única semente, elevando a equidade e a transparência do processo.

* **Dupla Listagem na Espera:** Atualização da função `calculaRestantes` (com o parâmetro `espera`), permitindo que candidatos não aprovados nas vagas imediatas concorram simultaneamente nas listas de espera da Ampla Concorrência e de sua modalidade de cota.

* **Melhorias de Interface:** Ajustes de layout e usabilidade na apresentação das tabelas de resultados.

### **v2.0 (2024/2025)**

* **Adequação Legal:** Atualização das regras conforme a **Lei nº 14.723/2023** e a **Resolução CONSUN nº 36/2022**.

* **Precedência da Ampla Concorrência:** Alteração da Ampla Concorrência para o código `M0` (exibida em primeiro lugar nos relatórios) e execução do seu sorteio antes das modalidades de reserva de vagas.

* **Suporte a Refugiados:** Utilização da modalidade `M7` sem realocação automática de vagas, conforme previsto em edital.

### **v1.0 (2022/2023)**

* Unificação dos antigos sistemas (Web + Java) em uma única plataforma HTML/CSS/JS.

* Implementação de leitura de planilha única CSV, validação por hash MD5, geração de semente via `seedrandom.js` e exportação direta em PDF via `jsPDF`.

## 👥 Equipe e Créditos

Desenvolvido no âmbito da **Escola de Educação Básica da Universidade Federal de Uberlândia (ESEBA-UFU)** — Uberlândia/MG.

* **Desenvolvimento (Estagiários):**

  * Paulo Diego S. Souza

  * Pedro Ribeiro

  * Cayo Oliveira

* **Supervisão:**

  * Edson Silva de Lima

© 2025 Escola de Educação Básica — Universidade Federal de Uberlândia (ESEBA-UFU). Todos os direitos reservados.