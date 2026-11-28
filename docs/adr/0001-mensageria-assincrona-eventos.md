# ADR 0001 — Adoção de Mensageria Assíncrona para Notificações e Eventos Internos

- **Status:** Aceito
- **Data:** 13/11/2026 (Revisado em 27/11/2026)
- **Projeto:** Sistema Web de Gestão de Aprendizagem e Estágios UniSenai (Equipe Equebis)
- **Autores:** Douglas Liz, Eduardo Paes, Gabriel Henrique e Giovanna Assis

## Contexto
O sistema atende três perfis de usuários (Alunos UniSenai, Representantes de Empresas Parceiras e Coordenação UniSenai) em fluxos críticos como envio de candidaturas, publicação de vagas e aprovação cadastral de empresas. Executar a persistência principal no `BancoDados` e, na mesma requisição HTTP síncrona, gerar notificações internas e indexar competências para filtros gera acoplamento temporal, degrada o tempo de resposta e expõe a interface web a falhas em cascata quando tarefas secundárias sofrem lentidão.

## Decisão
Adotar uma camada de **Mensageria Assíncrona** (`Mensageria`) acompanhada de um consumidor dedicado (`WorkerNotificacao`) entre a `APIBackend` e os processamentos secundários, abstraída no código por meio da interface `IEventoPublisher`. Após o `COMMIT` da transação principal no `BancoDados`, a `APIBackend` publica um evento de domínio na fila (com confirmação explícita / padrão Outbox) e retorna imediatamente a confirmação HTTP ao cliente (`WebApp`).

## Consequências Positivas
- **Resiliência e isolamento de falhas:** Oscilações temporárias no módulo de notificações ou indexação de busca não interrompem candidaturas, credenciamentos nem publicações de vagas.
- **Menor latência percebida:** O tempo de resposta da `APIBackend` restringe-se à validação e gravação transacional no banco relacional (limitada ao timeout de 3 segundos).
- **Suporte nativo a retentativas:** Eventos não processados retornam `NACK`, podendo ser reprocessados automaticamente com *backoff* ou isolados em *Dead Letter Queue* (DLQ) para auditoria.

## Consequências Negativas (Trade-offs)
- **Complexidade operacional adicional:** Exige configurar, manter e monitorar um serviço de broker de mensageria e o processo contínuo do `WorkerNotificacao`.
- **Consistência eventual:** Notificações internas e índices de busca passam a ser atualizados instantes após o retorno HTTP `201 Created` / `200 OK`.
- **Exigência de idempotência:** O `WorkerNotificacao` precisa validar `evento_id` único (`INotificacaoRepository.existeEventoProcessado`) para não gravar alertas repetidos em cenários de reentrega (*at-least-once delivery*).
