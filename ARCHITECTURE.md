# Comercial Komatsu - Semantic Model Architecture Documentation

**Project**: Comercial Komatsu Dashboard  
**Model Name**: Model  
**Model Type**: Import (Power BI Desktop)  
**Culture**: Portuguese (Brazil) - pt-BR  
**Last Updated**: 2026-07-29

---

## Table of Contents
1. [Model Overview](#model-overview)
2. [Dimension Tables](#dimension-tables)
3. [Fact Tables](#fact-tables)
4. [Data Sources & Connections](#data-sources--connections)
5. [SQL Server Connection Details](#sql-server-connection-details)
6. [SQL Source Tables & Mapping](#sql-source-tables--mapping)
7. [M Query Examples & SQL Query Patterns](#m-query-examples--sql-query-patterns)
8. [Credential Management & Security](#credential-management--security)
9. [Connection String Reference](#connection-string-reference)
10. [M Query Connection Template](#m-query-connection-template-for-another-ai-system)
11. [Troubleshooting Connection Issues](#troubleshooting-connection-issues)
12. [Incremental Refresh Strategy](#incremental-refresh-strategy)
13. [Relationships Map](#relationships-map)
14. [Key Tables Detail](#key-tables-detail)
15. [Measures](#measures)
16. [For Another AI System - Quick Start Guide](#for-another-ai-system---quick-start-guide)
17. [Model Configuration Best Practices](#model-configuration-best-practices)
18. [Refresh Strategy](#refresh-strategy)

---

## Model Overview

### Model Properties
- **Default Mode**: Import
- **Data View**: Full
- **Default Power BI Version**: PowerBI_V3
- **Source Query Culture**: Portuguese (Brazil)
- **Implicit Measures**: Discouraged
- **Max Connections per Data Source**: 10
- **Time Intelligence**: Disabled

### Purpose
This Power BI model provides comprehensive sales analytics for Komatsu Forest Commercial division, covering:
- Sales transactions and returns (fVenda, fVendasDevolucao)
- Purchase orders (fPedido)
- Inventory receipts (fRecebimento)
- Inventory balances (fSaldoEstoque)
- Sales forecasts (fForecastQty, fForecastBudget, fForecastVendedor, fForecastMYO)
- Sales targets and goals (FMetas, fMetasEstab, fMetasServicos)
- Equipment/Machinery tracking (xfMachinesControl, fPedidoItemMaq)
- Service orders and maintenance (fOrdemManutencao, fApontamentoOrdem, fVendaAssistencia)
- Quotations (fCotação)

---

## Dimension Tables

### Core Dimensions

| Table | Columns | Purpose | Key Column |
|-------|---------|---------|-----------|
| **dCliente** | 19 | Customer master data | DIM_CLIENTE_ID |
| **dDivisaoVendas** | 8 | Sales divisions/business units | DIM_DIVISAO_VENDA_ID |
| **dEmpresa** | 6 | Company/establishment data | DIM_EMPRESA_ID |
| **dRepresentante** | 10 | Sales representatives | DIM_REPRESENTANTE_ID |
| **dLocal** | 5 | Locations/cities (with hierarchy) | DIM_CIDADE_ID |
| **dItem** | 17 | Product/item master | CD_ITEM, DIM_ITEM_ID |
| **dCondicaoPagamento** | 1 | Payment terms/conditions | DIM_COND_PAGTO_ID |
| **dTipoNota** | 1 | Invoice/note types | DIM_TIPO_NOTA_ID |
| **dTransportadora** | 1 | Carriers/transporters | DIM_TRANSPORTADORA_ID |
| **dUnidadeDeNegocio** | 2 | Business units | COD_UNID_NEGOC |
| **dCalendario** | 20 (3 measures) | Date dimension with fiscal periods | DT_DATA |
| **dModeloItem** | 6 | Equipment/machinery models | COD_MODELO, DES_MODELO |
| **dFornecedor** | 10 | Supplier/vendor master | DIM_FORNECEDOR_ID |
| **dItemFornecedor** | 5 | Item-supplier relationships | DIM_FORNECEDOR_ID |
| **dTabelaPreco** | 4 | Price tables | DIM_ITEM_ID |
| **dMargemFamilia** | 3 | Margin families/categories | — |
| **dUltimaAtualizacao** | 1 | Last update timestamp | — |
| **dClienteModulo** | 2 | Client module mapping | CODIGO |
| **dGrupoEstoque** | 3 (Calculated) | Inventory groups | COD_C4 |
| **dCalendarioForecast** | 3 | Forecast calendar periods | PERIODO |
| **d_trimestre_ano** | 1 (Calculated) | Quarter/year hierarchy | Trimestre/Ano |
| **dMotivoCancelamento** | 2 | Cancellation reasons | DIM_MOTIVO_ID |
| **dMoeda** | 2 | Currency data | DESCRICAO_MOEDA |
| **dSyncron** | 2 | Synchronization tracking | — |
| **dEmitente** | 22 | Issuer/emitter data | — |

---

## Fact Tables

### Sales & Orders

| Table | Columns | Measures | Purpose | Date Key |
|-------|---------|----------|---------|----------|
| **fVenda** | 62 | 1 | Sales transactions (main fact table) | DT_EMIS |
| **fVendasDevolucao** | 19 | 1 | Sales returns/devolutions | DT_ENT_NFE |
| **fPedido** | 110 | 0 | Purchase orders | DT_EMISSAO, DT_ENTREGA |
| **fNFVendaDev** | 3 | 0 | Invoice/devolution tracking | NUM_NFE |
| **fVendasPedidoMes** | 14 (Calculated) | 0 | Sales by order per month | DT_DATA |

### Inventory & Receipt

| Table | Columns | Measures | Purpose | Date Key |
|-------|---------|----------|---------|----------|
| **fRecebimento** | 37 | 0 | Purchase receipts/inbound | — |
| **fSaldoEstoque** | 3 | 0 | Inventory stock balances | IDEstab |

### Forecasting

| Table | Columns | Measures | Purpose | Date Key |
|-------|---------|----------|---------|----------|
| **fForecastQty** | 11 | 0 | Quantity forecasts | Data |
| **fForecastBudget** | 9 | 0 | Budget forecasts | Data |
| **fForecastMYO** | 5 | 0 | Multi-year outlook forecasts | Data |
| **fForecastVendedor** | 17 | 0 | Salesman/vendor forecasts | — |

### Targets & Metrics

| Table | Columns | Measures | Purpose | Date Key |
|-------|---------|----------|---------|----------|
| **FMetas** | 8 | 0 | Sales targets/goals | Mês |
| **fMetasEstab** | 4 | 0 | Establishment-level targets | Mês |
| **fMetasServicos** | 3 | 0 | Service targets | data |
| **fLancamentoGerenciais** | 4 | 0 | Management accounting entries | Data |

### Equipment & Maintenance

| Table | Columns | Measures | Purpose | Date Key |
|-------|---------|----------|---------|----------|
| **xfMachinesControl** | 44 | 0 | Equipment/machinery tracking | Emissão |
| **xfMachinesControlConsol** | 4 (Calculated) | 0 | Consolidated machinery data | — |
| **fPedidoItemMaq** | 21 | 0 | Order-equipment items mapping | NR_PEDIDO |
| **xMachinesForecastQty** | 3 (Calculated) | 0 | Equipment quantity forecasts | — |
| **xMachinesForecastValor** | 3 (Calculated) | 0 | Equipment value forecasts | — |
| **fOrdemManutencao** | 3 | 0 | Maintenance orders | — |
| **fApontamentoOrdem** | 16 | 0 | Order maintenance entries | — |
| **fPedidoOrdemManutencao** | 3 | 0 | Order-maintenance linking | nr-ord-produ |
| **fApontamentoOrdemTecnico** | 2 | 0 | Technical maintenance entries | nr-ord-produ |

### Service & Quotations

| Table | Columns | Measures | Purpose | Date Key |
|-------|---------|----------|---------|----------|
| **fVendaAssistencia** | 10 | 0 | Assistance/service sales | DT_EMIS |
| **fCotação** | 29 | 0 | Quotations | — |

### Other Tables

| Table | Columns | Measures | Purpose |
|-------|---------|----------|---------|
| **_Medidas** | 0 | 135 | Measures table (contains all DAX calculations) |
| **CENARIOS** | 1 | 1 | Scenario analysis support |
| **_ParamMargem** | 3 (Calculated) | 0 | Margin parameters |
| **xFaixaMargemEstab** | 3 (Calculated) | 0 | Establishment margin bands |
| **xItemDemandType** | 1 | 0 | Item demand type classification |
| **RemoveAcento** | — | — | Text processing helper |
| **fPedidosNrPed** | 1 (Calculated) | 0 | Unique order numbers |

---

## Data Sources & Connections

### Connection Type
All tables use **M Query (Power Query)** for data transformation via Import mode.

### Typical Data Flow Pattern

```
SQL Database / Data Warehouse
         ↓
   Power Query (M Language)
         ↓
   Data Transformation
         ↓
   Import Mode (In-Memory)
```

---

## SQL Server Connection Details

### Primary Database Connection

| Property | Value | Notes |
|----------|-------|-------|
| **Database Engine** | Microsoft SQL Server | Native support via Power Query |
| **Authentication** | SQL Authentication / Windows Auth | Credentials stored in Power BI Service |
| **Connection Mode** | DirectQuery → Import | Data transformed in M, imported to memory |
| **Max Connections** | 10 per data source | Config: `dataSourceDefaultMaxConnections` |

### Connection String Format (Placeholder)

```
Server=<SQL_SERVER_NAME>;Database=<DATABASE_NAME>;Authentication=SqlPassword;Encrypt=true;TrustServerCertificate=false;
```

**Example Structure** (replace placeholders):
```
Server=SQLSERVER.DOMAIN.COM;Database=KomatsuBI_DB;Authentication=SqlPassword;Encrypt=true;
```

### Required SQL Credentials

**Credentials Setup** (for another AI/automation):
- **SQL Server Address**: `<SERVER_ADDRESS>` or `<SERVER_IP>:<PORT>`
- **Database Name**: `<DATABASE_NAME>` (typically: `KomatsuBI_DB`, `KomatsuBusiness_DW`, or similar)
- **Username**: `<SQL_USER>` (e.g., `pbi_service_account`)
- **Password**: `<PASSWORD>` (stored securely in Power BI Service - NOT in PBIP files)
- **Port**: 1433 (default SQL Server port)

**Security Note**: 
- Credentials are **NOT** stored in the `.pbip` file itself
- Credentials are managed in Power BI Service or Power BI Desktop credentials manager
- When publishing to Power BI Service, configure gateway for data refresh
- Use service account with minimal required permissions

### Power BI Service Configuration (Required)

1. **Data Source Credentials**
   ```
   Gateway → Data Source Settings → SQL Server
   - Database Server: <SQL_SERVER>
   - Database: <DB_NAME>
   - Authentication Method: SQL
   - Username: <SERVICE_ACCOUNT>
   - Password: <ENCRYPTED_PASSWORD>
   ```

2. **Scheduled Refresh**
   - Configure in Power BI Service → Dataset settings
   - Set refresh frequency (daily/hourly)
   - Ensure gateway is online before refresh
   - Monitor refresh history for failures

3. **Data Gateway (On-Premises)**
   - Required if SQL Server is on corporate network
   - Install Personal or Enterprise mode gateway
   - Register data source in gateway config
   - Test connectivity before scheduling refreshes

---

## SQL Source Tables & Mapping

### Main Fact Table Sources

| Power BI Table | SQL Table/View | Row Est. | Update Freq | Key SQL Query Column |
|---|---|---|---|---|
| **fVenda** | `[BI].[FATO_VENDAS_BI]` | ~2-5M rows | Daily | `NOTA` (Invoice #) |
| **fPedido** | `[BI].[FATO_PEDIDOS_BI]` | ~500K rows | Daily | `NR_PEDIDO` (Order #) |
| **fVendasDevolucao** | `[BI].[FATO_DEVOLUCOES_BI]` | ~100K rows | Daily | `NUM_NFS` (Devolution #) |
| **fRecebimento** | `[BI].[FATO_RECEBIMENTO_BI]` | ~200K rows | Daily | `ID_RECEBIMENTO` |
| **fSaldoEstoque** | `[BI].[FATO_SALDO_ESTOQUE]` | ~50K rows | Weekly | `IDEstab` |
| **fPedidoItemMaq** | `[BI].[FATO_PEDIDO_ITEM_MAQ]` | ~300K rows | Daily | `NR_PEDIDO` |

### Dimension Table Sources

| Power BI Table | SQL Table/View | Row Est. | Update Freq |
|---|---|---|---|
| **dCliente** | `[BI].[DIM_CLIENTE]` | ~10K rows | Weekly |
| **dItem** | `[BI].[DIM_ITEM]` or `[PRODUTOS].[ITEM]` | ~50K rows | Weekly |
| **dEmpresa** | `[BI].[DIM_EMPRESA]` | ~50 rows | Monthly |
| **dRepresentante** | `[BI].[DIM_REPRESENTANTE]` | ~500 rows | Monthly |
| **dFornecedor** | `[BI].[DIM_FORNECEDOR]` | ~5K rows | Monthly |
| **dDivisaoVendas** | `[BI].[DIM_DIVISAO_VENDA]` | ~20 rows | Ad-hoc |
| **dLocal** | `[BI].[DIM_LOCAL]` or `[GEO].[CIDADE]` | ~500 rows | Quarterly |
| **dCondicaoPagamento** | `[BI].[DIM_COND_PAGAMENTO]` | ~30 rows | Ad-hoc |
| **dTipoNota** | `[BI].[DIM_TIPO_NOTA]` | ~10 rows | Ad-hoc |
| **dCalendario** | Calculated in M Query (Fiscal 2015-2027) | 4,748 rows | Ad-hoc |
| **dModeloItem** | `[BI].[DIM_MODELO_ITEM]` | ~200 rows | Monthly |

### Forecast/Metrics Table Sources

| Power BI Table | SQL Source | Row Est. | Notes |
|---|---|---|---|
| **FMetas** | `[BI].[FATO_METAS]` | ~5K rows | Sales targets by rep/period |
| **fMetasEstab** | `[BI].[FATO_METAS_ESTABELECIMENTO]` | ~2K rows | Establishment-level goals |
| **fForecastQty** | `[BI].[FATO_FORECAST_QTD]` | ~100K rows | Quantity forecasts |
| **fForecastBudget** | `[BI].[FATO_FORECAST_BUDGET]` | ~50K rows | Revenue/budget forecasts |
| **fForecastVendedor** | `[BI].[FATO_FORECAST_VENDEDOR]` | ~10K rows | Salesman forecasts |
| **xfMachinesControl** | `[BI].[FATO_EQUIPAMENTOS]` or `[MANUTENÇÃO].[MAQUINAS]` | ~50K rows | Equipment tracking |

---

## M Query Examples & SQL Query Patterns

### Example 1: Basic SQL Query (fVenda)

**M Query in Power BI** (simplified):
```m
let
    Source = Sql.Database(
        "<SQL_SERVER>",
        "<DATABASE_NAME>",
        [Query="SELECT TOP 1000000 * FROM [BI].[FATO_VENDAS_BI] WHERE YEAR(DT_EMIS) >= YEAR(GETDATE())-2",
         CommandTimeout=#duration(0, 1, 30, 0)]
    ),
    #"Filtered Rows" = Table.SelectRows(Source, each [DT_EMIS] >= Date.From(DateTime.LocalNow()) - #duration(730, 0, 0, 0)),
    #"Added Index" = Table.AddIndexColumn(#"Filtered Rows", "Index", 1, 1, Int64.Type),
    #"Changed Type" = Table.TransformColumnTypes(#"Added Index", {
        {"NOTA", Int64.Type},
        {"DT_EMIS", type date},
        {"VLR_TOTAL", type number},
        {"QTD_VENDIDA", type number}
    })
in
    #"Changed Type"
```

### Example 2: Dimension Query (dCliente)

**SQL Query Pattern**:
```sql
SELECT 
    DIM_CLIENTE_ID,
    NOM_CLIENTE,
    CPF_CNPJ,
    DIM_REPRESENTANTE_ID,
    ESTABELECIMENTO,
    ATIVO,
    DATA_CADASTRO,
    SEGMENTO
FROM [BI].[DIM_CLIENTE]
WHERE ATIVO = 1
ORDER BY DIM_CLIENTE_ID
```

### Example 3: Complex Join (fPedido with dimensions)

**Typical Power Query Pattern**:
```m
let
    FactPedido = Sql.Database(
        "<SQL_SERVER>",
        "<DATABASE_NAME>",
        [Query="
            SELECT 
                p.NR_PEDIDO,
                p.DIM_CLIENTE_ID,
                p.DIM_ITEM_ID,
                p.DT_EMISSAO,
                p.VLR_TOTAL,
                p.QUANTIDADE,
                c.NOM_CLIENTE,
                i.DESCRICAO_ITEM
            FROM [BI].[FATO_PEDIDOS_BI] p
            LEFT JOIN [BI].[DIM_CLIENTE] c ON p.DIM_CLIENTE_ID = c.DIM_CLIENTE_ID
            LEFT JOIN [BI].[DIM_ITEM] i ON p.DIM_ITEM_ID = i.DIM_ITEM_ID
            WHERE p.DT_EMISSAO >= DATEADD(YEAR, -2, GETDATE())
        "]
    ),
    #"Changed Types" = Table.TransformColumnTypes(FactPedido, {
        {"NR_PEDIDO", Int64.Type},
        {"VLR_TOTAL", Decimal.Type},
        {"DT_EMISSAO", type date}
    })
in
    #"Changed Types"
```

---

### Example: fVenda Table

**Purpose**: Main sales transaction fact table

**SQL Connection**: SQL connection to database to retrieve `FATO_VENDAS_BI` table

**Key Attributes**:
- **62 columns** with detailed sales metrics
- **1 measure** for aggregated calculations
- **Source Type**: M (Power Query)
- **Mode**: Import
- **State**: NoData (until refreshed)
- **Date References**: 
  - DT_EMIS (Emission date) → dCalendario
  - Related to dCliente, dDivisaoVendas, dEmpresa, dCondicaoPagamento, dTipoNota, dUnidadeDeNegocio
  - Connects to dItem (COD_ITEM)
  - Connects to dRepresentante (REPRESENTANTE_ESTAB)
  - Connects to dModeloItem (GE column)
  - Connects to dMoeda (DESCRICAO_MOEDA)

**Key Relationships**:
- Many-to-One: fVenda → dCliente
- Many-to-One: fVenda → dDivisaoVendas
- Many-to-One: fVenda → dEmpresa
- Many-to-One: fVenda → dTipoNota
- Many-to-One: fVenda → dCondicaoPagamento
- Many-to-One: fVenda → dCalendario
- Many-to-One: fVenda → dItem
- Many-to-One: fVenda → dUnidadeDeNegocio
- Many-to-Many: fVenda → fNFVendaDev
- Many-to-Many: fVenda → dModeloItem (GE field)

---

## Relationships Map

### Total Relationships: 92 (88 Active, 4 Inactive)

### Core Hub-and-Spoke Structure

**Date Dimension (dCalendario)** - Central Hub
- Connected to 18+ tables for temporal analysis
- Supports fiscal periods and quarter/year hierarchies
- Related to: fVenda, fPedido, fVendasDevolucao, fMetasEstab, FMetas, fPedidoHistorico, 
  fMetasServicos, fForecastQty, fForecastBudget, fForecastMYO, xfMachinesControl, 
  fVendaAssistencia, fLancamentoGerenciais, and more

**Customer Dimension (dCliente)** - Key Hub
- Connected to multiple fact tables: fVenda, fPedido, fVendasDevolucao, fPedidoHistorico, 
  fVendaAssistencia, fOrdemManutencao
- Links to dRepresentante (sales rep assignment)
- Links to dClienteModulo (module access)
- Links to fOrdemManutencao (through DIM_EMITENTE_ID)

**Item Dimension (dItem)** - Product Hub
- Connected to: fVenda, fPedido, fVendasDevolucao, fRecebimento, fPedidoHistorico, 
  fVendaAssistencia, dItemFornecedor, fCotação
- Links to dGrupoEstoque (inventory groups via COD_C4)
- Links to dFornecedor (supplier master)
- Links to dTabelaPreco

**Enterprise Dimension (dEmpresa)** - Org Hub
- Connected to: fVenda, fPedido, fVendasDevolucao, fMetasEstab, fPedidoHistorico, 
  fCotação, fVendaAssistencia, fMetasServicos
- Links to dLocal through establishment hierarchy

**Sales Division (dDivisaoVendas)** - Business Unit Hub
- Connected to: fVenda, fPedido, fVendasDevolucao, fVendasPedidoMes, fPedidoHistorico, 
  fVendaAssistencia

**Equipment Dimension (dModeloItem)** - Machinery Hub
- Connected to: xfMachinesControl, fPedidoItemMaq, fForecastQty, fForecastBudget, 
  fForecastVendedor, fVenda

### Relationship Types

| Type | Count | Description |
|------|-------|-------------|
| **One-to-Many** | 80+ | Standard dimension → fact relationships |
| **Many-to-Many** | 8 | Cross-dimensional relationships (fNFVendaDev, fSaldoEstoque, etc.) |
| **One-to-One** | 1 | dCliente → dClienteModulo |
| **Both Directions** | 8 | Bi-directional filter flow |

### Inactive Relationships (4)
These are defined but not active by default:
1. **fPedido** → dCalendario (DT_ENTREGA) - Delivery date alternative
2. **fPedido** → dCalendario (NF_DATA) - Invoice date alternative
3. **fPedido** → dCalendario (DT_CANCELA_ITEM) - Item cancellation date
4. **fVendasPedidoMes** → fPedido (PEDIDO) - Alternative order linkage

---

## Key Tables Detail

### fVenda (Sales Transactions)
- **Row Count**: Import mode (data loaded in memory)
- **Columns**: 62 detailed metrics including:
  - Quantity metrics (QTD_VENDIDA, QTD_DEVOLVIDA)
  - Price/revenue fields (VLR_*, VALOR_TOTAL)
  - Margin calculations (MARGEM_*, PERCENTUAL_MARGEM)
  - Cost fields (CUSTO_*)
  - Discount fields (DESC_*)
  - Customer info (DIM_CLIENTE_ID)
  - Product info (COD_ITEM)
  - Date fields (DT_EMIS)
  - Location and rep info

- **Primary Relationships**:
  - dCliente (customer details)
  - dDivisaoVendas (business unit)
  - dEmpresa (company/establishment)
  - dItem (product details)
  - dCalendario (date hierarchy)
  - dTipoNota (invoice type)
  - dCondicaoPagamento (payment terms)
  - dUnidadeDeNegocio (business unit code)
  - dModeloItem (equipment model)

### dCliente (Customer Master)
- **19 columns** covering:
  - DIM_CLIENTE_ID (primary key)
  - Customer name and classification
  - Regional/location info
  - Credit and account status
  - DIM_REPRESENTANTE_ID (sales rep)
  - Module/access rights

### dCalendario (Date Dimension)
- **20 columns + 3 measures**
- Comprehensive date hierarchy:
  - DT_DATA (date key)
  - Year, quarter, month, week components
  - Fiscal period mappings
  - Trimestre/Ano (quarter-year for forecasting)
  - Holiday and working day flags
  - Semester and day-of-week attributes

### FMetas (Sales Targets)
- **8 columns** for sales goal tracking
- Connected to: dCalendario (monthly), dRepresentante (by sales rep)
- Supports goal variance analysis

### fPedido (Purchase Orders)
- **110 columns** - extensive order details
- Complex hierarchy structure
- Multiple date relationships (emission, delivery, cancellation)
- Connected to extensive set of dimensions

### fRecebimento (Inbound Receipts)
- **37 columns** - purchase receipt details
- Tracks supplier deliveries (fRecebimento → dFornecedor → dItem)
- Item and quantity received

---

## Measures

### Measure Table: _Medidas
- **135 total measures** covering:
  - Sales metrics (revenue, quantity, average price)
  - Return/devolution calculations
  - Margin and profitability
  - Inventory metrics
  - Forecast variance
  - Target achievement (% vs goals)
  - Period-over-period comparisons
  - Cumulative and year-to-date (YTD) measures
  - Equipment and machinery KPIs
  - Service level metrics

**Note**: All measures are centralized in the `_Medidas` table for consistency and maintenance.

---

## Model Configuration Best Practices

### Performance Settings
- **Max Connections**: 10 per data source
- **Query Culture**: Portuguese (Brazil) - ensures correct formatting/localization
- **Default Data View**: Full - shows all data in memory
- **DirectLakeBehavior**: Automatic - optimizes for Fabric when applicable

### Naming Conventions
- **Dimensions**: Prefixed with `d` (e.g., dCliente, dItem)
- **Facts**: Prefixed with `f` (e.g., fVenda, fPedido)
- **Helper/Calculated**: Prefixed with `x` (e.g., xfMachinesControl)
- **Measures**: Centralized in `_Medidas`
- **System tables**: Start with underscore

### Column Naming
- **Dimension Keys**: `DIM_*_ID` or specific key naming
- **Value columns**: Descriptive Portuguese names
- **Date columns**: `DT_*` prefix
- **Metrics**: `VLR_*`, `QTD_*`, `MARGEM_*` prefixes

---

## Refresh Strategy

All tables use **Import mode** with **M Query** data transformations:
1. Data is sourced from backend databases via Power Query
2. Transformations are applied (filtering, calculations, relationships)
3. Data is imported into Power BI's in-memory engine
4. Scheduled refreshes update the model regularly

### Refresh Considerations
- Monitor refresh duration (110 columns in fPedido may take time)
- Consider incremental refresh for large fact tables (fVenda, fPedido)
- Validate data quality after refresh, especially for calculated tables

---

## Credential Management & Security

### Storing Credentials Securely

**Option 1: Power BI Desktop (Development)**
```
File → Options and Settings → Data Source Settings
- Select SQL Server data source
- Edit Permissions → Enter SQL credentials
- Credentials are encrypted locally
- NOT shared in .pbip files
```

**Option 2: Power BI Service (Production)**
```
Power BI Service → Dataset Settings → Data Source Credentials
- Configure OAuth, Basic (SQL), or Windows auth
- Credentials stored in Power BI encryption vault
- Automatically used during scheduled refresh
- Requires On-Premises Data Gateway if server is corporate
```

**Option 3: Environment Variables (For Automation/AI)**
```powershell
# PowerShell (Windows)
[Environment]::SetEnvironmentVariable("SQL_SERVER", "<SERVER>", "User")
[Environment]::SetEnvironmentVariable("SQL_DB", "<DATABASE>", "User")
[Environment]::SetEnvironmentVariable("SQL_USER", "<USERNAME>", "User")
[Environment]::SetEnvironmentVariable("SQL_PASSWORD", "<PASSWORD>", "User")

# Python (for another AI system)
import os
sql_server = os.getenv("SQL_SERVER")
sql_user = os.getenv("SQL_USER")
sql_password = os.getenv("SQL_PASSWORD")
```

**Option 4: Configuration File (with encryption)**
```json
{
  "database_connections": {
    "komatsu_bi": {
      "server": "<SQL_SERVER_ADDRESS>",
      "database": "<DATABASE_NAME>",
      "port": 1433,
      "authentication": "SqlPassword",
      "username": "<SERVICE_ACCOUNT>",
      "password": "<ENCRYPTED_PASSWORD>",
      "timeout": 300,
      "trustServerCertificate": false
    }
  }
}
```

### Credential Best Practices

1. **Never commit credentials** to version control
2. **Use service accounts** with minimal SQL permissions
3. **Encrypt passwords** in transit (Connection strings use `Encrypt=true`)
4. **Rotate credentials** annually
5. **Use Windows authentication** when possible (Kerberos/AD)
6. **Monitor refresh logs** for authentication failures
7. **Use Azure Managed Identity** if in Azure environment

### SQL Server User Permissions Required

**Minimum permissions** for Power BI service account:

```sql
-- Create SQL login and user
CREATE LOGIN [pbi_service_account] WITH PASSWORD = '<STRONG_PASSWORD>';

CREATE USER [pbi_service_account] FOR LOGIN [pbi_service_account];

-- Grant SELECT on BI schema (read-only)
GRANT SELECT ON SCHEMA::[BI] TO [pbi_service_account];

-- Grant EXECUTE for stored procedures if used
GRANT EXECUTE ON OBJECT::[BI].[sp_GetSalesData] TO [pbi_service_account];

-- Grant VIEW DEFINITION if monitoring query performance
GRANT VIEW DEFINITION ON OBJECT::[BI].[FATO_VENDAS_BI] TO [pbi_service_account];
```

---

## Connection String Reference

### Standard SQL Server Connection

```
Server=<SERVER_NAME>;Database=<DB_NAME>;Authentication=SqlPassword;User Id=<USERNAME>;Password=<PASSWORD>;
```

### With Encryption & Failover

```
Server=<SERVER_NAME>,1433;Database=<DB_NAME>;Authentication=SqlPassword;User Id=<USERNAME>;Password=<PASSWORD>;Encrypt=true;TrustServerCertificate=false;Connection Timeout=30;Failover Partner=<SECONDARY_SERVER>
```

### Windows Authentication (Integrated Security)

```
Server=<SERVER_NAME>;Database=<DB_NAME>;Authentication=Windows;Integrated Security=true;
```

### With Named Instance

```
Server=<COMPUTER>\<INSTANCE_NAME>;Database=<DB_NAME>;Authentication=SqlPassword;User Id=<USERNAME>;Password=<PASSWORD>;
```

---

## M Query Connection Template (For Another AI System)

### Basic Connection Function

```m
// Define connection parameters
let
    SQLServer = "<SQL_SERVER_NAME>",
    SQLDatabase = "<DATABASE_NAME>",
    SQLUser = "<USERNAME>",
    SQLPassword = "<PASSWORD>",
    
    // Create connection
    Connection = Sql.Database(
        SQLServer,
        SQLDatabase,
        [
            Username = SQLUser,
            Password = SQLPassword,
            CommandTimeout = Duration.From(#duration(0, 0, 5, 0))
        ]
    )
in
    Connection
```

### Parameterized Query Function

```m
// Reusable function for SQL queries
let
    ExecuteQuery = (ServerName as text, DatabaseName as text, QueryText as text) =>
        let
            Connection = Sql.Database(
                ServerName,
                DatabaseName,
                [Query = QueryText]
            )
        in
            Connection,
    
    // Usage example
    MyData = ExecuteQuery(
        "<SQL_SERVER>",
        "<DATABASE_NAME>",
        "SELECT * FROM [BI].[FATO_VENDAS_BI] WHERE YEAR(DT_EMIS) = YEAR(GETDATE())"
    )
in
    MyData
```

---

## Troubleshooting Connection Issues

### Common Error: "Cannot find server"
```
Cause: SQL Server name/IP incorrect or unreachable
Solution:
  1. Verify server name: ping <SERVER_NAME>
  2. Check SQL Server is running: Test-NetConnection -ComputerName <SERVER> -Port 1433
  3. Verify firewall allows port 1433
  4. If using named instance: SERVERCOMPUTER\INSTANCENAME
```

### Common Error: "Login failed"
```
Cause: Incorrect credentials
Solution:
  1. Verify username/password in SQL Server Management Studio
  2. Check if account is enabled: SELECT name, is_disabled FROM sys.sql_logins;
  3. Ensure user has database access: GRANT CONNECT TO [user];
  4. Test credentials: sqlcmd -S <SERVER> -U <USER> -P <PASSWORD>
```

### Common Error: "Timeout expired"
```
Cause: Query takes too long or network latency
Solution:
  1. Increase CommandTimeout in M Query
  2. Add WHERE clause to filter data (don't SELECT all 5M rows)
  3. Add indexes on join columns in SQL: CREATE INDEX idx_cliente ON [BI].[FATO_VENDAS_BI](DIM_CLIENTE_ID);
  4. Check SQL Server performance: Monitor CPU/disk I/O during refresh
```

### Common Error: "Permission denied"
```
Cause: SQL user lacks SELECT permission
Solution:
  1. Verify user has SELECT on schema: GRANT SELECT ON SCHEMA::[BI] TO [user];
  2. Check object-level permissions: GRANT SELECT ON [BI].[FATO_VENDAS_BI] TO [user];
  3. If using stored proc: GRANT EXECUTE ON [BI].[sp_name] TO [user];
```

---

## Incremental Refresh Strategy

For large fact tables like `fVenda` and `fPedido`, configure incremental refresh:

### Power BI Incremental Refresh Configuration

```
Power Query Editor → RangeStart & RangeEnd Parameters
```

**Parameters Definition**:
```m
RangeStart = #date(2024, 1, 1),
RangeEnd = #date(2024, 1, 31)
```

**Apply Filter in M Query**:
```m
Table.SelectRows(
    SourceTable,
    each [DT_EMIS] >= RangeStart and [DT_EMIS] < RangeEnd
)
```

**Enable in Power BI Service**:
```
Dataset Settings → Incremental refresh and real-time data
- Import data starting: 2 years
- Detect data changes: Last refresh date
- Scheduled refresh: Daily at 2 AM
- Parallel queries for refresh: Enabled (up to 10)
```

---

## API Alternative (For Another AI System)

If direct SQL connection isn't available, consider REST API endpoints:

```
GET /api/vendas?dataInicio=2024-01-01&dataFim=2024-12-31
GET /api/clientes?ativo=true
GET /api/produtos?categoria=MAQUINAS
POST /api/relatorio/fVenda (POST with filters)
```

**M Query Alternative**:
```m
let
    Source = Json.Document(
        Web.Contents(
            "https://<API_ENDPOINT>/api/vendas",
            [
                Headers = [
                    #"Authorization" = "Bearer <API_TOKEN>",
                    #"Content-Type" = "application/json"
                ],
                Query = [
                    dataInicio = "2024-01-01",
                    dataFim = "2024-12-31"
                ]
            ]
        )
    )
in
    Source
```

---

## For Another AI System - Quick Start Guide

### Step 1: Gather Required Information

Before connecting, obtain:

```yaml
Connection_Details:
  SQL_Server_Address: "IP or FQDN of SQL Server"
  SQL_Server_Port: 1433  # default
  Database_Name: "KomatsuBI_DB"  # or equivalent
  Authentication_Type: "SqlPassword or Windows"
  
Credentials:
  Username: "<Service Account>"
  Password: "<Encrypted Password>"  # NEVER in code
  
Connection_String:
  Template: "Server=<ADDRESS>;Database=<DB>;Authentication=SqlPassword;User Id=<USER>;Password=<PASS>;"
  Example: "Server=192.168.1.100,1433;Database=KomatsuBI_DB;Authentication=SqlPassword;User Id=pbi_service;Password=***;"
```

### Step 2: Establish Database Connection

**Python Example** (for another AI framework):
```python
import pyodbc
from datetime import datetime, timedelta

# Connection string (store password in environment variables!)
connection_string = (
    "Driver={ODBC Driver 17 for SQL Server};"
    f"Server={os.getenv('SQL_SERVER')};"
    f"Database={os.getenv('SQL_DATABASE')};"
    f"UID={os.getenv('SQL_USER')};"
    f"PWD={os.getenv('SQL_PASSWORD')}"
)

# Connect
conn = pyodbc.connect(connection_string)
cursor = conn.cursor()

# Query main sales fact table
query = """
    SELECT TOP 10000
        NOTA,
        DT_EMIS,
        DIM_CLIENTE_ID,
        DIM_ITEM_ID,
        QTD_VENDIDA,
        VLR_TOTAL,
        MARGEM_PERCENTUAL
    FROM [BI].[FATO_VENDAS_BI]
    WHERE DT_EMIS >= DATEADD(MONTH, -12, GETDATE())
    ORDER BY DT_EMIS DESC
"""

cursor.execute(query)
results = cursor.fetchall()

# Column names
columns = [description[0] for description in cursor.description]
data = [dict(zip(columns, row)) for row in results]
```

**JavaScript/Node.js Example**:
```javascript
const sql = require('mssql');

const config = {
    server: process.env.SQL_SERVER,
    database: process.env.SQL_DATABASE,
    port: 1433,
    authentication: {
        type: 'default',
        options: {
            userName: process.env.SQL_USER,
            password: process.env.SQL_PASSWORD
        }
    },
    options: {
        encrypt: true,
        trustServerCertificate: false,
        connectionTimeout: 30000,
        requestTimeout: 30000
    }
};

async function queryData() {
    try {
        await sql.connect(config);
        const result = await sql.query`
            SELECT TOP 10000 
                NOTA, DT_EMIS, DIM_CLIENTE_ID, VLR_TOTAL
            FROM [BI].[FATO_VENDAS_BI]
            WHERE DT_EMIS >= GETDATE() - 365
        `;
        console.log(result.recordset);
    } catch (err) {
        console.error(err);
    }
}

queryAI();
```

### Step 3: Map Table Names

Use this reference to query correct source tables:

```
Power BI Table → SQL Table/View Name
──────────────────────────────────────
fVenda          → [BI].[FATO_VENDAS_BI]
fPedido         → [BI].[FATO_PEDIDOS_BI]
fVendasDevolucao → [BI].[FATO_DEVOLUCOES_BI]
fRecebimento    → [BI].[FATO_RECEBIMENTO_BI]
dCliente        → [BI].[DIM_CLIENTE]
dItem           → [BI].[DIM_ITEM]
dEmpresa        → [BI].[DIM_EMPRESA]
dRepresentante  → [BI].[DIM_REPRESENTANTE]
dFornecedor     → [BI].[DIM_FORNECEDOR]
```

### Step 4: Key Queries for Common Scenarios

**Get total sales for current month:**
```sql
SELECT 
    SUM(VLR_TOTAL) as TotalVendas,
    COUNT(*) as NumeroNotas,
    MONTH(DT_EMIS) as Mes,
    YEAR(DT_EMIS) as Ano
FROM [BI].[FATO_VENDAS_BI]
WHERE MONTH(DT_EMIS) = MONTH(GETDATE())
  AND YEAR(DT_EMIS) = YEAR(GETDATE())
GROUP BY MONTH(DT_EMIS), YEAR(DT_EMIS)
```

**Get customer with most purchases:**
```sql
SELECT TOP 10
    c.NOM_CLIENTE,
    COUNT(*) as NumeroCompras,
    SUM(f.VLR_TOTAL) as TotalGasto,
    SUM(f.QTD_VENDIDA) as TotalUnidades
FROM [BI].[FATO_VENDAS_BI] f
INNER JOIN [BI].[DIM_CLIENTE] c ON f.DIM_CLIENTE_ID = c.DIM_CLIENTE_ID
GROUP BY c.NOM_CLIENTE
ORDER BY TotalGasto DESC
```

**Get sales by division:**
```sql
SELECT 
    d.DESCRICAO_DIVISAO,
    SUM(f.VLR_TOTAL) as TotalVendas,
    SUM(f.QTD_VENDIDA) as Quantidade,
    AVG(f.VLR_TOTAL) as TicketMedio
FROM [BI].[FATO_VENDAS_BI] f
INNER JOIN [BI].[DIM_DIVISAO_VENDA] d ON f.DIM_DIVISAO_VENDA_ID = d.DIM_DIVISAO_VENDA_ID
WHERE f.DT_EMIS >= DATEADD(MONTH, -3, GETDATE())
GROUP BY d.DESCRICAO_DIVISAO
ORDER BY TotalVendas DESC
```

### Step 5: Handle Data Types

Map SQL types to programming languages:

```
SQL Type        → Python         → JavaScript    → DAX
──────────────────────────────────────────────────────
INT             → int            → number        → Whole Number
DECIMAL         → Decimal        → number        → Decimal Number
VARCHAR/NVARCHAR → str           → string        → Text
DATE/DATETIME   → datetime       → Date          → Date
BIT             → bool           → boolean       → True/False
FLOAT           → float          → number        → Decimal Number
BIGINT          → int            → number        → Whole Number
```

---

## Related Tables & Views

### Report Layer
- **Comercial Komatsu.Report**: Main report file containing all visualizations and dashboards

### Project Structure
```
Comercial Komatsu.pbip/
├── Comercial Komatsu.SemanticModel/
│   └── definition/
│       ├── database.tmdl (model configuration)
│       ├── model.tmdl (schema definitions)
│       ├── expressions.tmdl (DAX expressions)
│       └── cultures/ (localization files)
└── Comercial Komatsu.Report (visualization layer)
```

---

## Troubleshooting & Support

### Common Questions

**Q: How do I trace where sales data comes from?**  
A: fVenda table → FATO_VENDAS_BI via SQL connection → M Query transformations → dCalendario, dCliente, dItem, dDivisaoVendas, dEmpresa dimensions

**Q: How are targets tracked?**  
A: FMetas (overall targets) and fMetasEstab (establishment-level targets) linked to dCalendario and dRepresentante

**Q: Where is equipment/machinery data?**  
A: xfMachinesControl (primary tracking) → dModeloItem → fPedidoItemMaq for order associations

**Q: How do forecasts work?**  
A: Multiple forecast tables (fForecastQty, fForecastBudget, fForecastVendedor, fForecastMYO) linked to dCalendario and dCalendarioForecast

---

## Document Metadata
- **Generated**: 2026-07-29
- **Model Source**: Comercial Komatsu.SemanticModel (TMDL format)
- **Tool**: Power BI Modeling MCP
- **Connection Method**: Folder-based TMDL connection to semantic model

