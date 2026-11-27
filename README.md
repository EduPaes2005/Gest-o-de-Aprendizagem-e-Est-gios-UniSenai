# Sistema Web de Gestão de Aprendizagem e Estágios — UniSenai (Equipe Equebis)

Documentação Viva (*Diagrams-as-Code* / *Models-as-Code*) da arquitetura do **Sistema Web de Gestão de Aprendizagem e Estágios da UniSenai**, desenvolvida para a disciplina de Modelagem de Software (Engenharia de Software — 4º Período, Prof. Sidney Leandro da Silva).

## Equipe Equebis
- Douglas Liz
- Eduardo Paes
- Gabriel Henrique
- Giovanna Assis

## Escopo Consolidado (Versão 1 — Sprint 2 / Sprint 4)
A plataforma conecta perfis de competências de alunos da UniSenai às demandas técnicas de empresas parceiras, atendendo três perfis de acesso:
1. **Aluno UniSenai:** cadastra/atualiza perfil e competências, pesquisa/filtra vagas por competência e modalidade, envia/cancela candidaturas e acompanha histórico de status.
2. **Representante da Empresa Parceira:** cadastra empresa, publica/edita/encerra vagas, analisa perfis de candidatos e atualiza situação das candidaturas.
3. **Coordenação UniSenai:** avalia/aprova/suspende cadastro de empresas parceiras, gerencia catálogo de competências, fiscaliza/suspende vagas e consulta indicadores.

### Fora de Escopo na Versão 1
Itens originalmente aventados na Sprint 1/MoSCoW inicial que foram formalmente removidos do escopo v1 na Sprint 2:
- Motor de recomendação automática / algoritmo de *match* e classificação de candidatos;
- Elaboração, assinatura digital ou gestão de contratos/termos de estágio;
- Pagamento de bolsas, salários ou encargos;
- Integrações externas (Gateway de Sistema Acadêmico/SGA, Provedor OAuth/SSO externo, portais de recrutamento);
- Videochamadas, testes técnicos, controle de ponto ou certificação de competências.

## Índice da Documentação Viva

| Visão | Artefato | Caminho |
| :--- | :--- | :--- |
| Macroarquitetura | C4 Nível 1 — Contexto do Sistema | [`docs/architecture/c4-context.md`](docs/architecture/c4-context.md) |
| Macroarquitetura | C4 Nível 2 — Contêineres | [`docs/architecture/c4-containers.md`](docs/architecture/c4-containers.md) |
| Macroarquitetura | C4 Nível 3 — Componentes (`APIBackend`) | [`docs/architecture/c4-components.md`](docs/architecture/c4-components.md) |
| Funcional | Diagrama de Casos de Uso (v1) | [`docs/architecture/use-cases.md`](docs/architecture/use-cases.md) |
| Estrutural | Diagrama de Classes e Interfaces | [`docs/architecture/class-diagram.md`](docs/architecture/class-diagram.md) |
| Comportamental | Diagramas de Sequência Resilientes | [`docs/architecture/sequence-flows.md`](docs/architecture/sequence-flows.md) |
| Comportamental | Atividades com Raias e Máquina de Estados | [`docs/architecture/activities-states.md`](docs/architecture/activities-states.md) |
| Governança | ADR 0001 — Mensageria Assíncrona para Eventos | [`docs/adr/0001-mensageria-assincrona-eventos.md`](docs/adr/0001-mensageria-assincrona-eventos.md) |
| Governança | ADR 0002 — Integridade Transacional e Histórico | [`docs/adr/0002-integridade-transacional-historico.md`](docs/adr/0002-integridade-transacional-historico.md) |
