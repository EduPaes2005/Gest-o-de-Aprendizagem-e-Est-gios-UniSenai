# C4 Nível 1 — Diagrama de Contexto do Sistema

Visão macro do **Sistema Web de Gestão de Aprendizagem e Estágios da UniSenai**, alinhada à Declaração de Escopo da Sprint 2 e à Auditoria Cross-Diagrams da Sprint 4 (sem integrações externas com SGA ou OAuth/SSO de terceiros).

```mermaid
C4Context
    title C4 Nivel 1 - Contexto do Sistema de Estagios UniSenai (Equebis)

    Person(aluno, "Aluno UniSenai", "Cadastra perfil e competencias, filtra vagas, envia candidaturas e acompanha historico")
    Person(empresa, "Empresa Parceira", "Cadastra empresa, publica/edita/encerra vagas e avalia candidatos")
    Person(coord, "Coordenacao UniSenai", "Aprova ou suspende empresas, fiscaliza vagas e consulta indicadores")

    System(sys, "Sistema de Estagios UniSenai", "Plataforma web que centraliza perfis, empresas parceiras, vagas e fluxo de candidaturas com notificacoes internas")

    Rel(aluno, sys, "Gerencia perfil, pesquisa vagas e candidata-se", "HTTPS")
    Rel(empresa, sys, "Publica vagas e atualiza situacao de candidaturas", "HTTPS")
    Rel(coord, sys, "Credencia empresas e monitora conformidade de vagas", "HTTPS")
```
