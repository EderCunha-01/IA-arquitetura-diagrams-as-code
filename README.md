# IA-arquitetura-diagrams-as-code
Estudo para elaborar diagramas arquiteturais em código, para uso de desenvolvimento com IA

## Caso de uso: consulta de faturas no aplicativo de ecare

Este é um **cenário ilustrativo**, não uma descrição da arquitetura interna de uma empresa. A proposta é permitir que um cliente autenticado consulte suas faturas pelo aplicativo.

### 1. Escopo

O sistema permite ao cliente:
- Consultar as faturas dos serviços vinculados à sua conta.
- Visualizar competência, vencimento, valor e situação de pagamento.
- Acessar o documento da fatura, quando disponível.

**Fora do escopo:** realizar pagamentos, negociar dívidas, contestar cobranças, alterar dados cadastrais ou calcular valores de faturamento.

### 2. Nível da visão

A descrição está no **nível de contexto do sistema**, equivalente ao C4 — Nível 1.

O foco é identificar quem utiliza o sistema, suas responsabilidades e os sistemas externos com os quais se comunica. Não são definidos componentes internos, bancos de dados ou tecnologias de implementação.

### 3. Limites e responsabilidades

O **Sistema de Consulta de Faturas** recebe solicitações do aplicativo, verifica se o cliente pode acessar a conta solicitada e obtém as informações de faturamento.

Ele é responsável por:
- Disponibilizar apenas as faturas às quais o cliente tem direito de acesso.
- Apresentar informações e mensagens de erro de maneira compreensível.
- Informar quando os dados estiverem indisponíveis ou tiverem atualização defasada.
- Registrar eventos operacionais e de auditoria sem expor dados pessoais desnecessariamente.

Ele **não é responsável** por emitir faturas, calcular cobranças ou confirmar pagamentos. Essas responsabilidades pertencem aos sistemas de faturamento e financeiros, considerados externos ao sistema em estudo.

O aplicativo é o canal de interação com o cliente; o provedor de identidade autentica o usuário. A autorização para consultar uma conta deve ser validada pelo sistema, não apenas pela autenticação.

### 4. Integrações

| Integração | Finalidade | Limite de responsabilidade |
|---|---|---|
| Aplicativo da Vivo | Solicitar e apresentar as faturas | Não é a fonte oficial dos dados de cobrança. |
| Provedor de identidade | Validar a identidade e a sessão do cliente | Autenticação não substitui a autorização de acesso à conta. |
| Cadastro de clientes e contas | Identificar os vínculos entre cliente, conta e serviços | As regras para titulares e representantes precisam ser confirmadas. |
| Sistema de faturamento | Fornecer valores, vencimentos e situação das faturas | É a fonte oficial; a consulta não altera seus registros. |
| Serviço de documentos | Disponibilizar o arquivo da fatura | O acesso ao documento também deve exigir autorização. |

As integrações são descritas por sua finalidade. **Protocolos, produtos e contratos de API ainda não estão definidos.**

### 5. Restrições e premissas

Para este exercício, considere as seguintes **hipóteses a validar**:
- O sistema de faturamento é legado e não pode ser substituído neste projeto.
- A consulta deve ser somente leitura e não prejudicar a operação do faturamento.
- Pode haver aumento de acessos próximo às datas de vencimento.
- A equipe tem capacidade limitada de desenvolvimento e sustentação.
- Dados de faturamento exigem controles de segurança, privacidade e acesso, considerando LGPD e políticas internas aplicáveis.
- Uma falha na integração não pode ser apresentada como “cliente sem faturas”.
- Caso seja permitido usar dados armazenados temporariamente, a defasagem deve ser informada e respeitar um limite aprovado pelo negócio.

### 6. Lacunas a esclarecer

Antes de propor uma arquitetura, seria necessário responder:

1. **Público e contas:** o serviço atende clientes pessoa física, jurídica ou ambos? Como funcionam contas com múltiplos serviços e representantes?
2. **Volume e desempenho:** quantas consultas são esperadas e qual é o tempo de resposta aceitável?
3. **Atualização:** quanto tempo uma confirmação de pagamento pode levar para aparecer?
4. **Legado:** existem APIs disponíveis? Quais são seus limites, horários de operação e garantias de disponibilidade?
5. **Histórico e documentos:** quantos meses de faturas serão exibidos e por quanto tempo os documentos ficam disponíveis?
6. **Operação e governança:** quem responde por incidentes, quais eventos precisam de auditoria e quais políticas internas se aplicam?

## Diagramas Mermaid

Abaixo estão duas versões mais apresentáveis, pensadas para leitura direta em README do GitHub.

### Visão de contexto

```mermaid
flowchart LR
    classDef cliente fill:#EAF3FF,stroke:#4F81BD,stroke-width:1.5px,color:#1F2A44;
    classDef sistema fill:#F2E8FF,stroke:#8A63D2,stroke-width:1.5px,color:#1F2A44;
    classDef externo fill:#F4F4F4,stroke:#666666,stroke-width:1.2px,color:#1F2A44;
    classDef rel fill:#FFF2CC,stroke:#D9A441,stroke-width:1px,color:#1F2A44;

    A[Aplicativo do cliente\nconsulta de faturas]:::cliente
    S[Sistema de Consulta de Faturas\n<contêiner>\nTecnologia a definir]:::sistema

    ID[Provedor de identidade\n<externo>]:::externo
    CT[Cadastro de clientes e contas\n<externo>]:::externo
    FAT[Faturamento\n<externo>\nValores, vencimentos e status]:::externo
    DOC[Serviço de documentos\n<externo>\nArquivos da fatura]:::externo

    A -->|Solicita faturas e acesso\na documentos autorizados| S
    S -->|Valida identidade e sessão| ID
    S -->|Verifica vínculo da conta| CT
    S -->|Consulta faturas e pagamentos| FAT
    S -->|Obtém documento autorizado| DOC

    ID -.->|autenticação| S
    CT -.->|autorização| S
    FAT -.->|dados oficiais| S
    DOC -.->|documentos| S
```

### Fluxo principal de consulta

```mermaid
sequenceDiagram
    autonumber
    actor Cliente as Cliente
    participant App as Aplicativo
    participant S as Sistema de Consulta de Faturas
    participant Id as Provedor de identidade
    participant C as Cadastro de contas e serviços
    participant F as Sistema de faturamento
    participant D as Serviço de documentos

    Cliente->>App: Acessa a tela de faturas
    App->>S: Solicita faturas da conta do cliente

    S->>Id: Valida identidade e sessão
    Id-->>S: Identidade confirmada

    alt Identidade inválida
        S-->>App: Retorna erro de autenticação
    else Identidade válida
        S->>C: Verifica acesso à conta e aos serviços
        C-->>S: Autorização da conta confirmada

        alt Acesso negado
            S-->>App: Informa que o cliente não tem acesso
        else Acesso permitido
            S->>F: Consulta faturas e status de pagamento
            F-->>S: Lista de faturas e detalhes

            alt Sistema indisponível
                S-->>App: Informa indisponibilidade temporária
            else Dados disponíveis
                S-->>App: Exibe faturas, vencimento e valores

                alt Documento disponível
                    S->>D: Solicita documento da fatura
                    D-->>S: Arquivo da fatura
                    S-->>App: Permite abrir o documento
                else Documento indisponível
                    S-->>App: Informa que o documento não está disponível
                end
            end
        end
    end
```

### Observação

Esses diagramas foram pensados para serem visualmente mais limpos e adaptados ao uso em README do GitHub, mantendo a mesma semântica do cenário de consulta de faturas.

- [Arquivo de contexto em Mermaid](Diagrama/consulta-faturas-contexto.mmd)
- [Arquivo de sequência em Mermaid](Diagrama/consulta-faturas-sequencia.mmd)
