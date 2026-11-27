# Diagramas de Atividades com Raias e Máquinas de Estados

Artefatos comportamentais atualizados para explicitar nos blocos *Fork/Join* a atuação coordenada entre `APIBackend`, `BancoDados`, `Mensageria` e `WorkerNotificacao`, mantendo total aderência aos estados das entidades no Diagrama de Classes.

## 1. Atividades — Fluxo de Candidatura e Avaliação (Com Raias e Fork/Join)

```mermaid
flowchart TD
    subgraph Aluno ["Raia: Aluno UniSenai (WebApp)"]
        A1(["Inicio"]) --> A2["Cadastrar/Atualizar perfil e competencias"]
        A2 --> A3["Buscar vagas com filtros de competencia e modalidade"]
        A3 --> A4["Selecionar vaga e Enviar Candidatura"]
        A9["Receber confirmacao e Acompanhar historico"]
        A10(["Fim - Candidatura Recusada/Cancelada"])
        A11(["Fim - Processo Concluido"])
    end

    subgraph Sistema ["Raia: APIBackend + Mensageria + WorkerNotificacao"]
        S1{"Vaga esta Aberta e sem duplicidade?"}
        S2["Retornar HTTP 409 e notificar recusa"]

        Fork1[["Fork: Persistencia + Evento Assincrono"]]
        S3["APIBackend grava Candidatura + Historico no BancoDados (ENVIADA)"]
        S4["Mensageria aciona WorkerNotificacao para gerar alerta interno"]
        Join1[["Join: Sincronizacao"]]

        S5["APIBackend atualiza Situacao e grava Historico no BancoDados"]
        S6["WorkerNotificacao publica atualizacao e notifica Aluno"]
    end

    subgraph Empresa ["Raia: Representante da Empresa (WebApp)"]
        E1["Consultar candidaturas da vaga"]
        E2["Analisar perfil e competencias do candidato"]
        E3{"Decisao da Avaliacao"}
        E4["Registrar aprovacao para etapas externas"]
        E5["Registrar reprovacao na vaga"]
    end

    A4 --> S1
    S1 -- "Nao" --> S2
    S2 --> A10

    S1 -- "Sim" --> Fork1
    Fork1 --> S3
    Fork1 --> S4
    S3 --> Join1
    S4 --> Join1

    Join1 --> A9
    Join1 --> E1

    E1 --> E2
    E2 --> E3
    E3 -- "Aprovada" --> E4
    E3 -- "Reprovada" --> E5

    E4 --> S5
    E5 --> S5
    S5 --> S6
    S6 --> A9
    A9 --> A11
```

## 2. Atividades — Gestão e Publicação de Vaga

```mermaid
flowchart TD
    subgraph Empresa ["Raia: Representante da Empresa (WebApp)"]
        E1(["Inicio"]) --> E2["Preencher dados da Vaga, Modalidade e Competencias"]
        E2 --> E3["Solicitar Publicacao da Vaga"]
        E4["Gerenciar Vaga / Receber Candidaturas"]
        E5["Solicitar Encerramento da Vaga"]
    end

    subgraph Sistema ["Raia: APIBackend + Mensageria + WorkerNotificacao"]
        S1{"Empresa esta Aprovada pela Coordenacao?"}
        S2["Bloquear publicacao (HTTP 403) e exibir erro"]

        Fork1[["Fork: Publicacao Concorrente"]]
        S3["APIBackend grava Vaga no BancoDados (ABERTA, aberta=true)"]
        S4["WorkerNotificacao indexa competencias via Mensageria"]
        Join1[["Join"]]

        S5["APIBackend atualiza Vaga (aberta=false)"]
        S6(["Fim do Fluxo"])
    end

    subgraph Coordenacao ["Raia: Coordenacao UniSenai (WebApp)"]
        C1["Consultar oportunidades publicadas"]
        C2{"Vaga em conformidade?"}
        C3["Manter vaga ativa"]
        C4["Suspender Vaga"]
    end

    E3 --> S1
    S1 -- "Nao" --> S2
    S2 --> S6

    S1 -- "Sim" --> Fork1
    Fork1 --> S3
    Fork1 --> S4
    S3 --> Join1
    S4 --> Join1

    Join1 --> E4
    Join1 --> C1

    C1 --> C2
    C2 -- "Sim" --> C3
    C2 -- "Nao" --> C4

    E4 --> E5
    E5 --> S5
    C4 --> S5
    S5 --> S6
```

## 3. Atividades — Credenciamento de Empresa

```mermaid
flowchart TD
    subgraph Empresa ["Raia: Representante da Empresa (WebApp)"]
        E1(["Inicio"]) --> E2["Realizar cadastro da Empresa e CNPJ"]
        E3["Acesso liberado para publicar vagas"]
        E4(["Fim - Cadastro Recusado"])
    end

    subgraph Sistema ["Raia: APIBackend + Mensageria + WorkerNotificacao"]
        S1["APIBackend valida dados e cria Empresa (PENDENTE_APROVACAO)"]
        Fork1[["Fork"]]
        S2["Registrar solicitacao no BancoDados para a Coordenacao"]
        S3["WorkerNotificacao gera alerta interno via Mensageria"]
        Join1[["Join"]]
        S4["Liberar permissoes de Empresa Parceira (APROVADA)"]
        S5["Bloquear acesso (RECUSADA) e notificar motivo"]
    end

    subgraph Coordenacao ["Raia: Coordenacao UniSenai (WebApp)"]
        C1["Consultar empresas pendentes"]
        C2["Avaliar cadastro da empresa"]
        C3{"Aprovar Empresa?"}
    end

    E2 --> S1
    S1 --> Fork1
    Fork1 --> S2
    Fork1 --> S3
    S2 --> Join1
    S3 --> Join1

    Join1 --> C1
    C1 --> C2
    C2 --> C3

    C3 -- "Aprovada" --> S4
    C3 -- "Recusada" --> S5

    S4 --> E3
    S5 --> E4
```

## 4. Máquina de Estados — Ciclo de Vida da Candidatura

```mermaid
stateDiagram-v2
    [*] --> Enviada : Aluno envia candidatura [Vaga Aberta == true e sem duplicidade]

    state Enviada {
        [*] --> AguardandoVisualizacao
        AguardandoVisualizacao : entry / registrar DataEnvio e HistoricoStatusCandidatura
        AguardandoVisualizacao : do / aguardar consulta da empresa
        AguardandoVisualizacao : exit / gerar notificacao de leitura
    }

    Enviada --> EmAnalise : Empresa consulta candidatura / Consultar perfil do candidato
    Enviada --> Cancelada : Aluno solicita cancelamento [Antes da decisao]

    state EmAnalise {
        [*] --> AvaliandoCompetencias
        AvaliandoCompetencias : entry / exibir curriculo e competencias
        AvaliandoCompetencias : do / analisar aderencia a vaga
    }

    EmAnalise --> Cancelada : Aluno solicita cancelamento
    EmAnalise --> EncerradaPorVaga : Empresa/Coordenador encerra ou suspende a vaga
    EmAnalise --> Aprovada : Empresa atualiza situacao [Candidato Apto]
    EmAnalise --> Reprovada : Empresa atualiza situacao [Perfil Incompativel]

    Aprovada : entry / gravar historico e notificar Aluno para etapas externas
    Reprovada : entry / gravar historico e notificar Aluno sobre encerramento
    Cancelada : entry / registrar desistencia no historico
    EncerradaPorVaga : entry / notificar encerramento da oportunidade

    Aprovada --> [*]
    Reprovada --> [*]
    Cancelada --> [*]
    EncerradaPorVaga --> [*]
```

## 5. Máquina de Estados — Ciclo de Vida da Vaga

```mermaid
stateDiagram-v2
    [*] --> Rascunho : Empresa inicia cadastro da vaga

    Rascunho --> Aberta : Empresa publica vaga [Empresa Ativa == true]

    state Aberta {
        [*] --> RecebendoCandidaturas
        RecebendoCandidaturas : entry / definir aberta = true e publicar VagaPublicada
        RecebendoCandidaturas : do / exibir nas buscas e aceitar candidaturas
        RecebendoCandidaturas --> EmEdicao : Empresa atualiza descricao/competencias
        EmEdicao --> RecebendoCandidaturas : Salvar alteracoes
    }

    Aberta --> Suspensa : Coordenador suspende vaga [Irregularidade identificada]
    Suspensa : entry / definir aberta = false e ocultar das buscas
    Suspensa --> Aberta : Coordenador libera vaga apos ajuste
    Suspensa --> Encerrada : Empresa ou Coordenacao cancela definitivamente

    Aberta --> Encerrada : Empresa encerra vaga [Processo seletivo concluido]
    Encerrada : entry / definir aberta = false e bloquear novas candidaturas

    Encerrada --> [*]
```

## 6. Máquina de Estados — Ciclo de Vida da Empresa

```mermaid
stateDiagram-v2
    [*] --> PendenteAprovacao : Representante cadastra empresa (CNPJ, Razao Social)

    state PendenteAprovacao {
        [*] --> AguardandoCoordenacao
        AguardandoCoordenacao : entry / restringir publicacao de vagas
        AguardandoCoordenacao : do / aguardar avaliacao do Coordenador
    }

    PendenteAprovacao --> Aprovada : Coordenador aprova cadastro
    PendenteAprovacao --> Recusada : Coordenador reprova cadastro [Dados invalidos]

    state Aprovada {
        [*] --> Ativa
        Ativa : entry / liberar acesso ao modulo de vagas
        Ativa : do / permitir publicacao de vagas e analise de candidatos
    }

    Aprovada --> Suspensa : Coordenador suspende empresa
    Suspensa : entry / bloquear novas vagas e suspender vagas ativas
    Suspensa --> Aprovada : Coordenador reativa empresa

    Recusada --> [*]
    Suspensa --> [*] : Encerramento definitivo de parceria
```
