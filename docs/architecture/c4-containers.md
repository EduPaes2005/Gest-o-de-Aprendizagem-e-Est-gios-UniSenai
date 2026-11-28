# C4 Nível 2 — Diagrama de Contêineres

Topologia resiliente após auditoria de SPOFs e acoplamento temporal (Sprint 4), introduzindo `Mensageria` e `WorkerNotificacao` para isolar notificações internas e indexação de competências da resposta HTTP síncrona.

```mermaid
C4Container
    title C4 Nivel 2 - Gestao de Aprendizagem e Estagios UniSenai (Equebis)

    Person(aluno, "Aluno UniSenai", "Gerencia perfil, filtra e candidata-se")
    Person(empresa, "Empresa Parceira", "Publica vagas e avalia candidatos")
    Person(coord, "Coordenacao UniSenai", "Aprova empresas e fiscaliza vagas")

    System_Boundary(sys, "Sistema de Estagios UniSenai") {
        Container(web, "WebApp", "SPA Web", "Interface por perfil de acesso")
        Container(api, "APIBackend", "API REST", "RBAC e regras de negocio")
        Container(mq, "Mensageria", "Fila", "Eventos de vaga, empresa e candidatura")
        Container(wk, "WorkerNotificacao", "Worker", "Notificacoes internas e indexacao")
        ContainerDb(db, "BancoDados", "SGBD Relacional", "Perfis, vagas, candidaturas e historico")
    }

    Rel(aluno, web, "Acessa plataforma", "HTTPS")
    Rel(empresa, web, "Acessa plataforma", "HTTPS")
    Rel(coord, web, "Acessa plataforma", "HTTPS")
    Rel(web, api, "Consome endpoints REST", "JSON/HTTPS")
    Rel(api, db, "Le e grava entidades com timeout de 3s", "SQL/TLS")
    Rel(api, mq, "Publica eventos de dominio", "Async")
    Rel(wk, mq, "Consome eventos com retentativa e DLQ", "Async")
    Rel(wk, db, "Persiste notificacoes idempotentes e indices", "SQL/TLS")
```
