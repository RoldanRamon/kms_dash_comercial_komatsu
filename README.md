# 🌲 Komatsu Forest - Dashboard Comercial 🚜

Este repositório contém o projeto de Power BI do painel comercial da **Komatsu Forest**, com foco no acompanhamento de faturamento, metas, backlog de pedidos, margens e indicadores logísticos (OTIF) e comerciais (Churn).

---

## 🎯 Objetivos do Dashboard

O principal objetivo da dashboard é fornecer à diretoria e aos gestores comerciais uma visão consolidada e detalhada sobre:
1. **Performance Comercial:** Acompanhamento do faturamento real vs. metas por vendedor e por estabelecimento.
2. **Gestão de Peças e Máquinas:** Segmentação da receita e análise específica de quantidade e valores de máquinas faturadas contra o planejado (Budget/Forecast).
3. **Eficiência de Processo (Logística):** Monitoramento do indicador OTIF (On-Time In-Full) para a divisão de peças.
4. **Saúde da Carteira de Clientes:** Monitoramento de churn e controle do backlog de pedidos.

---

## 🧭 Estrutura de Páginas do Relatório

O relatório é composto por 12 páginas principais:

*   📊 **Performance de Vendas:** Dashboard inicial focado no consolidado comercial, faturamento e cumprimento de metas.
*   💰 **Faturamento:** Detalhamento da receita gerada, segregando faturamento líquido e devoluções.
*   📈 **Faturamento Comparativos:** Análise temporal e comparativos Year-over-Year (YoY) do faturamento.
*   ⚖️ **Fat. Comp. Peças:** Comparativo focado especificamente na divisão de peças ("Parts").
*   🏆 **Parts - Faturamento x Meta Vendedores:** Acompanhamento das metas individuais de vendas de peças da equipe comercial.
*   🏢 **Parts - Faturamento x Meta Estabelecimento:** Visão regionalizada das metas de faturamento de peças por filial/estabelecimento.
*   🔄 **Parts - Evolução Pedidos x Faturamento:** Acompanhamento do ciclo de vida dos pedidos, desde a entrada até a emissão da Nota Fiscal.
*   📦 **Parts - Histórico Backlog de Pedidos:** Monitoramento da carteira de pedidos em aberto e tempo de atendimento.
*   📋 **Tabela de Faturamento:** Relatório analítico detalhado das Notas Fiscais emitidas para auditorias e exportação.
*   🚚 **Parts - OTIF:** Indicador de qualidade de entrega de peças (dentro do prazo e completo).
*   🛑 **Análise de Churn:** Identificação de clientes inativos ou em risco de inatividade comercial.
*   📅 **Horário de Atualização:** Painel técnico indicando a última atualização das cargas de dados.

---

## 🛠️ Regras de Negócio e Medidas DAX

Abaixo estão as definições das principais lógicas e regras de negócio codificadas no modelo semântico:

### 1. Faturamento Líquido Total
Calcula o faturamento real deduzindo as devoluções de vendas. Possui tratamento especial para clientes **Intercompany** (onde é filtrado o tipo de conta correspondente e removidos os filtros padrão da dimensão empresa):
```dax
measure 'Faturamento Liquido Total' = 
SWITCH(
    TRUE(),
    SELECTEDVALUE(dEmpresa[DIM_EMPRESA_ID]) = "INTERCOMPANY",
        CALCULATE(
            ([TOTAL VLR VENDAS LIQ] - [TOTAL VLR DEVOLUCOES LIQ] ),
            REMOVEFILTERS(dEmpresa),  
            FILTER(
                dCliente,
                dCliente[TIPO DE CONTA] IN {"AB", "COL", "IND", "KDB", "UY"}
            )
        ),
    ([TOTAL VLR VENDAS LIQ] - [TOTAL VLR DEVOLUCOES LIQ] )
)
```

### 2. Projeção de Faturamento e Dias Úteis (Fiscal vs Orçamentário)
As previsões consideram os dias úteis realizados contra os dias úteis planejados (previstos) para o ano, calculando o faturamento projetado da seguinte forma:
$$\text{Faturamento Projetado} = \left(\frac{\text{Faturamento Líquido}}{\text{Dias Úteis Realizados}} \times \text{Dias Úteis Previstos}\right) + \text{Faturamento Gerencial}$$

### 3. Vendas e Forecast de Máquinas
*   **Quantidade Vendida:** Contagem distinta de registros de máquinas faturadas na tabela `fPedidoItemMaq` (usando `Qtde Máquinas Vendidas`).
*   **Aderência ao Forecast:** Comparação entre o volume real faturado e o budget estabelecido (tanto em quantidade com `% Forecast Qtde Máquina` quanto em valor financeiro com `% Forecast Valor Máquina`).

### 4. Gestão de Margem
*   **% Margem de Produtos:** Mede a rentabilidade direta das vendas:
    ```dax
    var vFat = [TOTAL VLR VENDAS LIQ]
    var vCusto = SUMX(fVenda, fVenda[CUSTO PRODUTO REAL] * fVenda[QUANTIDADE])
    var vResultado = DIVIDE((vFat - vCusto), vFat, 0)
    return vResultado
    ```
*   **Valor Disponível e Perda de Margem:** Lógicas que calculam o ganho de margem sob parâmetros cadastrados (`Valor Disponível Margem`) e as perdas financeiras em transações efetuadas abaixo do custo médio (`Valor Perda Margem`).

### 5. Carteira de Pedidos (Backlog)
*   **% Backlog:** Acompanhamento da parcela de faturamento pendente no backlog histórico.
*   **% Carteira Atual:** Relação do faturamento de pedidos do mês atual em relação ao valor total de pedidos de venda (PDV).

### 6. OTIF (On-Time In-Full) Logístico
*   Mede a porcentagem de linhas de pedidos de peças faturadas e despachadas no prazo acordado e em sua totalidade (`% OTIF LOG`).

---

## 🗄️ Estrutura do Repositório (PBIP)

O projeto está organizado no formato moderno de arquivos de metadados do Power BI (PBIP), facilitando o versionamento via Git:
*   [Comercial Komatsu.pbip](file:///report/Comercial Komatsu.pbip): Arquivo de entrada do projeto.
*   **`report/Comercial Komatsu.SemanticModel/`**: Contém arquivos TMDL (Tabular Model Definition Language) que definem tabelas, colunas, relacionamentos e medidas.
    *   [_Medidas.tmdl](file:///report/Comercial Komatsu.SemanticModel/definition/tables/_Medidas.tmdl): Centralizador de todas as medidas DAX do projeto.
*   **`report/Comercial Komatsu.Report/`**: Contém a definição das páginas, visuais e layouts das dashboards.
