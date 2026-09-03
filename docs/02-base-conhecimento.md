# Base de Conhecimento

## Dados Utilizados

| Arquivo | Formato | Para que serve no Radar? |
|---------|---------|---------------------|
| `historico_atendimento.csv` | CSV | Para que serve no Radar?Contextualizar o histórico de transações e interações para mapear a evolução dos hábitos financeiros e manter a consistência do acompanhamento proativo. |
| `perfil_investidor.json` | JSON | Personalizar as intervenções e ajustar o tom de voz (de conselheiro empático a treinador firme) com base na gravidade do desvio estatístico. |
| `produtos_financeiros.json` | JSON | Conhecer os gatilhos comportamentais do usuário, identificando de forma precisa os horários, dias da semana e categorias com maior risco de impulsividade. |
| `transacoes.csv` | CSV | Analisar os padrões de microtransações e médias móveis em tempo real para prever anomalias e alertar o usuário antes que o orçamento seja estourado. |

---

## Adaptações nos Dados

> Você modificou ou expandiu os dados mockados? Descreva aqui.

Os dados mockados originais foram reestruturados para focar em finanças comportamentais em vez de apenas rentabilidade.

---

## Estratégia de Integração

### Como os dados são carregados?
> Descreva como seu agente acessa a base de conhecimento.

Os arquivos JSON e CSV são consumidos pelo backend (em Python ou Node.js) no momento do login do usuário para construir o estado inicial.

### Como os dados são usados no prompt?
> Os dados vão no system prompt? São consultados dinamicamente?

Os dados estáticos e de personalidade (perfil do investidor, gatilhos e catálogo de produtos) são fixados no System Prompt para definir as restrições e o estilo de resposta da IA.

---

## Exemplo de Contexto Montado

> Mostre um exemplo de como os dados são formatados para o agente.

```
[System Context - Regras do Radar]
Dados do Cliente:
- Nome: João Silva
- Perfil: Impulsivo
- Renda Mensal: R$ 5.000,00
- Gatilho Mapeado: Compras noturnas e delivery aos finais de semana
- Objetivo Ativo: Completar reserva de emergência sem desvios

[Dynamic Input - Gatilho de Avaliação]
Transações Críticas nas últimas 48h:
- 02/09: iFood Burger (R$ 89,90) - alimentacao_impulsiva
- 03/09: iFood Pizza (R$ 112,50) - alimentacao_impulsiva
Status Atual: Usuário abrindo aplicativo de delivery no sábado à noite.
Ação Requerida da IA: Intervir imediatamente. Assumir tom de "Treinador Firme" e sugerir transferência para o "Cofre Virtual Programado"
```
