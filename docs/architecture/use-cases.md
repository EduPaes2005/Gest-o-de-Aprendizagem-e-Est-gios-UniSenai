# Diagrama de Casos de Uso — Visão Funcional (Escopo v1)

Casos de uso consolidados a partir da Declaração de Escopo da Sprint 2 e revisados na auditoria da Sprint 4 (mantendo exclusivamente os três perfis da v1, filtros manuais por competência/modalidade e notificações internas).

```mermaid
flowchart LR
    Aluno(("Aluno UniSenai"))
    Empresa(("Representante da Empresa"))
    Coord(("Coordenacao UniSenai"))

    subgraph Sistema["Sistema de Gestao de Aprendizagem e Estagios UniSenai"]
        subgraph AreaEmpresa["Area da Empresa"]
            UC_E1(["Cadastrar empresa"])
            UC_E2(["Atualizar perfil da empresa"])
            UC_E3(["Publicar vaga"])
            UC_E4(["Atualizar vaga"])
            UC_E5(["Encerrar vaga"])
            UC_E6(["Consultar candidaturas"])
            UC_E7(["Analisar candidato"])
            UC_E8(["Consultar perfil do candidato"])
            UC_E9(["Atualizar situacao da candidatura"])
        end

        subgraph AreaCoord["Area da Coordenacao"]
            UC_C1(["Consultar empresas"])
            UC_C2(["Avaliar cadastro de empresa"])
            UC_C3(["Suspender empresa"])
            UC_C4(["Gerenciar catalogo de competencias"])
            UC_C5(["Consultar oportunidades"])
            UC_C6(["Suspender vaga"])
            UC_C7(["Consultar indicadores"])
        end

        subgraph AcessoNotif["Acesso e Notificacoes"]
            UC_N1(["Autenticar-se"])
            UC_N2(["Gerar notificacao interna"])
            UC_N3(["Consultar notificacoes"])
        end

        subgraph AreaAluno["Area do Aluno"]
            UC_A1(["Cadastrar conta"])
            UC_A2(["Consultar proprio perfil"])
            UC_A3(["Atualizar perfil e curriculo"])
            UC_A4(["Informar competencias"])
            UC_A5(["Pesquisar vagas"])
            UC_A6(["Filtrar vagas"])
            UC_A7(["Visualizar detalhes da vaga"])
            UC_A8(["Consultar perfil da empresa"])
            UC_A9(["Candidatar-se a vaga"])
            UC_A10(["Acompanhar candidatura"])
            UC_A11(["Consultar historico da candidatura"])
            UC_A12(["Cancelar candidatura"])
        end
    end

    %% Associacoes Empresa
    Empresa --> UC_E1
    Empresa --> UC_E2
    Empresa --> UC_E3
    Empresa --> UC_E4
    Empresa --> UC_E5
    Empresa --> UC_E6
    Empresa --> UC_E7
    Empresa --> UC_E9
    Empresa --> UC_N1
    Empresa --> UC_N3

    %% Associacoes Coordenacao
    Coord --> UC_C1
    Coord --> UC_C2
    Coord --> UC_C3
    Coord --> UC_C4
    Coord --> UC_C5
    Coord --> UC_C6
    Coord --> UC_C7
    Coord --> UC_N1
    Coord --> UC_N3

    %% Associacoes Aluno
    Aluno --> UC_A1
    Aluno --> UC_A2
    Aluno --> UC_A3
    Aluno --> UC_A4
    Aluno --> UC_A5
    Aluno --> UC_A7
    Aluno --> UC_A8
    Aluno --> UC_A9
    Aluno --> UC_A10
    Aluno --> UC_A11
    Aluno --> UC_A12
    Aluno --> UC_N1
    Aluno --> UC_N3

    %% Relacionamentos <<include>> e <<extend>>
    UC_E7 -. "<<include>>" .-> UC_E8
    UC_E9 -. "<<include>>" .-> UC_N2
    UC_C2 -. "<<include>>" .-> UC_N2
    UC_A9 -. "<<include>>" .-> UC_N2
    UC_A6 -. "<<extend>>" .-> UC_A5
```
