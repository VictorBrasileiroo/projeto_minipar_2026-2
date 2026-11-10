# MiniPar 2026.2 — Compiladores / UFAL

Compilador educacional da linguagem MiniPar, desenvolvido na disciplina de Compiladores da Universidade Federal de Alagoas. A versão 2026.2 estende a base 2026.1 com Fork/Join local e distribuído por TCP, uma Web didática e os programas de teste T08–T11.

**Entrega acadêmica:** 11/11/2026 · **Repositório:** [projeto_minipar_2026-2](https://github.com/VictorBrasileiroo/projeto_minipar_2026-2) · **[Relatório técnico em PDF](docs/relatorio/relatorio.pdf)**

## Funcionalidades implementadas

- Análise léxica com tokens e diagnósticos; análise sintática com AST.
- Análise semântica, escopos, tipos e tabela de símbolos.
- Geração de código intermediário TAC e transpilação para C++ a partir da AST.
- Compilação nativa com G++ e execução dos binários gerados.
- Funções, classes, coleções e construções de controle no escopo da [BNF](docs/12_BNF_MINIPAR_2026_2.md).
- `seq`, `par`, `fork` e `join`, com runtime de tarefas local e remoto por TCP, identificação de tarefas e tratamento de falhas.
- Três interfaces: terminal, IDE Web original e Web didática com inspeção das fases, execução e download de artefatos.
- Programas T01–T11, incluindo o MiniCompilador escrito em MiniPar que produz tokens, AST, TAC, símbolos, resultado e Assembly ARMv7 para o CPUlator.

## Arquitetura

O compilador hospedeiro é escrito em Python. TAC e C++ são gerados em ramos independentes da mesma AST; o backend C++ não consome TAC.

```text
Fonte MiniPar → Lexer → Parser/AST → Semântico e símbolos
                              ├──→ TAC
                              └──→ C++ → G++ → Binário
                                                └── Runtime Fork/Join local ou TCP
```

| Área | Papel |
|---|---|
| [src/](src/) | Lexer, parser, semântico, TAC, backend e serviço de compilação |
| [runtime/](runtime/) | Biblioteca C++ de tarefas, protocolo TCP, workers e testes |
| [main.py](main.py), [menu.py](menu.py), [app.py](app.py) | Terminal/CLI e Web original |
| [web_didatica/](web_didatica/) | Interface educativa integrada ao serviço real |
| [tests/](tests/) | Programas, oráculos e verificadores T01–T11 |
| [scripts/](scripts/) | Regressão e verificações de integração |
| [docs/](docs/) | Relatório, gramática, pseudocódigos, UML, backlogs e evidências |
| [legacy/minipar-2026.1/](legacy/minipar-2026.1/) | Referência histórica preservada |

O MiniCompilador T11 é uma aplicação escrita em MiniPar: ele analisa expressões e imprime ARMv7. Não é um backend ARM do compilador hospedeiro.

## Requisitos

- Python com `venv` e `pip`; ambiente validado com **Python 3.12.4**.
- **Flask 3.0.0**, declarado em [requeriments.txt](requeriments.txt).
- G++ com suporte a **C++20** e threads; no Windows foi validado o **MSYS2 UCRT64 G++ 16.2.0**.
- Navegador para as interfaces Web; a IDE original carrega CodeMirror de uma CDN.
- CPUlator ARMv7 generic para executar os arquivos ARM do T11.

O ambiente da auditoria foi Windows 11. Outros sistemas não foram homologados nesta rodada. Os serviços Web destinam-se ao uso local em `127.0.0.1`.

## Instalação

Com Python e G++ já instalados, abra o PowerShell na raiz do projeto. Ajuste apenas o caminho de `CXX` caso sua instalação do compilador seja diferente.

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requeriments.txt

$env:CXX = 'C:\msys64\ucrt64\bin\g++.exe'
$env:PATH = (Split-Path -Parent $env:CXX) + ';' + $env:PATH
$env:PYTHONIOENCODING = 'utf-8'
$OutputEncoding = [Console]::InputEncoding = [System.Text.UTF8Encoding]::new($false)

.\.venv\Scripts\python.exe -m pip check
& $env:CXX --version
```

O diretório das DLLs do UCRT64 deve estar no PATH do processo para executar os binários. Essas variáveis valem apenas para a janela atual. Não é necessário ativar o ambiente virtual ao usar seu interpretador diretamente.

Detalhes do ambiente: [guia de instalação](docs/10_GUIA_SETUP_REPOSITORIO.md). Deploy em dois computadores: [README do runtime, seção 7](runtime/README.md#7-deploy-em-dois-computadores).

## Como executar

### Terminal e CLI

```powershell
.\.venv\Scripts\python.exe -B main.py
```

Escolha **1** para o menu de testes ou **2** para abrir a Web original. Para compilar diretamente um programa:

```powershell
.\.venv\Scripts\python.exe -B main.py tests/teste7_quicksort_iterativo_tac.minipar t07
.\output\t07.exe
```

A CLI gera `output/t07.tac`, `output/t07.cpp` e `output/t07.exe`. Artefatos de execução ficam em `output/`, ignorado pelo Git.

### Web original

```powershell
.\.venv\Scripts\python.exe -B app.py
```

Acesse **http://127.0.0.1:5000**. Selecione um exemplo ou edite o fonte, clique em **Compile & Run** e consulte saída, TAC e C++. A opção 2 do menu inicia a mesma interface.

### Web didática

Em outra janela, com o mesmo ambiente de compilação configurado:

```powershell
.\.venv\Scripts\python.exe -B web_didatica/server.py
```

Acesse **http://127.0.0.1:5001**. A interface apresenta tokens, AST navegável, símbolos, diagnósticos, TAC, C++, compilação e execução com stdin/timeout. Permite baixar TAC, C++ e binário; no T11, também extrair o `.arm` da saída. Instruções completas no [README da Web didática](web_didatica/README.md).

## Testes

Com G++ e suas DLLs configurados na janela:

```powershell
.\.venv\Scripts\python.exe -B scripts/regressao_int02.py --repeat 10
.\.venv\Scripts\python.exe -B runtime/tests/run_tests.py --keep-going
```

A regressão executa T01–T11, terminal, as duas Webs, integração INT-01 e suítes do núcleo. Os logs e JSONs são gravados em `output/int02/` e nos diretórios dos verificadores. O orquestrador retorna **0** com todas as etapas PASS, **1** com FAIL e **3** quando há PARTIAL/BLOCKED sem FAIL. É necessário ler os resultados individuais: o **T04 PARTIAL conhecido pode produzir exit 3**.

O benchmark pode ser repetido separadamente:

```powershell
.\.venv\Scripts\python.exe -B tests/2026_2/bench/run_bench.py --rapido
```

Método e parâmetros completos: [benchmark](tests/2026_2/bench/README.md). Critérios de comparação: [validação e aceite](docs/05_VALIDACAO_E_ACEITE.md).

## Resultados e limitações

A auditoria local de 10/11/2026, no commit `d851170` desta cópia, reproduziu **dez PASS e um PARTIAL em T01–T11**. T01 e T02 passaram **30/30 execuções cada**. Núcleo, runtime, três interfaces e benchmark passaram; as duas Webs foram verificadas em navegador real. Os registros dessa rodada são locais, em `output/validacao_final/`, e não são versionados.

- **T04:** compila, executa e classifica corretamente as quatro entradas XOR. Os valores numéricos diferem da referência; o aceite numérico continua PARTIAL, sem tolerância presumida. [Análise numérica](docs/auditorias/REL-PRE_FECHAMENTO_EIXO_A.md#2-t04--auditoria-numérica-e-proposta-para-decisão).
- **Fork/Join:** testes locais e TCP em loopback executados. A prova histórica do **T08 entre dois computadores** está no [RUN-06](docs/evidencias/RUN-06_REPRODUCAO_DOIS_PCS.md); ela não equivale a uma nova execução física de T09/T10. RUN-06 mantém pendente a reprodução por outro integrante somente pelo README e a queda durante uma tarefa em rede.
- **T10:** cálculo e oráculos aprovados; o programa produz ASCII e o harness gera SVG. O aceite desse formato em relação à referência Turtle continua na [ADR-014](docs/03_DECISOES.md#adr-014--proposta-de-resolução-em-duas-etapas).
- **Desempenho e falhas:** cargas equivalentes, medições repetidas e cenários de worker indisponível, worker morto e timeout documentados no [RUN-05](docs/evidencias/RUN-05_BENCHMARKS.md). Tempos de loopback e de rede física são apresentados separadamente.
- **T11/CPUlator:** casos obrigatórios com resultados **22, −84 e 22**. Há [evidências ARM e capturas versionadas](tests/2026_2/test11/evidencias/README.md); os três casos foram reexecutados na auditoria local. A conferência humana e a comparação pendente do Java no caso 2 permanecem explicitadas.
- O runtime remoto não oferece autenticação/TLS nem cancelamento de tarefa após timeout; as Webs executam binários locais. Use apenas em ambiente controlado, conforme a [documentação do runtime](runtime/README.md).

Resultados versionados e escopo da regressão anterior: [INT-02](docs/auditorias/INT-02_REGRESSAO_INTEGRAL.md). O planejamento e as auditorias mantêm suas datas e estados históricos; eles não substituem os resultados de execução.

## Documentação

- [Relatório técnico final (PDF)](docs/relatorio/relatorio.pdf)
- [BNF MiniPar 2026.2](docs/12_BNF_MINIPAR_2026_2.md)
- [Pseudocódigos do núcleo](docs/13_PSEUDOCODIGOS_NUCLEO.md)
- [UML: casos de uso, componentes, classes e sequências](docs/15_UML_MINIPAR_2026_2.md)
- [Contratos técnicos](docs/02_CONTRATOS_TECNICOS.md)
- [Product Backlog e Sprint Backlogs](docs/14_PRODUCT_E_SPRINT_BACKLOGS.md)
- [Testes e critérios de aceite](docs/05_VALIDACAO_E_ACEITE.md)
- [Evidências de rede RUN-03](docs/evidencias/RUN-03_DOIS_COMPUTADORES.md), [RUN-05](docs/evidencias/RUN-05_BENCHMARKS.md) e [RUN-06](docs/evidencias/RUN-06_REPRODUCAO_DOIS_PCS.md)
- [Runtime: API, protocolo e deploy](runtime/README.md)
- [MiniCompilador T11](tests/2026_2/test11/README.md)
- [Checklist de entrega](docs/11_CHECKLIST_DE_ENTREGA.md)

## Equipe e créditos

**MiniPar 2026.2 — UFAL, disciplina Compiladores, professor Arturo Hernandez Dominguez:**

- **Victor André Lopes Brasileiro:** núcleo do compilador e integração.
- **José Riquelme Teixeira da Silva:** Web didática e MiniCompilador T11.
- **Erasmo da Silva Sá Junior:** runtime, comunicação distribuída e testes T08–T10.

A implementação herdada de 2026.1 é referenciada em [luizwhirl/minipar-improvement](https://github.com/luizwhirl/minipar-improvement). Créditos preservados: **Emanuele Vitória de Jesus Lima** ([Manu-Vii](https://github.com/Manu-Vii)), **José Lucas de Oliveira Quintela** ([lucasqtl](https://github.com/lucasqtl)) e **Luiz Miguel de Melo Bomfim** ([luizwhirl](https://github.com/luizwhirl)).

O snapshot histórico permanece em [legacy/minipar-2026.1/](legacy/minipar-2026.1/). Seu commit de origem e a autorização de redistribuição não foram comprovados nos materiais locais; não se declara licença ou autoria exclusiva sobre o código herdado. [Proveniência](docs/08_ESTADO_DA_BASE_2026_1.md) e [decisões registradas](docs/03_DECISOES.md).
