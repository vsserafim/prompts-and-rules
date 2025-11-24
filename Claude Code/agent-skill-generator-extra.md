# Referência Detalhada - Subagents e Skills

Complemento ao prompt principal com exemplos completos e casos de uso.

## Exemplo 1: Subagent com Script Python

```yaml
---
name: analisador-logs
description: Analisa logs de aplicação e extrai métricas de performance
model: haiku
skills: parse-log-entry
---

Analisar logs de aplicação para métricas de performance.

## Entrada
`log_file`: string - Caminho do arquivo de log
`periodo`: object - {inicio: datetime, fim: datetime}

## Processamento
1. Executar `scripts/log_analyzer.py` com parâmetros
2. Validar métricas extraídas
3. Agregar estatísticas por endpoint
4. Retornar relatório estruturado

## Scripts Utilizados
- `scripts/log_analyzer.py`: Parser e análise de logs
- `scripts/metrics_aggregator.py`: Agregação de métricas

## Saída
```yaml
metricas:
  requisicoes_total: integer
  tempo_medio_ms: float
  taxa_erro: float
  endpoints_lentos: array
periodo_analisado: string
```
```

### Script Completo (scripts/log_analyzer.py)

```python
#!/usr/bin/env python3
import re
import json
from datetime import datetime
from collections import defaultdict

def parse_log_line(line):
    pattern = r'(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) \[(\w+)\] (\w+) (\/\S+) (\d+)ms status:(\d+)'
    match = re.match(pattern, line)
    if match:
        return {
            'timestamp': match.group(1),
            'level': match.group(2),
            'method': match.group(3),
            'endpoint': match.group(4),
            'duration_ms': int(match.group(5)),
            'status': int(match.group(6))
        }
    return None

def analyze_logs(log_path, start_time, end_time):
    metrics = defaultdict(list)
    errors = 0
    total = 0
    
    with open(log_path, 'r') as f:
        for line in f:
            entry = parse_log_line(line)
            if entry:
                total += 1
                metrics[entry['endpoint']].append(entry['duration_ms'])
                if entry['status'] >= 400:
                    errors += 1
    
    return {
        'requisicoes_total': total,
        'tempo_medio_ms': sum(sum(v) for v in metrics.values()) / total if total > 0 else 0,
        'taxa_erro': errors / total if total > 0 else 0,
        'endpoints_lentos': [ep for ep, times in metrics.items() if sum(times)/len(times) > 1000]
    }

if __name__ == '__main__':
    import sys
    params = json.loads(sys.stdin.read())
    result = analyze_logs(
        params['log_file'],
        params['periodo']['inicio'],
        params['periodo']['fim']
    )
    print(json.dumps(result))
```

## Exemplo 2: Skill com Validação via Script

```yaml
---
name: validar-documento
description: Valida documentos brasileiros usando script Python
---

# validar-documento

## Instruções
1. Identificar tipo de documento (CPF, CNPJ, CNH)
2. Executar script `scripts/doc_validator.py`
3. Retornar validação com detalhes
4. Em caso de erro, incluir correções sugeridas

## Parâmetros
- `documento`: string (obrigatório)
- `tipo`: string (opcional, auto-detecta se omitido)

## Saída
```json
{
  "valido": true,
  "tipo": "CPF",
  "formatado": "123.456.789-09",
  "detalhes": "Documento válido"
}
```

## Scripts
- `scripts/doc_validator.py`: Validação de CPF/CNPJ
```

### Script Completo (scripts/doc_validator.py)

```python
#!/usr/bin/env python3
import re
import json
import sys

def validar_cpf(cpf):
    cpf = re.sub(r'[^0-9]', '', cpf)
    if len(cpf) != 11:
        return False, "CPF deve ter 11 dígitos"
    
    if cpf == cpf[0] * 11:
        return False, "CPF com dígitos repetidos"
    
    # Primeiro dígito verificador
    soma = sum(int(cpf[i]) * (10 - i) for i in range(9))
    digito1 = 11 - (soma % 11)
    digito1 = 0 if digito1 > 9 else digito1
    
    if int(cpf[9]) != digito1:
        return False, "Primeiro dígito verificador inválido"
    
    # Segundo dígito verificador
    soma = sum(int(cpf[i]) * (11 - i) for i in range(10))
    digito2 = 11 - (soma % 11)
    digito2 = 0 if digito2 > 9 else digito2
    
    if int(cpf[10]) != digito2:
        return False, "Segundo dígito verificador inválido"
    
    return True, f"{cpf[:3]}.{cpf[3:6]}.{cpf[6:9]}-{cpf[9:]}"

def validar_cnpj(cnpj):
    cnpj = re.sub(r'[^0-9]', '', cnpj)
    if len(cnpj) != 14:
        return False, "CNPJ deve ter 14 dígitos"
    
    # Validação dos dígitos verificadores
    pesos1 = [5,4,3,2,9,8,7,6,5,4,3,2]
    pesos2 = [6,5,4,3,2,9,8,7,6,5,4,3,2]
    
    soma = sum(int(cnpj[i]) * pesos1[i] for i in range(12))
    digito1 = 11 - (soma % 11)
    digito1 = 0 if digito1 > 9 else digito1
    
    if int(cnpj[12]) != digito1:
        return False, "Primeiro dígito verificador inválido"
    
    soma = sum(int(cnpj[i]) * pesos2[i] for i in range(13))
    digito2 = 11 - (soma % 11)
    digito2 = 0 if digito2 > 9 else digito2
    
    if int(cnpj[13]) != digito2:
        return False, "Segundo dígito verificador inválido"
    
    return True, f"{cnpj[:2]}.{cnpj[2:5]}.{cnpj[5:8]}/{cnpj[8:12]}-{cnpj[12:]}"

def main():
    try:
        entrada = json.loads(sys.stdin.read())
        doc = entrada.get('documento', '')
        tipo = entrada.get('tipo', '').upper()
        
        # Auto-detectar tipo
        doc_limpo = re.sub(r'[^0-9]', '', doc)
        if not tipo:
            tipo = 'CPF' if len(doc_limpo) == 11 else 'CNPJ' if len(doc_limpo) == 14 else 'DESCONHECIDO'
        
        if tipo == 'CPF':
            valido, resultado = validar_cpf(doc)
        elif tipo == 'CNPJ':
            valido, resultado = validar_cnpj(doc)
        else:
            valido, resultado = False, "Tipo de documento não suportado"
        
        saida = {
            'valido': valido,
            'tipo': tipo,
            'formatado': resultado if valido else None,
            'detalhes': 'Documento válido' if valido else resultado
        }
        
        print(json.dumps(saida, ensure_ascii=False))
        
    except Exception as e:
        print(json.dumps({'error': str(e)}), file=sys.stderr)
        sys.exit(1)

if __name__ == '__main__':
    main()
```

## Exemplo 3: Subagent Minimalista (sem script)

```yaml
---
name: converter-temperatura
description: Converte temperatura entre Celsius, Fahrenheit e Kelvin
model: haiku
---

Converter valor de temperatura entre escalas.

## Entrada
`valor`: number - Temperatura a converter
`de`: string - Escala origem (C, F, K)
`para`: string - Escala destino (C, F, K)

## Processamento
1. Validar escalas suportadas
2. Aplicar fórmula de conversão
3. Arredondar para 2 decimais
4. Retornar resultado

## Saída
```yaml
temperatura: float
escala: string
formula_usada: string
```

## Erros
- Escala inválida: retornar código INVALID_SCALE
- Valor Kelvin negativo: retornar INVALID_VALUE
```

## Exemplo 4: Skill Minimalista (sem script)

```yaml
---
name: extrair-email
description: Extrai endereços de email de texto usando regex
---

# extrair-email

## Instruções
1. Aplicar regex de email padrão RFC 5322
2. Remover duplicatas
3. Validar formato básico
4. Retornar lista ordenada

## Entrada
- `texto`: string (obrigatório) - Texto para análise
- `unique`: boolean (opcional, default: true) - Remover duplicatas

## Saída
```json
{
  "emails": ["email1@domain.com", "email2@domain.com"],
  "total": 2
}
```

## Regex Usado
`[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`
```

## Padrões de Integração Script-Skill

### Invocação via stdin/stdout

```yaml
## Uso do Script
```bash
echo '{"param": "valor"}' | python scripts/processor.py
```
```

### Tratamento de Erros

```python
try:
    # processamento
    print(json.dumps(resultado))
except ValueError as e:
    print(json.dumps({"error": "INVALID_INPUT", "message": str(e)}), file=sys.stderr)
    sys.exit(1)
except Exception as e:
    print(json.dumps({"error": "PROCESSING_ERROR", "message": str(e)}), file=sys.stderr)
    sys.exit(1)
```

## Casos de Uso para Scripts

### ✅ Ideal para Scripts

- Parser de logs estruturados
- Validação de CPF/CNPJ/cartões
- Cálculos matemáticos complexos
- Transformação JSON ↔ XML ↔ CSV
- Análise de expressões regulares
- Ordenação/filtragem de datasets
- Geração de hashes/checksums
- Processamento de lote (batch)

### ❌ Evitar Scripts

- Respostas conversacionais simples
- Decisões baseadas em contexto
- Tarefas que exigem criatividade
- Processamento que muda frequentemente
- Integrações com APIs (usar tools)

## Checklist de Qualidade

### Subagent
- [ ] Nome em kebab-case válido
- [ ] Descrição ≤100 caracteres
- [ ] Papel único e bem definido
- [ ] E/S estruturadas em YAML/JSON
- [ ] Tratamento de erro explícito
- [ ] Script Python se lógica >3 passos

### Skill
- [ ] Nome em kebab-case válido
- [ ] Reutilizável em múltiplos contextos
- [ ] Instruções ≤8 passos
- [ ] Exemplos representativos
- [ ] Parâmetros tipados e documentados
- [ ] Script Python para lógica complexa

### Script Python
- [ ] Shebang `#!/usr/bin/env python3`
- [ ] Type hints nos parâmetros
- [ ] Leitura de stdin via JSON
- [ ] Saída em stdout via JSON
- [ ] Erros em stderr com código
- [ ] Docstrings em funções principais
- [ ] Tratamento de exceções robusto