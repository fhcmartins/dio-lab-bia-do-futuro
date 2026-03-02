# Base de Conhecimento

## Dados Utilizados

Descreva se usou os arquivos da pasta `data`, por exemplo:

| Arquivo | Formato | Utilização no Agente |
|---------|---------|---------------------|
| `historico_atendimento.csv` | CSV | Fornece a memória de contexto. Permite que o agente identifique se o usuário já teve dúvidas recorrentes ou problemas técnicos anteriores, garantindo um atendimento mais fluido e evitando repetições de instruções já fornecidas. |
| `perfil_investidor.json` | JSON | Atua como a camada de governança e personalização. O agente extrai o perfil de risco (Moderado) para filtrar o tipo de recomendação e utiliza os valores das Metas (Reserva de Emergência e Viagem) como pontos de referência para calcular o progresso financeiro do usuário. |
| `produtos_financeiros.json` | JSON | Serve como o catálogo de soluções. O agente cruza os produtos disponíveis com o perfil moderado do usuário (CDB, Tesouro, Ações) para sugerir apenas investimentos que estejam em conformidade com o apetite ao risco do cliente. |
| `transacoes.csv` | CSV | Funciona como o motor de análise comportamental. O agente processa as colunas de categoria, valor e tipo para realizar o fechamento mensal, identificar gastos excessivos (ex: Lazer e Restaurante) e verificar a disponibilidade de saldo para novos aportes. |

---

## Adaptações nos Dados

> Você modificou ou expandiu os dados mockados? Descreva aqui.

Para este projeto, expandi o arquivo perfil_investidor.json incluindo uma seção de 'Metas Financeiras', permitindo que o agente faça recomendações proativas para objetivos específicos (ex: Reserva de Emergência ou Viagem). Também limpei o transacoes.csv para garantir que todas as datas estejam no formato ISO-8601, facilitando a análise temporal pela LLM local.

---

## Estratégia de Integração

### Como os dados são carregados?
> Descreva como seu agente acessa a base de conhecimento.

Utilizo a biblioteca Pandas para carregar os arquivos CSV e a biblioteca nativa json do Python. Os dados são processados e convertidos em strings formatadas (Markdown) para que o Ollama consiga interpretar tabelas e listas de forma estruturada, otimizando o consumo da janela de contexto.

### Como os dados são usados no prompt?
> Os dados vão no system prompt? São consultados dinamicamente?

Os dados são injetados dinamicamente no System Prompt. Para evitar sobrecarga de memória no Ollama, o agente realiza um pré-processamento: ele resume o saldo total e lista apenas as últimas 10 transações relevantes para a pergunta do usuário, garantindo que o modelo foque no contexto mais recente e evite alucinações numéricas.

```text
DADOS DO CLIENTE E PERFIL (data/perfil_investidor.json):

{
  "nome": "João Silva",
  "idade": 32,
  "profissao": "Analista de Sistemas",
  "renda_mensal": 5000.00,
  "perfil_investidor": "moderado",
  "objetivo_principal": "Construir reserva de emergência",
  "patrimonio_total": 15000.00,
  "reserva_emergencia_atual": 10000.00,
  "aceita_risco": false,
  "metas": [
    {
      "meta": "Completar reserva de emergência",
      "valor_necessario": 15000.00,
      "prazo": "2026-06"
    },
    {
      "meta": "Entrada do apartamento",
      "valor_necessario": 50000.00,
      "prazo": "2027-12"
    }
  ]
}

TRANSAÇÕES DO CLIENTE (data/transacoes.csv):

data,descricao,categoria,valor,tipo
2025-10-01,Salário,receita,5000.00,entrada
2025-10-02,Aluguel,moradia,1200.00,saida
2025-10-03,Supermercado,alimentacao,450.00,saida
2025-10-05,Netflix,lazer,55.90,saida
2025-10-07,Farmácia,saude,89.00,saida
2025-10-10,Restaurante,alimentacao,120.00,saida
2025-10-12,Uber,transporte,45.00,saida
2025-10-15,Conta de Luz,moradia,180.00,saida
2025-10-20,Academia,saude,99.00,saida
2025-10-25,Combustível,transporte,250.00,saida

HISTÓRICO DE ATENDIMENTO DO CLIENTE (data/historico_atendimento.csv):

data,canal,tema,resumo,resolvido
2025-09-15,chat,CDB,Cliente perguntou sobre rentabilidade e prazos,sim
2025-09-22,telefone,Problema no app,Erro ao visualizar extrato foi corrigido,sim
2025-10-01,chat,Tesouro Selic,Cliente pediu explicação sobre o funcionamento do Tesouro Direto,sim
2025-10-12,chat,Metas financeiras,Cliente acompanhou o progresso da reserva de emergência,sim
2025-10-25,email,Atualização cadastral,Cliente atualizou e-mail e telefone,sim

PRODUTOS FINANCEIROS (data/produtos_financeiros.json)

[
  {
    "nome": "Tesouro Selic",
    "categoria": "renda_fixa",
    "risco": "baixo",
    "rentabilidade": "100% da Selic",
    "aporte_minimo": 30.00,
    "indicado_para": "Reserva de emergência e iniciantes"
  },
  {
    "nome": "CDB Liquidez Diária",
    "categoria": "renda_fixa",
    "risco": "baixo",
    "rentabilidade": "102% do CDI",
    "aporte_minimo": 100.00,
    "indicado_para": "Quem busca segurança com rendimento diário"
  },
  {
    "nome": "LCI/LCA",
    "categoria": "renda_fixa",
    "risco": "baixo",
    "rentabilidade": "95% do CDI",
    "aporte_minimo": 1000.00,
    "indicado_para": "Quem pode esperar 90 dias (isento de IR)"
  },
  {
    "nome": "Fundo Multimercado",
    "categoria": "fundo",
    "risco": "medio",
    "rentabilidade": "CDI + 2%",
    "aporte_minimo": 500.00,
    "indicado_para": "Perfil moderado que busca diversificação"
  },
  {
    "nome": "Fundo de Ações",
    "categoria": "fundo",
    "risco": "alto",
    "rentabilidade": "Variável",
    "aporte_minimo": 100.00,
    "indicado_para": "Perfil arrojado com foco no longo prazo"
  }
]
...


---

## Exemplo de Contexto Montado

> Mostre um exemplo de como os dados são formatados para o agente.

```
### 👤 PERFIL DO CLIENTE
- **Nome:** João Silva (32 anos)
- **Perfil de Risco:** Moderado
- **Objetivo Principal:** Construir reserva de emergência
- **Patrimônio Total:** R$ 15.000,00

### 🎯 METAS ATUAIS
1. **Completar reserva de emergência:** Necessário R$ 15.000,00 até 2026-06 (Atual: R$ 10.000,00)
2. **Entrada do apartamento:** Necessário R$ 50.000,00 até 2027-12

### 💰 RESUMO FINANCEIRO RECENTE (Últimas Transações)
| Data | Categoria | Descrição | Valor | Tipo |
|---|---|---|---|---|
| 2025-10-25 | transporte | Combustível | R$ 250,00 | saída |
| 2025-10-20 | saude | Academia | R$ 99,00 | saída |
| 2025-10-15 | moradia | Conta de Luz | R$ 180,00 | saída |
| ... | ... | ... | ... | ... |
| 2025-10-01 | receita | Salário | R$ 5.000,00 | entrada |

**Saldo do período analisado:** +R$ 2.500,10

### 💬 MEMÓRIA DE ATENDIMENTO
- Último contato em 2025-10-25 sobre 'Atualização cadastral'.
- Em 2025-10-12 o cliente acompanhou o progresso da reserva de emergência.

### 🏦 PRODUTOS RECOMENDADOS (Para Perfil Moderado)
- **Fundo Multimercado:** Risco médio, CDI + 2% (Diversificação).
- **CDB Liquidez Diária:** Risco baixo, 102% do CDI (Segurança).
- **Tesouro Selic:** Risco baixo, 100% da Selic (Ideal para reserva).

---
**INSTRUÇÃO:** Com base nos dados acima, responda ao usuário de forma consultiva. Foque no progresso das metas e sugira o uso do saldo positivo para a Reserva de Emergência.
```

```
