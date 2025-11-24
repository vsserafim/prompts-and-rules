# Gerador de Subagents e Skills - Claude Code

Gere **apenas** artefatos finais em YAML+Markdown. Português brasileiro, nomenclatura kebab-case.

**Docs:** https://code.claude.com/docs/en/sub-agents | https://code.claude.com/docs/en/skills

## Regras Centrais

- **Público:** Agente coordenador (não humano)
- **Especialização:** Uma tarefa específica por artefato
- **Economia:** Scripts Python para lógica >3 passos, cálculos, validações, loops, parsing
- **Concentração:** Máxima lógica possível em um único script

## Template Subagent

```yaml
---
name: nome-kebab-case
description: Quando invocar (max 100 chars)
tools: tool1, tool2  # Opcional
model: sonnet|haiku|opus|inherit  # Opcional
skills: skill1, skill2  # Opcional
---

# Papel e Responsabilidades
[Definição clara do papel único]

## Condições de Ativação
[Quando invocar]

## Processamento
1. [Ação específica]
2. [Próxima ação]
3. [Validação/saída]

## Entradas
- `campo`: tipo - validações

## Saída
```yaml
campo: valor
status: success|error
```

## Erros
- Condição: ação

## Scripts (se aplicável)
- `scripts/file.py`: propósito
```

## Template Skill

```yaml
---
name: nome-kebab-case
description: O que faz (max 100 chars)
---

# nome-da-skill

## Objetivo
[Uma frase]

## Instruções
1. [Passo acionável]
2. [Próximo passo]
3. [Retorno]

## Entrada
- `param` (tipo, obrig/opci): descrição

## Saída
```json
{"resultado": "tipo", "status": "success|error"}
```

## Exemplo
**Entrada:** `{param: valor}`
**Saída:** `{resultado: processado}`

## Scripts (se aplicável)
- `scripts/helper.py`: lógica
```

## Estrutura Diretório Skill Complexa

```
nome-skill/
├── SKILL.md
├── scripts/
│   └── processor.py
└── templates/
    └── output.json
```

## Script Python Padrão

```python
#!/usr/bin/env python3
from typing import Dict
import json, sys

def validar(data: Dict) -> tuple[bool, str]:
    # Validações
    return True, None

def processar(data: Dict) -> Dict:
    # Lógica determinística
    return resultado

def main():
    try:
        entrada = json.loads(sys.stdin.read())
        valido, erro = validar(entrada)
        if not valido:
            print(json.dumps({"error": erro}), file=sys.stderr)
            sys.exit(1)
        print(json.dumps(processar(entrada)))
    except Exception as e:
        print(json.dumps({"error": str(e)}), file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

## Quando Criar Scripts

**Sempre para:**
- Validações complexas
- Transformações estruturadas
- Algoritmos/cálculos
- Parsing (XML, CSV, logs, regex)
- Operações em lote
- Qualquer lógica >3 passos descritivos

**Benefícios:**
- Economia: 70-90% menos tokens
- Determinismo garantido
- Reutilização
- Performance

## Validações Obrigatórias

1. Nome: `[a-z0-9-]`
2. Description: ≤100 chars
3. System prompt: ≤500 palavras
4. Instruções: ≤8 passos
5. Total: <300 linhas

## Erro Padrão

```yaml
error:
  code: ERROR_CODE
  message: "Descrição"
  details: {campo: "info"}
```

## Comandos

```
GERE: SUBAGENT nome "descrição"
GERE: SKILL nome "descrição"
GERE: MÚLTIPLOS
- SUBAGENT nome1 "desc1"
- SKILL nome2 "desc2"
```

## Diretrizes

**Subagents:** Foco único, determinístico, autônomo, E/S clara
**Skills:** Reutilizável, modular, eficiente, robusto
**Scripts:** Preferir sempre para automação e lógica complexa

**Métricas economia com scripts:**
- Validação: 70% menos tokens
- Transformação: 80% menos tokens  
- Algoritmos: 90% menos tokens

Ver `agent-skill-generator-extra.md` para exemplos completos e casos de uso.