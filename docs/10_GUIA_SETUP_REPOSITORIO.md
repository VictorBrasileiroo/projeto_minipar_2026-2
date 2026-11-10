# Instalação e execução do MiniPar 2026.2

Este guia se aplica ao projeto implementado no repositório [projeto_minipar_2026-2](https://github.com/VictorBrasileiroo/projeto_minipar_2026-2), branch `main`. O código ativo, o runtime, os testes e a referência 2026.1 já estão presentes. Não é necessário importar a base novamente ou inicializar outro repositório.

O histórico de importação e ambiente permanece nas auditorias [SET-01](auditorias/SET-01_VALIDACAO_ESTRUTURAL.md) e [SET-02](auditorias/SET-02_AMBIENTE_E_SMOKE.md); os comandos antigos nelas são registros das respectivas rodadas.

## 1. Ferramentas

O ambiente verificado em 10/11/2026 utiliza:

| Ferramenta | Ambiente validado |
|---|---|
| Sistema | Windows 11 |
| Python | CPython 3.12.4, com `venv` e `pip` |
| Dependência Python direta | Flask 3.0.0, em [requeriments.txt](../requeriments.txt) |
| Compilador | G++ 16.2.0, MSYS2 UCRT64, com C++20 e threads |
| Navegador | Chrome, para as duas interfaces Web |
| Simulador ARM | CPUlator, sistema ARMv7 generic |

Instale Python e uma toolchain com C++20 antes de configurar o projeto. No Windows, a instalação MSYS2 usada nas verificações fornece `C:\msys64\ucrt64\bin\g++.exe` e as DLLs `libstdc++-6.dll`, `libgcc_s_seh-1.dll` e `libwinpthread-1.dll`. Um MinGW antigo no PATH pode não atender ao requisito C++20.

Essas versões descrevem o ambiente executado, não uma homologação de todas as versões ou sistemas operacionais. O [README do runtime](../runtime/README.md) inclui instruções técnicas para Windows e Linux; Linux não foi executado na auditoria desta cópia.

## 2. Ambiente virtual

Na raiz do projeto, em PowerShell:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requeriments.txt
.\.venv\Scripts\python.exe --version
.\.venv\Scripts\python.exe -m pip check
```

Use o interpretador da venv nos próximos comandos. O nome histórico `requeriments.txt` é o nome real do arquivo. Se a venv já existe, basta utilizá-la e conferir as dependências; não é necessário recriá-la.

## 3. G++ e codificação na janela atual

```powershell
$env:CXX = 'C:\msys64\ucrt64\bin\g++.exe'
$env:PATH = (Split-Path -Parent $env:CXX) + ';' + $env:PATH
$env:PYTHONIOENCODING = 'utf-8'
$OutputEncoding = [Console]::InputEncoding = [System.Text.UTF8Encoding]::new($false)
& $env:CXX --version
```

Ajuste o caminho de `CXX` para o executável instalado; não inclua flags nessa variável. O PATH deve incluir o diretório das DLLs da mesma toolchain, necessário tanto para compilar quanto para executar. As variáveis acima são locais ao processo e aos seus filhos.

A codificação sem BOM evita que entradas como `par` enviadas pelo PowerShell cheguem com um caractere extra ao programa. O procedimento para demonstrar tarefas remotas está detalhado no [runtime, seção 7](../runtime/README.md#7-deploy-em-dois-computadores).

## 4. Executar

```powershell
# Menu: 1 = terminal; 2 = Web original
.\.venv\Scripts\python.exe -B main.py

# CLI direta: gera TAC, C++ e executável
.\.venv\Scripts\python.exe -B main.py tests/teste7_quicksort_iterativo_tac.minipar t07
.\output\t07.exe
```

Para iniciar diretamente cada Web, em janelas separadas e com o ambiente acima:

```powershell
.\.venv\Scripts\python.exe -B app.py
# http://127.0.0.1:5000

.\.venv\Scripts\python.exe -B web_didatica/server.py
# http://127.0.0.1:5001
```

A Web didática usa o serviço real por padrão. Seus exemplos fornecem o fonte, e as fases e resultados são produzidos pelo compilador/binário. Consulte a [documentação da interface](../web_didatica/README.md). A Web original depende do carregamento de CodeMirror pela CDN.

## 5. Conferir o funcionamento

```powershell
.\.venv\Scripts\python.exe -B scripts/regressao_int02.py --repeat 10
.\.venv\Scripts\python.exe -B runtime/tests/run_tests.py --keep-going
```

A regressão salva evidências em `output/`. Exit 3 do orquestrador significa PARTIAL/BLOCKED sem FAIL; o T04 tem divergência numérica conhecida, com classificação correta e aceite numérico pendente. Leia o JSON individual, conforme a [matriz de validação](05_VALIDACAO_E_ACEITE.md) e a [regressão INT-02](auditorias/INT-02_REGRESSAO_INTEGRAL.md).

## 6. Diagnóstico básico

- **G++ não encontrado ou incompatível:** confira `CXX`, a versão e o suporte C++20; use a toolchain correta, não apenas o primeiro MinGW do PATH.
- **Binário não inicia por DLL ausente:** inclua o diretório de DLLs do mesmo UCRT64 no PATH da janela.
- **Porta 5000/5001 ocupada:** encerre a instância que você iniciou antes de iniciar outra. Não é necessário abrir portas públicas.
- **Editor original não carrega:** confira o acesso à CDN usada pelo CodeMirror.
- **Falhas de worker/timeout:** consulte o [protocolo e os problemas comuns do runtime](../runtime/README.md).

As Webs compilam e executam programas locais; mantenha o uso em ambiente controlado. Créditos e proveniência da base estão no [README](../README.md#equipe-e-créditos) e no [estado histórico da base](08_ESTADO_DA_BASE_2026_1.md).
