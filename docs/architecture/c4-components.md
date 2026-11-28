# C4 Nível 3 — Diagrama de Componentes (`APIBackend`)

Decomposição interna do contêiner `APIBackend` demonstrando separação de responsabilidades (Controllers, Services de domínio e abstrações de infraestrutura via Injeção de Dependência).

```mermaid
C4Component
    title C4 Nivel 3 - Componentes do APIBackend (Equebis)

    Container(web, "WebApp", "SPA Web", "Interface dos tres perfis")
    ContainerDb(db, "BancoDados", "SGBD Relacional", "Persistencia transacional")
    Container(mq, "Mensageria", "Fila", "Eventos de dominio")

    Container_Boundary(api, "APIBackend") {
        Component(authCtrl, "AuthRBACMiddleware", "Middleware", "Autenticacao e controle de permissoes por perfil")
        Component(empCtrl, "EmpresaController", "REST Controller", "Endpoints de credenciamento e aprovacao de empresas")
        Component(vagaCtrl, "VagaController", "REST Controller", "Endpoints de publicacao, edicao, busca e suspensao de vagas")
        Component(candCtrl, "CandidaturaController", "REST Controller", "Endpoints de envio, acompanhamento e avaliacao de candidaturas")

        Component(empSvc, "EmpresaService", "Domain Service", "Regras de cadastro, aprovacao e suspensao de empresas")
        Component(vagaSvc, "VagaService", "Domain Service", "Regras de ciclo de vida da vaga e filtros por competencia")
        Component(candSvc, "CandidaturaService", "Domain Service", "Validacao atomica de vaga aberta, unicidade e historico de status")

        Component(repos, "Repositorios Relacionais", "IEmpresaRepository / IVagaRepository / ICandidaturaRepository", "Acesso transacional com timeout de 3s e lock")
        Component(pub, "EventoPublisher", "IEventoPublisher", "Publicacao assincrona de eventos de dominio na fila")
    }

    Rel(web, authCtrl, "Envia requisicoes autenticadas", "JSON/HTTPS")
    Rel(authCtrl, empCtrl, "Autoriza perfil Empresa/Coordenador")
    Rel(authCtrl, vagaCtrl, "Autoriza perfil Aluno/Empresa/Coordenador")
    Rel(authCtrl, candCtrl, "Autoriza perfil Aluno/Empresa")

    Rel(empCtrl, empSvc, "Delega operacoes de empresa")
    Rel(vagaCtrl, vagaSvc, "Delega operacoes de vaga")
    Rel(candCtrl, candSvc, "Delega operacoes de candidatura")

    Rel(empSvc, repos, "Persiste empresas via interface")
    Rel(vagaSvc, repos, "Persiste vagas via interface")
    Rel(candSvc, repos, "Valida vaga e grava candidatura + historico")

    Rel(empSvc, pub, "Dispara EmpresaCadastrada / EmpresaAvaliada")
    Rel(vagaSvc, pub, "Dispara VagaPublicada / VagaEncerrada")
    Rel(candSvc, pub, "Dispara CandidaturaEnviada / SituacaoAtualizada")

    Rel(repos, db, "Executa transacoes SQL", "SQL/TLS")
    Rel(pub, mq, "Enfileira eventos", "Async")
```
