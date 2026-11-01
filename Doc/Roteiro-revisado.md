## 1) Lacunas e perguntas

Retomamos o escopo de **consulta de faturas**, desconsiderando o cenário de criação de pedidos. A restrição sobre provedores de pagamento será preservada, mas não justifica incluir integrações de pagamento, pois elas estão fora do escopo.

### Lacunas relevantes

1. **Fronteira do sistema:** não está definido se o aplicativo faz parte da solução ou é um canal externo.
2. **Identidade do cliente:** não foi indicada a origem da identidade autenticada utilizada nas consultas.
3. **Autorização:** faltam as regras que permitem ao cliente consultar determinada conta, serviço, fatura ou documento.
4. **Vínculos cadastrais:** não foi identificada a fonte oficial dos relacionamentos entre cliente, contas e serviços.
5. **Fontes de faturamento:** não se sabe se existe uma fonte consolidada ou múltiplas fontes por serviço.
6. **Documento da fatura:** faltam sua origem, as condições de disponibilidade e a forma autorizada de acesso.
7. **Atualidade e divergências:** não há critérios para atualização da situação de pagamento ou conflitos entre fontes.
8. **Falhas e resultados parciais:** faltam regras para fontes indisponíveis, ausência de faturas e documentos inacessíveis.
9. **Requisitos e implantação:** não foram definidos volume, latência, disponibilidade, privacidade, auditoria ou containers existentes.

### Perguntas de esclarecimento

1. O aplicativo pertence à fronteira da solução ou é externo?
2. Quem autentica o cliente e quais regras autorizam cada consulta?
3. Qual fonte oficial relaciona cliente, contas e serviços?
4. Quais fontes fornecem as faturas e a situação de pagamento?
5. Onde estão os documentos e como o cliente deve acessá-los?
6. Qual defasagem é aceitável e como tratar divergências ou respostas parciais?
7. Quais requisitos não funcionais e restrições de implantação são obrigatórios?

## 2) Roteiro revisado

1. **Escopo:** permitir ao cliente consultar faturas dos serviços vinculados à sua conta.
2. **Escopo:** apresentar competência, vencimento, valor e situação de pagamento, permitindo acesso ao documento quando disponível.
3. **Escopo:** excluir pagamentos, negociação de dívidas, contestação, alteração cadastral e cálculo de faturamento.
4. **Nível:** representar uma visão estrutural C4 — Containers, sem componentes internos, sequência de execução ou topologia de implantação.
5. **Limites:** delimitar a camada de consulta autorizada; a inclusão do aplicativo nessa fronteira permanece pendente.
6. **Responsabilidades:** verificar a autorização e consultar dados oficiais, sem assumir geração de faturas ou manutenção dos dados de origem.
7. **Integrações Externas:** identificar o canal, a origem da identidade, os vínculos cliente–conta–serviço e as fontes de faturas e documentos.
8. **Restrições:** marcar integrações externas com `<<external>>`, mostrar apenas dependências relevantes e não listar endpoints, rotas ou payloads.
9. **Restrições:** não expor detalhes internos dos sistemas externos nem incluir provedores de pagamento, desnecessários para este escopo.
10. **Lacunas:** validar fronteiras, autorização, fontes, acesso documental, atualidade, falhas e requisitos antes de decidir tecnologias, cache ou processamento assíncrono.

## 3) PlantUML

### Suposições explícitas da proposta

- O aplicativo será tratado provisoriamente como **canal externo**.
- A camada de consulta será representada por **um container de aplicação**. Isso não determina linguagem, framework ou arquitetura de microserviços.
- Identidade, vínculos, faturamento e documentos serão representados como **sistemas externos lógicos distintos**. Eles podem pertencer ao mesmo sistema real; essa separação precisa ser validada.
- A aplicação verificará o acesso às contas e aos serviços usando a identidade autenticada e os vínculos oficiais. As regras específicas ainda não foram definidas.
- O acesso ao documento será **intermediado pela aplicação**, sujeito à autorização. Um acesso direto por mecanismo protegido é uma alternativa ainda não avaliada.
- Não serão representados banco local, cache, filas ou cópias persistentes, pois sua necessidade não foi estabelecida.
- As setas representam dependências estruturais, não ordem de execução, protocolo ou fluxo completo de autenticação.
- A situação de pagamento será obtida da fonte oficial de faturamento, **sem integração direta com um provedor de pagamentos**.

O código abaixo usa a biblioteca C4-PlantUML. Para reprodução estável em produção, recomenda-se versionar a biblioteca no repositório em vez de depender da referência remota.

```plantuml
@startuml
!includeurl https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

title Consulta de Faturas — C4 Containers\nProposta inicial sujeita a validação

LAYOUT_LEFT_RIGHT()

AddElementTag("external",
  $bgColor="#F2F2F2",
  $fontColor="#222222",
  $borderColor="#777777",
  $legendText="Sistema externo à camada de consulta")

System_Ext(canal,
  "Aplicativo do cliente <<external>>",
  "Canal de consulta e visualização. Estrutura interna omitida.",
  $tags="external")

System_Ext(identidade,
  "Sistema de Identidade <<external>>",
  "Origem da identidade autenticada. Mecanismo de integração a confirmar.",
  $tags="external")

System_Ext(vinculos,
  "Fonte de Contas e Serviços <<external>>",
  "Fornece os vínculos oficiais entre cliente, contas e serviços.",
  $tags="external")

System_Ext(faturamento,
  "Fonte Oficial de Faturamento <<external>>",
  "Fornece faturas, competência, vencimento, valor e situação de pagamento.",
  $tags="external")

System_Ext(documentos,
  "Fonte de Documentos de Fatura <<external>>",
  "Fornece documentos quando disponíveis. Forma de acesso a confirmar.",
  $tags="external")

System_Boundary(consulta, "Sistema de Consulta de Faturas") {
  Container(aplicacao,
    "Aplicação de Consulta de Faturas",
    "Tecnologia a definir",
    "Verifica autorização, consulta dados oficiais e disponibiliza faturas e documentos ao canal.")
}

Rel(canal, aplicacao,
  "Consulta faturas e acessa documentos autorizados")

Rel(aplicacao, identidade,
  "Valida a identidade apresentada pelo canal")

Rel(aplicacao, vinculos,
  "Consulta vínculos para verificar acesso à conta e aos serviços")

Rel(aplicacao, faturamento,
  "Consulta faturas e situação de pagamento")

Rel(aplicacao, documentos,
  "Obtém documentos para acesso autorizado")

SHOW_LEGEND()

@enduml
```

**Limite de interpretação:** essa proposta não resolve autenticação, regras de autorização, acesso direto a documentos ou comportamento diante de falhas. Também não pressupõe que consultar a situação de pagamento implique realizar pagamentos.

## 4) Checklist para revisão em pull request

- [ ] O diagrama contempla somente consulta de faturas e acesso aos documentos.
- [ ] A fronteira da solução e a classificação do aplicativo foram confirmadas.
- [ ] Os elementos internos são containers; não há componentes, classes ou nós de implantação.
- [ ] Todas as integrações externas estão identificadas com `<<external>>`, sem decomposição interna.
- [ ] As responsabilidades de consulta estão separadas das responsabilidades de produzir e manter dados oficiais.
- [ ] As dependências de identidade e vínculos sustentam as regras de autorização definidas.
- [ ] O acesso ao documento preserva a autorização e não pressupõe acesso público.
- [ ] Não há integração de pagamento nem exposição de detalhes de seu provedor.
- [ ] As relações mostram somente dependências relevantes, sem endpoints, payloads ou sequência de execução.
- [ ] As fontes lógicas representadas foram conciliadas com os sistemas reais, evitando duplicações artificiais.
- [ ] Suposições e lacunas estão registradas, sem introduzir cache, persistência ou mensageria sem justificativa.
- [ ] O PlantUML renderiza corretamente, e a versão da biblioteca C4 está fixada para reprodução no repositório.
