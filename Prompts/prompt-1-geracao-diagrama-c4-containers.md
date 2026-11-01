# Prompt 1 — Geração de Diagrama C4 (Containers)

Você é um arquiteto de software sênior. Antes de gerar qualquer diagrama, identifique lacunas e faça perguntas de esclarecimento.

## Descrição (linguagem natural)

### Objetivo do diagrama
Tipo: estrutural  
Nível: C4 — Containers  
Escopo: fluxo completo do disparo proativo, desde o upload do arquivo até a entrega no canal e possível interação com o Digital Assistant Engine.

### Restrições
- Marcar integrações externas como `<<external>>`.
- Não listar endpoints, rotas ou payloads.
- Não misturar níveis do C4.
- Mostrar apenas dependências relevantes.
- Não expor detalhes internos dos canais externos.

### Entregue
1. Liste 5–10 lacunas ou ambiguidades relevantes.
2. Liste 5–8 perguntas de esclarecimento (curtas e objetivas).
3. Reescreva a descrição como um roteiro revisado (8–12 linhas), incorporando:
   - Escopo
   - Nível
   - Limites e responsabilidades
   - Integrações externas
   - Restrições
   - Lacunas
   - Regras
4. Se faltar contexto, explicite; não assuma fatos.
5. Não gere o diagrama nesta etapa.
6. O PlantUML será solicitado em um próximo prompt.

### Formato
Markdown estruturado.

### Saída esperada
Um roteiro que caiba nos elementos de Escopo, Nível, Limites, Integrações, Restrições, Lacunas.
