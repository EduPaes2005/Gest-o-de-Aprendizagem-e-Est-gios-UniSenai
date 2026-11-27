# Diagramas de Sequência Resilientes — Visão Comportamental

Todos os participantes reproduzem exatamente a nomenclatura do Modelo C4 Nível 2 (`WebApp`, `APIBackend`, `BancoDados`, `Mensageria`, `WorkerNotificacao`) e invocam assinaturas existentes no Diagrama de Classes (`CandidaturaService`, `VagaService`, `EmpresaService`, `WorkerNotificacaoService`).

## 1. Fluxo de Candidatura com Engenharia do Caos (Timeout 3s, Conflito e DLQ)

```mermaid
sequenceDiagram
    autonumber
    actor Aluno
    participant WebApp
    participant APIBackend
    participant BancoDados
    participant Mensageria
    participant WorkerNotificacao

    Aluno->>WebApp: Seleciona vaga e clica em Candidatar-se
    WebApp->>APIBackend: POST /api/vagas/{id}/candidaturas [enviarCandidatura()]
    APIBackend->>BancoDados: BEGIN + buscarAbertaComLock(vagaId) & existeCandidatura() [timeout 3s]

    alt Banco <= 3s e vaga Aberta sem duplicidade
        BancoDados-->>APIBackend: salvarComHistorico() (status=ENVIADA) + COMMIT
        APIBackend-)Mensageria: publicarEvento("CandidaturaEnviada") (async)
        APIBackend-->>WebApp: HTTP 201 Created
        WebApp-->>Aluno: Exibe confirmacao de envio
        Mensageria->>WorkerNotificacao: Entrega evento [processarEvento()]
        alt WorkerNotificacao disponivel
            WorkerNotificacao->>BancoDados: salvarIdempotente(notificacao)
            WorkerNotificacao-->>Mensageria: ACK (processado)
        else WorkerNotificacao indisponivel
            WorkerNotificacao-->>Mensageria: NACK (retentativa com backoff / DLQ)
        end
    else Vaga Encerrada/Suspensa ou Candidatura Duplicada
        BancoDados-->>APIBackend: Violacao UNIQUE(aluno_id, vaga_id) ou status != ABERTA + ROLLBACK
        APIBackend-->>WebApp: HTTP 409 Conflict
        WebApp-->>Aluno: Exibe aviso de indisponibilidade ou duplicidade
    else Timeout > 3s no BancoDados
        BancoDados--xAPIBackend: TimeoutException (> 3000ms)
        APIBackend-->>WebApp: HTTP 503 Service Unavailable
        WebApp-->>Aluno: Orienta nova tentativa segura
    end
```

## 2. Fluxo de Credenciamento de Empresa e Publicação de Vaga

```mermaid
sequenceDiagram
    autonumber
    actor Empresa as Representante Empresa
    actor Coordenador as Coordenacao UniSenai
    participant WebApp
    participant APIBackend
    participant BancoDados
    participant Mensageria
    participant WorkerNotificacao

    Empresa->>WebApp: Envia cadastro da empresa (CNPJ, Razao Social)
    WebApp->>APIBackend: POST /api/empresas [cadastrarEmpresa()]
    APIBackend->>BancoDados: salvar(Empresa status=PENDENTE_APROVACAO)
    BancoDados-->>APIBackend: Empresa persistida
    APIBackend-)Mensageria: publicarEvento("EmpresaCadastrada") (async)
    APIBackend-->>WebApp: HTTP 201 Created (aguardando aprovacao)
    Mensageria->>WorkerNotificacao: processarEvento("EmpresaCadastrada")
    WorkerNotificacao->>BancoDados: salvarIdempotente(notificacao Coordenacao)

    Coordenador->>WebApp: Aprova cadastro da empresa
    WebApp->>APIBackend: PATCH /api/empresas/{id}/avaliar [avaliarCadastro(aprovada=true)]
    APIBackend->>BancoDados: UPDATE Empresa status=APROVADA
    BancoDados-->>APIBackend: COMMIT
    APIBackend-)Mensageria: publicarEvento("EmpresaAvaliada") (async)
    APIBackend-->>WebApp: HTTP 200 OK

    Empresa->>WebApp: Solicita publicacao de nova vaga
    WebApp->>APIBackend: POST /api/vagas [publicarVaga()]
    APIBackend->>BancoDados: buscarPorId(empresaId) + valida status == APROVADA [timeout 3s]

    alt Empresa APROVADA e Banco <= 3s
        APIBackend->>BancoDados: salvar(Vaga status=ABERTA, aberta=true) + COMMIT
        APIBackend-)Mensageria: publicarEvento("VagaPublicada") (async)
        APIBackend-->>WebApp: HTTP 201 Created
        Mensageria->>WorkerNotificacao: processarEvento("VagaPublicada")
        WorkerNotificacao->>BancoDados: Indexa competencias da vaga e grava alertas
    else Empresa Pendente ou Suspensa
        APIBackend-->>WebApp: HTTP 403 Forbidden (empresa sem permissao para publicar)
    else Timeout > 3s no BancoDados
        BancoDados--xAPIBackend: TimeoutException (> 3000ms)
        APIBackend-->>WebApp: HTTP 503 Service Unavailable
    end
```
