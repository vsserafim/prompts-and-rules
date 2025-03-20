# Regras do Cursor

## Regras do Usuário

Cursor->Settings->User Rules

```
# Regras Base

## Preferências de padrão de codificação
- Variáveis e comentários gerados para o código devem sempre e exclusivamente estar em inglês, independentemente da linguagem usada nos prompts,
- Sempre siga as convenções de nomenclatura de variáveis da linguagem em uso.
- Sempre prefira soluções simples
- Evite a duplicação de código sempre que possível, o que significa verificar outras áreas do código que possam já ter código e funcionalidade semelhantes
- Escreva código que leve em conta os diferentes ambientes: dev, test e prod
- Tenha cuidado para fazer apenas alterações solicitadas ou das quais você tenha certeza de que são bem compreendidas e relacionadas à alteração solicitada
- Ao corrigir um problema ou bug, não introduza um novo padrão ou tecnologia sem primeiro esgotar todas as opções para a implementação existente. E se finalmente fizer isso, certifique-se de remover a implementação antiga posteriormente para não termos lógica duplicada.
- Mantenha o código muito limpo e organizado
- Evite escrever scripts em arquivos sempre que possível, especialmente se o script provavelmente será executado apenas uma vez
- Evite ter arquivos com mais de 200–300 linhas de código. Refatore nesse ponto.
- Dados mock são necessários apenas para testes, nunca mocke dados para ambientes dev ou prod
- Nunca adicione padrões de stubbing ou dados falsos ao código que afeta os ambientes dev ou prod
- Nunca sobrescreva meu arquivo .env sem antes perguntar e confirmar
- No chat e nas explicações, o português brasileiro deve sempre ser usado
- Todas as mensagens de commit devem seguir o Conventional Commits (https://www.conventionalcommits.org/).

## Preferências de fluxo de codificação
- Foque nas áreas do código relevantes para a tarefa
- Não toque em código que não esteja relacionado à tarefa
- Escreva testes abrangentes para toda a funcionalidade principal
- Evite fazer alterações significativas nos padrões e na arquitetura de como uma funcionalidade funciona, após ela ter demonstrado funcionar bem, a menos que explicitamente instruído
- Sempre pense em quais outros métodos e áreas do código podem ser afetados pelas alterações no código
```

## Regras do Projeto

Cursor->Settings->Project Rules

### Pilhas

#### Exemplo de Pilha Python

```
Python para o backend
html/js para o frontend
Bancos de dados SQL, nunca armazenamento em arquivos JSON
Bancos de dados separados para dev, test e prod
Elasticsearch para busca, usando hospedagem elastic.co
Elastic.co terá índices dev e prod
Testes em Python
```
