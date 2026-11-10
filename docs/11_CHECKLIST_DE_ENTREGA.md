# Checklist acadêmico e técnico — entrega de 11/11/2026

Atualizado documentalmente em 10/11/2026 para a cópia independente, branch `main`, commit de implementação `d851170`. As marcações abaixo distinguem execução automatizada, evidências históricas e aceite humano. A organização documental não reclassifica cards nem transforma PARTIAL em PASS.

## Produto e testes

| Verificação | Situação | Evidência e escopo |
|---|---|---|
| Pipeline Lexer → Parser/AST → Semântico/símbolos; AST → TAC e AST → C++ → G++ → binário | PASS no escopo executado | Regressão local de 10/11; [INT-02 versionado](auditorias/INT-02_REGRESSAO_INTEGRAL.md) registra a rodada anterior |
| T01 e T02 | PASS, 30/30 cada na auditoria local | Comunicação loopback no T01 e resultados concorrentes no T02 |
| T03 e T05–T07 | PASS | Oráculos numéricos/ordenação; T07 inclui mutação do TAC e assertions de índices |
| T04 XOR | PARTIAL | Classificação correta; [aceite numérico pendente](auditorias/REL-PRE_FECHAMENTO_EIXO_A.md#2-t04--auditoria-numérica-e-proposta-para-decisão) |
| T08 | PASS no harness; rede física histórica documentada | 32 checagens locais/loopback; [RUN-06](evidencias/RUN-06_REPRODUCAO_DOIS_PCS.md) para dois computadores |
| T09 | PASS no harness | 54 checagens; cenário remoto executado em loopback, não prova física de outro computador |
| T10 | PASS no cálculo/harness; aceite gráfico pendente | 126 checagens; ASCII + SVG externo; [ADR-014](03_DECISOES.md#adr-014--proposta-de-resolução-em-duas-etapas) |
| T11 | PASS no verificador, 48/48 | MiniCompilador escrito em MiniPar, casos obrigatórios e inválidos, extração ARM |
| CPUlator | Casos obrigatórios 22, −84 e 22 | [Provas versionadas](../tests/2026_2/test11/evidencias/README.md); reexecução assistida por IA em 10/11, conferência humana separada |
| Runtime e falhas | PASS local/loopback | Seis grupos do runner; F1/F2/F3, cinco repetições cada no benchmark completo |
| Terminal, Web original e Web didática | PASS no recorte executado | Testes automáticos e inspeção em navegador real das duas Webs |

A rodada local de 10/11 é preservada em `output/validacao_final/` (ignorado pelo Git). Ela não substitui as evidências versionadas de rede física, nem demonstra portabilidade universal ou aceite docente.

## Documentação acadêmica

- [x] [BNF](12_BNF_MINIPAR_2026_2.md) e [pseudocódigos](13_PSEUDOCODIGOS_NUCLEO.md) preservados.
- [x] [UML](15_UML_MINIPAR_2026_2.md), [Product/Sprint Backlogs](14_PRODUCT_E_SPRINT_BACKLOGS.md), [contratos](02_CONTRATOS_TECNICOS.md) e [decisões](03_DECISOES.md) preservados.
- [x] Evidências RUN-03, RUN-05, RUN-06, regressões e CPUlator preservadas.
- [x] Instruções atuais de instalação, execução das três interfaces e testes no [README](../README.md) e no [guia de instalação](10_GUIA_SETUP_REPOSITORIO.md).
- [x] Créditos da implementação herdada preservados.
- [x] [Relatório acadêmico em PDF](relatorio/relatorio.pdf) presente, 132 páginas, 2.730.379 bytes; integridade preservada.
- [ ] **Corrigir manualmente a capa do PDF:** ainda contém `https://github.com/VictorBrasileiroo/minipar-2026-2`; o destino desta entrega é `https://github.com/VictorBrasileiroo/projeto_minipar_2026-2`. O PDF não foi editado nesta preparação.
- [ ] Equipe concluir a revisão acadêmica do PDF e conferir acesso aos arquivos após a publicação autorizada.

O material de apresentação será entregue separadamente e não é um arquivo obrigatório deste repositório.

## Ressalvas de aceite e demonstração

- [ ] Registrar decisão de aceite numérico do T04 sem substituir o oráculo histórico para forçar PASS.
- [ ] Registrar aceite do formato ASCII/SVG do T10, com confirmação docente quando necessária.
- [ ] Concluir RUN-06 por outro integrante seguindo somente o README; a queda durante tarefa em rede física ainda não está demonstrada. O registro atual mostra WORKER_UNAVAILABLE, não WORKER_LOST.
- [ ] Responsável humano conferir os casos CPUlator e tratar ou explicitar a pendência da comparação Java no caso 2.
- [ ] Registrar o tratamento de proveniência/direitos da base, conforme [ADR-018](03_DECISOES.md#adr-018--divergência-de-privacidade-e-direitos-sobre-o-legado).
- [ ] Trio conferir roteiro de apresentação, ambiente de demonstração e backup das evidências.

## Integridade e publicação

Na preparação documental, somente Markdown e as instruções internas explicitamente removidas podem mudar. Código, testes, configurações, dados, capturas, resultados e PDF devem conservar os hashes anteriores.

A verificação de integridade usa manifesto de hashes, `git diff --name-status`, `git diff --check` e conferência de branch/remoto. Não exige repetir a regressão quando os arquivos funcionais permanecem idênticos. Nenhum commit, push, mudança de remoto ou alteração no repositório original faz parte desta etapa.
