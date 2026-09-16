# Código da Aplicação

Esta pasta contém o código do seu agente financeiro.

## Estrutura Sugerida

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

## Exemplo de requirements.txt

```
streamlit
openai
python-dotenv
```

## Como Rodar

```bash
# Instalar dependências
pip install -r requirements.txt

# Rodar a aplicação
streamlit run app.py
```
## app.py

from pathlib import Path
import json
import pandas as pd
import requests
import streamlit as st

# =========== CONFIGURAÇÃO DA PÁGINA ===========
st.set_page_config(
    page_title="Radar - Educador Financeiro",
    page_icon="🎓",
    layout="wide"
)

# =========== CAMINHOS ===============
BASE_DIR = Path(__file__).resolve().parent.parent
DATA_DIR = BASE_DIR / "data"

# =========== CONFIGURAÇÃO ===============
OLLAMA_URL = "http://localhost:11434/api/generate"
MODELO = "llama3.1"

# ============ CARREGAR DADOS ============
@st.cache_data
def carregar_dados():
    with open(DATA_DIR / "perfil_investidor.json", "r", encoding="utf-8") as f:
        perfil = json.load(f)
    transacoes = pd.read_csv(DATA_DIR / "transacoes.csv", encoding="utf-8")
    historico = pd.read_csv(DATA_DIR / "historico_atendimento.csv", encoding="utf-8")
    with open(DATA_DIR / "produtos_financeiros.json", "r", encoding="utf-8") as f:
        produtos = json.load(f)
    return perfil, transacoes, historico, produtos

perfil, transacoes, historico, produtos = carregar_dados()

# ============ MONTAR CONTEXTO BASE ============
contexto_base = f"""
CLIENTE: {perfil['nome']}, {perfil['idade']} anos, profissão {perfil['profissao']}
RENDA MENSAL: R$ {perfil['renda_mensal']:.2f}
PERFIL INVESTIDOR: {perfil['perfil_investidor']}
OBJETIVO: {perfil['objetivo_principal']}
GATILHO COMPORTAMENTAL: {perfil['gatilho_comportamental']}
PATRIMÔNIO: R$ {perfil['patrimonio_total']:.2f} | RESERVA ATUAL: R$ {perfil['reserva_emergencia_atual']:.2f}
METAS: {json.dumps(perfil['metas'], ensure_ascii=False)}

TRANSAÇÕES RECENTES:
{transacoes.to_string(index=False)}

ATENDIMENTOS ANTERIORES:
{historico.to_string(index=False)}

PRODUTOS DISPONÍVEIS:
{json.dumps(produtos, indent=2, ensure_ascii=False)}
"""

# ============ SYSTEM PROMPT ============
SYSTEM_PROMPT = """Você é o Radar, um educador financeiro amigável, empático e didático especializado em economia comportamental e controle de impulsos.

OBJETIVO:
Ensinar conceitos de finanças pessoais de forma simples, usando os dados reais do cliente como exemplos práticos para ajudá-lo a vencer gatilhos emocionais e atingir seus objetivos.

REGRAS OBRIGATÓRIAS:
- NUNCA recomende investimentos específicos como consultoria financeira formal, apenas explique como as opções funcionam e como podem protegê-lo de impulsos;
- JAMAIS responda a perguntas fora do tema de finanças pessoais e economia comportamental. Se ocorrer, lembre gentilmente seu papel;
- Use os dados fornecidos (ex: gastos de delivery no iFood, compras na Steam, meta de reserva de emergência) para dar exemplos super personalizados;
- Linguagem simples, acolhedora e direta, como um mentor amigo;
- Sempre pergunte se o cliente entendeu e como se sente a respeito;
- Responda de forma sucinta e estruturada, com no máximo 3 parágrafos ou tópicos curtos.
"""

# ============ GERADOR DE RESPOSTA STREAMING ============
def stream_resposta(pergunta_atual, historico_mensagens):
    # Formata histórico recente da conversa (últimas 6 interações)
    dialogo = ""
    for m in historico_mensagens[-6:]:
        papel = "Cliente" if m["role"] == "user" else "Radar"
        dialogo += f"\n{papel}: {m['content']}"

    prompt = f"""{SYSTEM_PROMPT}

CONTEXTO DO CLIENTE:
{contexto_base}

HISTÓRICO RECENTE DA CONVERSA:
{dialogo}

Cliente: {pergunta_atual}
Radar:"""

    try:
        r = requests.post(
            OLLAMA_URL,
            json={"model": MODELO, "prompt": prompt, "stream": True},
            stream=True,
            timeout=120
        )
        if r.status_code != 200:
            yield f"⚠️ Erro ao comunicar com o Ollama (Status {r.status_code}): {r.text}"
            return

        for line in r.iter_lines():
            if line:
                data = json.loads(line.decode("utf-8"))
                if "response" in data:
                    yield data["response"]
                if data.get("done", False):
                    break
    except requests.exceptions.ConnectionError:
        yield "⚠️ Não foi possível conectar ao Ollama. Verifique se o serviço está em execução na porta 11434 com `ollama run llama3.1`."
    except requests.exceptions.Timeout:
        yield "⚠️ Tempo limite excedido ao aguardar resposta do modelo."
    except Exception as e:
        yield f"⚠️ Ocorreu um erro inesperado: {str(e)}"

# ============ BARRA LATERAL (PAINEL DO CLIENTE) ============
with st.sidebar:
    st.header("👤 Perfil do Cliente")
    st.write(f"**Nome:** {perfil['nome']}")
    st.write(f"**Perfil:** `{perfil['perfil_investidor'].upper()}`")
    st.write(f"**Gatilho:** {perfil['gatilho_comportamental']}")

    st.divider()

    # Progresso da Reserva de Emergência
    reserva_atual = perfil['reserva_emergencia_atual']
    meta_reserva = perfil['metas'][0]['valor_necessario']
    progresso = min(reserva_atual / meta_reserva, 1.0)

    st.subheader("🎯 Reserva de Emergência")
    st.progress(progresso, text=f"R$ {reserva_atual:,.2f} de R$ {meta_reserva:,.2f} ({int(progresso*100)}%)")

    st.divider()
    if st.button("🗑️ Reiniciar Conversa", use_container_width=True):
        st.session_state.messages = []
        st.rerun()

# ============ INICIALIZAR HISTÓRICO DE MENSAGENS ============
if "messages" not in st.session_state or len(st.session_state.messages) == 0:
    st.session_state.messages = [
        {
            "role": "assistant",
            "content": f"Olá, **{perfil['nome']}**! Sou o **Radar**, seu educador financeiro pessoal.\n\n"
                       f"Percebi que seu objetivo principal é **{perfil['objetivo_principal'].lower()}** e "
                       f"completar sua reserva de **R$ {meta_reserva:,.2f}**.\n\n"
                       f"Como posso te ajudar hoje?"
        }
    ]

# ============ CABEÇALHO DA INTERFACE ============
st.title("🎓 Radar, O Educador Financeiro")
st.caption("Seu assistente comportamental para decisões financeiras conscientes.")

# ============ EXIBIR HISTÓRICO CONTÍNUO ============
for msg in st.session_state.messages:
    with st.chat_message(msg["role"]):
        st.markdown(msg["content"])

# ============ SUGESTÕES RÁPIDAS (CHIPS/BUTTONS) ============
st.write("")
st.markdown("**💡 Sugestões Rápidas:**")
sugestoes = [
    "🍕 Quanto gastei com delivery e impulsos recentemente?",
    "🎯 Como chegar mais rápido na meta de R$ 15.000?",
    "🛡️ Quais produtos podem me proteger de gastar à noite?",
    "🧠 Por que eu sinto tanta vontade de gastar no fim de semana?"
]

cols = st.columns(2)
pergunta_clicada = None

for i, sug in enumerate(sugestoes):
    if cols[i % 2].button(sug, key=f"sug_{i}", use_container_width=True):
        pergunta_clicada = sug

# ============ PROCESSAR ENTRADA DO USUÁRIO ============
pergunta_digitada = st.chat_input("Pergunte ao Radar sobre seus hábitos ou finanças...")
pergunta_final = pergunta_digitada or pergunta_clicada

if pergunta_final:
    # 1. Registrar e exibir mensagem do usuário
    st.session_state.messages.append({"role": "user", "content": pergunta_final})
    with st.chat_message("user"):
        st.markdown(pergunta_final)

    # 2. Gerar e exibir resposta em streaming
    with st.chat_message("assistant"):
        resposta_completa = st.write_stream(stream_resposta(pergunta_final, st.session_state.messages[:-1]))

    # 3. Salvar resposta no histórico
    st.session_state.messages.append({"role": "assistant", "content": resposta_completa})
    st.rerun()
