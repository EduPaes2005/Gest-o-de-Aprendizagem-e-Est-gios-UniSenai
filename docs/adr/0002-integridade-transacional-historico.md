# ADR 0002 — Integridade Transacional, Timeout de Persistência e Trilha de Auditoria de Candidaturas

- **Status:** Aceito
- **Data:** 27/11/2026
- **Projeto:** Sistema Web de Gestão de Aprendizagem e Estágios UniSenai (Equipe Equebis)
- **Autores:** Douglas Liz, Eduardo Paes, Gabriel Henrique e Giovanna Assis

## Contexto
Durante a auditoria de engenharia do caos na Sprint 4 sobre o fluxo de candidatura, foram identificados quatro riscos críticos de persistência no `BancoDados`:
1. Possibilidade de candidaturas duplicadas do mesmo aluno na mesma vaga por cliques duplos ou retentativas;
2. Condição de corrida (*race condition*) quando uma vaga é encerrada ou suspensa simultaneamente ao envio de uma candidatura;
3. Sobrescrita da coluna `status` na entidade `Candidatura`, apagando o histórico de evolução exigido pelos casos de uso de acompanhamento;
4. Degradação em cascata caso consultas no banco ultrapassem limites aceitáveis de latência.

## Decisão
1. Impor restrição de chave única composta `UNIQUE(aluno_id, vaga_id)` na tabela de candidaturas, retornando `HTTP 409 Conflict` em caso de violação.
2. Executar a validação de status da vaga (`Aberta == true`) e a inserção da candidatura dentro da mesma transação atômica com bloqueio pessimista (`buscarAbertaComLock`) e **timeout máximo de 3 segundos**, retornando `HTTP 503 Service Unavailable` em caso de `TimeoutException`.
3. Separar o histórico de transições na entidade `HistoricoStatusCandidatura` (composição 1-para-N imutável), gravando cada mudança de estado junto à atualização de `Candidatura.statusAtual`.

## Consequências Positivas
- **Garantia ACID no núcleo de negócio:** Impossibilita candidaturas em vagas já encerradas/suspensas ou duplicadas mesmo sob alta concorrência.
- **Rastreabilidade completa:** Atende diretamente ao caso de uso *Consultar histórico da candidatura* e facilita auditoria pela Coordenação.
- **Proteção contra esgotamento de threads:** O corte de timeout em 3s impede travamento indefinido do pool de conexões da `APIBackend`.

## Consequências Negativas (Trade-offs)
- **Contenção sob alta concorrência na mesma vaga:** O bloqueio transacional na linha da vaga pode gerar leve fila de escrita em picos simultâneos.
- **Maior volume de armazenamento:** A manutenção da tabela `HistoricoStatusCandidatura` aumenta o número de registros inseridos por ciclo de vida da candidatura.
