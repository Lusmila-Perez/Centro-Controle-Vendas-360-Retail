<div align="center">

🌎 [🇧🇷 Português](#-português) · [🇪🇸 Español](#-español)

</div>

---

# 🇧🇷 Português

# 📊 Centro de Controle de Vendas 360° Retail

> **Projeto prático da minha formação em Power BI (nível intermediário).**
> Desenvolvido para aplicar, na prática, modelagem de dados, DAX, inteligência temporal e a Lei de Pareto (80/20) em um cenário de varejo. Faz parte do meu portfólio em construção.
---
![Demonstração do Dashboard](Principal.png)
![Demonstração do Dashboard](Filtro.png)
![Demonstração do Dashboard](Trimestre.png)
![Demonstração Interativa do Dashboard](Dashboard-demo.gif)
---

## 🎯 Cenário de Análise e Objetivos

No contexto do varejo (*Retail*), o projeto aplica recursos do Power BI na análise de dados comerciais, explorando indicadores de desempenho, evolução do faturamento e comportamento dos clientes.

A solução é um painel analítico interativo, construído a partir de uma base de dados de vendas, com quatro dimensões principais:

- 📈 **Evolução e Crescimento Histórico:** faturamento analisado por trimestre, mês e semana, com variação anual (YoY) e mensal (MoM).
- 👥 **Desempenho da Força de Vendas:** resultados por vendedor e agência, comparando contribuições individuais e destacando os principais resultados da operação.
- 💰 **Análise de Pareto (80/20) e Clientes Estratégicos:** identificação dos clientes com maior participação no faturamento e análise da concentração de resultados.
- 🎛️ **Análise Interativa e Multidimensional:** segmentadores por ano, agência, vendedor e cliente.

**Objetivo:** consolidar, em um único ambiente visual, os principais indicadores comerciais e aplicar conceitos de análise de dados, modelagem e visualização.

## 🔎 Principais Insights

Análise sobre o total do período (US$ 208 mill em vendas) e a base de 172 clientes.

- **Crescimento (YoY):** o faturamento cresceu 9,54% em 2025 em relação a 2024 (US$ 24,91 mill vs US$ 22,74 mill), após alta de 1,59% em 2024 e queda de 21,56% em 2023. 
- **Sazonalidade trimestral:** as vendas se distribuem de forma equilibrada entre os trimestres; o T3 se destaca com 26,79% do total (≈ US$ 56 mill de US$ 208 mill), pouco acima dos 25% de uma distribuição uniforme.
- **Força de vendas:** dos 10 vendedores, os 4 primeiros concentram 50,7% do faturamento e os 7 primeiros, 76,77%.
- **Concentração de clientes:** os 5 primeiros vendedores atendem 127 dos 172 clientes (73,84% da carteira) e geram cerca de 60% das vendas (US$ 125 mi).
- **Recomendação:** reter os clientes VIP, desenvolver a base intermediária, investigar a queda de 2023 por agência e por vendedor e replicar as práticas dos 4 vendedores líderes.

---
## 🏗️ Arquitetura de Dados e Estrutura do Modelo

O modelo foi organizado de forma modular: tabela de dados tratados pós-ETL, dimensão temporal e tabelas dedicadas apenas à organização das medidas DAX por funcionalidade.

### 1. Processo de ETL (Power Query) e Dados Consolidados

- **Fonte de Dados (Excel):** tratamento e limpeza dos dados comerciais brutos.
- **`Analisis_ventas`:** tabela principal com os dados tratados após o ETL (padronização de campos, tipos de dados e consistência dos registros transacionais).

### 2. Dimensão Temporal

- **`DIN_Calendario`:** tabela de datas criada para suportar a inteligência temporal (*Time Intelligence*), as análises YoY/MoM e a mudança de granularidade (trimestre, mês e semana).

### 3. Organização das Medidas DAX (Tabelas Temáticas)

Para facilitar a manutenção e a escalabilidade, as medidas foram separadas em tabelas dedicadas:

- **`medidas_base_bruto`:** métricas fundamentais (ex.: Faturamento Total, quantidade e contagens principais).
- **`medidas_inteligencia_tiempo`:** cálculos de inteligência temporal (ex.: vendas do ano anterior, variação percentual YoY e variação mensal (MoM)).
- **`medidas_participacion_ranking`:** participação percentual, ranqueamento da força de vendas e lógica acumulativa para a curva ABC / Lei de Pareto (80/20) de clientes VIP.

---

## 💡 Métricas e Lógica DAX

Estas são as medidas que melhor representam a lógica do modelo. O faturamento total (`ventas_totales`) é a soma da venda líquida e do IVA, e serve de base para todas as demais.

### ⏳ Inteligência temporal: crescimento anual (YoY)

```dax
Ventas_totales_ly =
CALCULATE([ventas_totales], SAMEPERIODLASTYEAR(DIN_Calendario[Fecha]))

Crecimiento_yoy =
DIVIDE([ventas_totales] - [Ventas_totales_ly], [Ventas_totales_ly], 0)
```

> O crescimento mensal (MoM) segue a mesma lógica, usando `DATEADD(DIN_Calendario[Fecha], -1, MONTH)` para obter o mês anterior. As visões **trimestral, mensal e semanal** usam a mesma medida `ventas_totales`: a granularidade muda conforme os campos da `DIN_Calendario` utilizados em cada visual.

### 💰 Análise de Pareto (80/20): ranking e acumulado de clientes

A lógica do Pareto é construída em três passos: ranquear os clientes por faturamento, acumular as vendas até a posição de cada cliente e calcular a participação acumulada sobre o total.

```dax
-- 1. Posição de cada cliente conforme o faturamento
ranking_ventas_clientes =
RANKX(ALLSELECTED(Analisis_ventas[Cliente]), [ventas_totales], , DESC)

-- 2. Faturamento acumulado até a posição do cliente
acum_ventas_clientes =
VAR r = [ranking_ventas_clientes]
RETURN
    SUMX(
        FILTER(
            ALLSELECTED(Analisis_ventas[Cliente]),
            [ranking_ventas_clientes] <= r
        ),
        [ventas_totales]
    )

-- 3. Percentual acumulado sobre o total (base da curva de Pareto)
%_acum_pareto_clientes =
DIVIDE(
    [acum_ventas_clientes],
    CALCULATE([ventas_totales], ALLSELECTED(Analisis_ventas[Cliente]))
)
```

Com o percentual acumulado, é possível identificar os clientes que concentram até 80% do faturamento (clientes VIP).

---

## 📈 Funcionalidades Analíticas

- 📈 **Análise de Desempenho:** acompanhamento do faturamento e da evolução das vendas.
- ⏳ **Inteligência Temporal:** visões por trimestre, mês e semana, com métricas YoY e MoM.
- 👥 **Análise de Clientes:** clientes de maior participação no faturamento (Pareto 80/20).
- 💼 **Análise Comercial:** classificação e acompanhamento de vendedores e agências.
- 🎛️ **Interatividade:** filtros, segmentadores e navegação entre páginas.

## 🛠️ Stack Tecnológico

- **ETL:** Excel, Power Query
- **Modelagem e Cálculos:** Power BI Desktop, DAX
- **Visualização:** Power BI (dashboards e relatórios interativos)

## 🔜 Próximos passos

- Publicar o arquivo `.pbix` 
- Adicionar o dataset (Excel) utilizado no projeto 

---

# 🇪🇸 Español

# 📊 Centro de Control de Ventas 360° Retail

> **Proyecto práctico de mi formación en Power BI (nivel intermedio).**
> Desarrollado para aplicar, en la práctica, modelado de datos, DAX, inteligencia temporal y la Ley de Pareto (80/20) en un escenario de retail. Forma parte de mi portafolio en construcción.

(Capturas y demo interactiva: ver sección en portugués más arriba.)

---

## 🎯 Escenario de Análisis y Objetivos

En el contexto del comercio minorista (*Retail*), el proyecto aplica herramientas de Power BI al análisis de datos comerciales, explorando indicadores de desempeño, evolución de la facturación y comportamiento de los clientes.

La solución es un panel analítico interactivo, construido a partir de una base de datos de ventas, con cuatro dimensiones principales:

- 📈 **Evolución y Crecimiento Histórico:** facturación analizada por trimestre, mes y semana, con variación anual (YoY) y mensual (MoM).
- 👥 **Desempeño de la Fuerza de Ventas:** resultados por vendedor y agencia, comparando aportes individuales y destacando los principales resultados de la operación.
- 💰 **Análisis de Pareto (80/20) y Clientes Estratégicos:** identificación de los clientes con mayor participación en la facturación y análisis de la concentración de resultados.
- 🎛️ **Análisis Interactivo y Multidimensional:** segmentadores por año, agencia, vendedor y cliente.

**Objetivo:** consolidar, en un único entorno visual, los principales indicadores comerciales y aplicar conceptos de análisis de datos, modelado y visualización.

## 🔎 Principales Insights

Análisis sobre el total del período (US$ 208 millones en ventas) y la base de 172 clientes.

- **Crecimiento (YoY):** la facturación creció un 9,54% en 2025 respecto a 2024 (US$ 24,91 millones vs. US$ 22,74 millones), tras un alza del 1,59% en 2024 y una caída del 21,56% en 2023.
- **Estacionalidad trimestral:** las ventas se distribuyen de forma equilibrada entre los trimestres; el T3 destaca con el 26,79% del total (≈ US$ 56 millones de US$ 208 millones), ligeramente por encima del 25% de una distribución uniforme.
- **Fuerza de ventas:** de los 10 vendedores, los 4 primeros concentran el 50,7% de la facturación y los 7 primeros, el 76,77%.
- **Concentración de clientes:** los 5 primeros vendedores atienden a 127 de los 172 clientes (73,84% de la cartera) y generan cerca del 60% de las ventas (US$ 125 millones).
- **Recomendación:** retener a los clientes VIP, desarrollar la base intermedia, investigar la caída de 2023 por agencia y por vendedor, y replicar las prácticas de los 4 vendedores líderes.

---

## 🏗️ Arquitectura de Datos y Estructura del Modelo

El modelo se organizó de forma modular: tabla de datos tratados post-ETL, dimensión temporal y tablas dedicadas solo a organizar las medidas DAX por funcionalidad.

### 1. Proceso de ETL (Power Query) y Datos Consolidados

- **Fuente de Datos (Excel):** tratamiento y limpieza de los datos comerciales brutos.
- **`Analisis_ventas`:** tabla principal con los datos tratados tras el ETL (estandarización de campos, tipos de datos y consistencia de los registros transaccionales).

### 2. Dimensión Temporal

- **`DIN_Calendario`:** tabla de fechas creada para respaldar la inteligencia temporal (*Time Intelligence*), los análisis YoY/MoM y el cambio de granularidad (trimestre, mes y semana).

### 3. Organización de Medidas DAX (Tablas Temáticas)

Para facilitar el mantenimiento y la escalabilidad, las medidas se separaron en tablas dedicadas:

- **`medidas_base_bruto`:** métricas fundamentales (ej.: Facturación Total, cantidad y conteos principales).
- **`medidas_inteligencia_tiempo`:** cálculos de inteligencia temporal (ej.: ventas del año anterior, variación porcentual YoY y variación mensual (MoM)).
- **`medidas_participacion_ranking`:** participación porcentual, ranking de la fuerza de ventas y lógica acumulativa para la curva ABC / Ley de Pareto (80/20) de clientes VIP.

---

## 💡 Métricas y Lógica DAX

Estas son las medidas que mejor representan la lógica del modelo. La facturación total (`ventas_totales`) es la suma de la venta neta y el IVA, y sirve de base para todas las demás.

### ⏳ Inteligencia temporal: crecimiento anual (YoY)

```dax
Ventas_totales_ly =
CALCULATE([ventas_totales], SAMEPERIODLASTYEAR(DIN_Calendario[Fecha]))

Crecimiento_yoy =
DIVIDE([ventas_totales] - [Ventas_totales_ly], [Ventas_totales_ly], 0)
```

> El crecimiento mensual (MoM) sigue la misma lógica, usando `DATEADD(DIN_Calendario[Fecha], -1, MONTH)` para obtener el mes anterior. Las vistas **trimestral, mensual y semanal** usan la misma medida `ventas_totales`: la granularidad cambia según los campos de `DIN_Calendario` usados en cada visual.

### 💰 Análisis de Pareto (80/20): ranking y acumulado de clientes

La lógica del Pareto se construye en tres pasos: ordenar a los clientes por facturación, acumular las ventas hasta la posición de cada cliente y calcular la participación acumulada sobre el total.

```dax
-- 1. Posición de cada cliente según su facturación
ranking_ventas_clientes =
RANKX(ALLSELECTED(Analisis_ventas[Cliente]), [ventas_totales], , DESC)

-- 2. Facturación acumulada hasta la posición del cliente
acum_ventas_clientes =
VAR r = [ranking_ventas_clientes]
RETURN
    SUMX(
        FILTER(
            ALLSELECTED(Analisis_ventas[Cliente]),
            [ranking_ventas_clientes] <= r
        ),
        [ventas_totales]
    )

-- 3. Porcentaje acumulado sobre el total (base de la curva de Pareto)
%_acum_pareto_clientes =
DIVIDE(
    [acum_ventas_clientes],
    CALCULATE([ventas_totales], ALLSELECTED(Analisis_ventas[Cliente]))
)
```

Con el porcentaje acumulado se identifican los clientes que concentran hasta el 80% de la facturación (clientes VIP).

## 📈 Funcionalidades Analíticas

- 📈 **Análisis de Desempeño:** seguimiento de la facturación y de la evolución de las ventas.
- ⏳ **Inteligencia Temporal:** vistas por trimestre, mes y semana, con métricas YoY y MoM.
- 👥 **Análisis de Clientes:** clientes con mayor participación en la facturación (Pareto 80/20).
- 💼 **Análisis Comercial:** clasificación y seguimiento de vendedores y agencias.
- 🎛️ **Interactividad:** filtros, segmentadores y navegación entre páginas.

## 🛠️ Stack Tecnológico

- **ETL:** Excel, Power Query
- **Modelado y Cálculos:** Power BI Desktop, DAX
- **Visualización:** Power BI (dashboards y reportes interactivos)

## 🔜 Próximos pasos

- Publicar el archivo `.pbix` 
- Agregar el dataset (Excel) utilizado en el proyecto
