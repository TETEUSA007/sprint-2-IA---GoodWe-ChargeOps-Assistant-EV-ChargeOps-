# Chatbot — GoodWe ChargeOps Assistant

Solução desenvolvida para o **EV Challenge 2026**, em parceria entre **FIAP** e **GoodWe**.
O foco é o **EV ChargeOps**: gestão do uso compartilhado de carregadores de veículos
elétricos em condomínios.

> Este README cobre a **Sprint 1** (planejamento) e a **Sprint 2** (implementação).

---

## Integrantes

- Matheus Sabino da Silva Guedes — RM 572907
- Lucas Bellezzo Figueiredo — RM 569734
- Maria Luiza Vieira de Freitas — RM 571535
- Matheus Arruda Camara Soares — RM 571594
- Giovanna Ferreira — RM 571822

---

## Problema abordado

Em condomínios com carregadores de veículos elétricos surgem desafios como:

- Dificuldade para organizar horários de uso.
- Falta de clareza sobre regras de recarga.
- Dúvidas recorrentes de moradores.
- Ausência de orientação rápida sobre consumo e cobrança.
- Sobrecarga de atendimento para o síndico ou administrador.
- Necessidade de registrar ciclos de recarga e orientar o uso correto.

## Proposta do chatbot

O **GoodWe ChargeOps Assistant** é um chatbot operacional voltado para **síndicos e
administradores** de condomínios. Ele orienta sobre agendamento, regras de uso, registro
de ciclos, cobrança proporcional, falhas básicas, comunicação com moradores e boas práticas.

A persona escolhida é o **síndico/administrador**, pois é quem concentra regras, controle de
uso, reclamações e dúvidas recorrentes — onde o chatbot mais reduz tempo de atendimento.

---

## Tecnologias e justificativa

| Tecnologia | Uso | Justificativa |
|---|---|---|
| **Python** | Backend do chatbot | Linguagem simples, muito usada em IA, boa integração com APIs. |
| **Groq + Llama 3.3 70B** | Interpretação e geração das respostas | Nível **gratuito sem cartão de crédito**, inferência muito rápida e suporte a function calling. O modelo **Llama** já era uma das opções previstas na Sprint 1. |
| **Streamlit** | Interface de interação | Permite uma UI simples e demonstrável para os testes. |
| **GitHub** | Versionamento e entrega | Documentação e histórico do projeto. |

> A Sprint 1 permitia "Llama, OpenAI, Gemini ou outra". Optamos por executar o **Llama 3.3
> via Groq** por ser totalmente gratuito (sem cartão e sem exigência de verificação de
> idade) e compatível com o padrão de API da OpenAI, mantendo o código simples e portátil.

---

## O que foi implementado na Sprint 2

- **Injeção de contexto via system prompt** (`system_prompt.txt`, da Sprint 1).
- **Histórico de mensagens / memória de contexto** — diálogos contínuos e coerentes.
- **Few-shot prompting** (diferencial) — exemplos que fixam o tom, distintos dos casos de teste.
- **Function calling** (diferencial) — a ferramenta `registrar_ciclo_recarga` grava ciclos em
  um CSV local, simulando o controle operacional de uma integração futura.
- **Duas interfaces**: terminal (CLI) e web (Streamlit).
- **Harness de testes** que executa os 6 casos da Sprint 1 e gera um relatório automático.

---

## Estrutura do projeto

```
goodwe-chargeops-chatbot/
├── README.md
├── requirements.txt
├── .env.example
├── .gitignore
├── system_prompt.txt          # contexto-base (Sprint 1)
├── src/
│   ├── config.py              # carrega env e system prompt
│   ├── tools.py               # function calling (registrar ciclo)
│   ├── few_shot.py            # exemplos few-shot
│   ├── chatbot.py             # núcleo: histórico + prompt + tools
│   ├── cli.py                 # interface de terminal
│   └── app.py                 # interface Streamlit
├── tests/
│   ├── test_cases.json        # 6 casos de teste da Sprint 1
│   ├── run_tests.py           # executa os testes e gera o relatório
│   └── demo_conversa.py       # demonstra a memória de contexto
└── docs/
    ├── RESULTADOS_TESTES.md   # resultados dos testes
    └── ROTEIRO_VIDEO.md       # roteiro do vídeo de demonstração
```

---

## Como obter a chave GRATUITA do Groq

1. Acesse **https://console.groq.com/keys** e crie a conta (e-mail, Google ou GitHub).
2. Não é preciso cartão de crédito nem verificação de idade.
3. Clique em **"Create API Key"**, copie a chave (começa com `gsk_...`) e cole no `.env`.

> Nível gratuito: ~30 requisições/minuto e centenas de milhares de tokens/dia — suficiente
> para o projeto.

---

## Como executar (local)

1. **Clone o repositório** e entre na pasta:
   ```bash
   git clone <URL_DO_REPOSITORIO>
   cd goodwe-chargeops-chatbot
   ```

2. **Crie um ambiente virtual e instale as dependências:**
   ```bash
   python -m venv .venv
   source .venv/bin/activate    # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. **Configure a chave da API** (via variável de ambiente — nunca no código):
   ```bash
   cp .env.example .env          # Windows: copy .env.example .env
   # edite o .env e preencha GROQ_API_KEY
   ```

4. **Rode a interface desejada:**
   ```bash
   # Web (recomendado para o vídeo):
   streamlit run src/app.py

   # Terminal:
   python -m src.cli
   ```

### Variáveis de ambiente

| Variável | Obrigatória | Padrão | Descrição |
|---|---|---|---|
| `GROQ_API_KEY` | Sim | — | Chave gratuita da API do Groq. |
| `GROQ_MODEL` | Não | `llama-3.3-70b-versatile` | Modelo usado. |
| `GROQ_TEMPERATURE` | Não | `0.3` | Criatividade das respostas. |

### Execução no Google Colab

No Colab, **não use `.env`**. Guarde a chave em **Secrets** e exporte antes de rodar:
```python
!pip install -r requirements.txt
from google.colab import userdata
import os
os.environ["GROQ_API_KEY"] = userdata.get("GROQ_API_KEY")

from src.chatbot import ChargeOpsChatbot
bot = ChargeOpsChatbot()
print(bot.enviar("Como organizo o uso do carregador entre os moradores?"))
```
> O Streamlit não roda bem dentro do Colab; lá, use a CLI ou chame a classe direto.

---

## Como rodar os testes

```bash
python -m tests.run_tests
```
Isso executa os 6 casos de `tests/test_cases.json` e **gera/atualiza**
`docs/RESULTADOS_TESTES.md` com pergunta, resposta obtida e uma avaliação
heurística (a ser confirmada manualmente).

Para ver a memória de contexto em ação:
```bash
python -m tests.demo_conversa
```

---

## Exemplos de uso

**Pergunta:** Como organizo o uso do carregador entre os moradores?
**Resposta (resumida):** sugere agendamento de horários, limite de tempo por morador,
registro de cada ciclo e regras de permanência após o término da carga.

**Pergunta:** Registre o ciclo do morador João, do dia 2026-06-10, com 12 kWh.
**Resposta:** o assistente chama a ferramenta `registrar_ciclo_recarga`, grava a linha no
CSV e confirma o registro ao síndico.

---

## Segurança

- A chave da API é lida **somente** de variável de ambiente (`.env` ou Colab Secrets).
- O `.env` está no `.gitignore` e **nunca** deve ser versionado.
- Nenhuma chave aparece no código ou no repositório.

---

## Vídeo de demonstração

Link (YouTube, não listado): **<INSERIR_LINK_DO_VIDEO>**
