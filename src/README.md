# Código da Aplicação

Esta pasta contém o código do Radar: um chat em **Streamlit** que consulta o modelo **Llama 3.1** rodando localmente via **Ollama**, usando os dados de `../data/` como contexto de cada resposta.

## Estrutura

```
Radar-seu-ajudante/
├── data/
│   ├── transacoes.csv
│   ├── historico_atendimento.csv
│   ├── perfil_investidor.json
│   └── produtos_financeiros.json
└── src/
    └── app.py
```

## Como rodar

```bash
# 1. Instalar dependências
pip install -r requirements.txt

# 2. Baixar e rodar o modelo via Ollama
ollama run llama3.1

# 3. Iniciar a aplicação
streamlit run app.py
```

## O que o app.py faz

- Carrega o perfil do cliente, transações, histórico de atendimento e produtos financeiros a partir de `data/`
- Monta um contexto personalizado e injeta no *system prompt* do Radar a cada pergunta
- Exibe um painel lateral com o perfil do cliente e o progresso da reserva de emergência
- Mantém o histórico da conversa em `st.session_state` e envia as últimas interações ao modelo
- Gera as respostas em streaming, consultando o Ollama (`http://localhost:11434/api/generate`)
- Oferece sugestões rápidas de perguntas (chips) para facilitar o uso
