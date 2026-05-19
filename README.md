Markdown
# 📊 Portal da Transparência: Dashboard de Auditoria e Viagens (ANAC)

Este projeto apresenta uma solução analítica de ponta a ponta (*End-to-End*) para o monitoramento, análise logística e auditoria de conformidade das viagens corporativas da **Agência Nacional de Aviação Civil (ANAC)** no período de 2026. 

O grande diferencial técnico deste projeto reside na **camada de inteligência e modelagem de dados construída diretamente no banco de dados**, utilizando **Views otimizadas** para abstrair a complexidade do modelo relacional bruto e entregar dados perfeitamente limpos e performáticos para o Power BI.

---

## 🏗️ Arquitetura e Engenharia de Dados

Em vez de sobrecarregar o Power BI com relacionamentos complexos, tabelas pesadas e tratamentos via Power Query (M), a estratégia adotada seguiu as melhores práticas de mercado: centralizar as regras de negócio e a transformação dos dados na origem (*Database Level*), utilizando **SQL Server / PostgreSQL**.

### O Papel das Views na Solução:
- **Performance:** Redução drástica do consumo de memória do Power BI, permitindo que o modelo trabalhe apenas com colunas estritamente necessárias.
- **Simplificação do Modelo Star Schema:** Centralização de múltiplas tabelas relacionais (`Viagens`, `Trechos` e `Pagamentos`) em visões consolidadas de Fato e Dimensão.
- **Padronização e Data Quality:** Tratamento de strings, valores nulos (`ISNULL`/`COALESCE`) e conversão de tipos de dados direto na query do banco.
- **Auditoria Automatizada:** Implementação de lógicas condicionais complexas (`CASE WHEN`) no SQL para criar uma "Malha Fina" automática antes do dado chegar ao painel visual.

---

## 🛠️ Implementação Técnica: A Criação da View de Auditoria

Abaixo está o código SQL desenvolvido para consolidar os dados orçamentários, unificar as justificativas de viagem e aplicar as **regras de compliance** para identificação de *outliers* (anomalias de gastos):

```sql
CREATE OR ALTER VIEW vw_BI_Auditoria_Alertas AS
SELECT 
    V.id_processo_viagem,
    V.orgao_solicitante,
    V.nome_viajante,
    V.motivo AS Justificativa_Viagem,
    V.situacao AS Status_Processo,
    ISNULL(V.valor_total_passagens, 0) AS Valor_Passagens,
    ISNULL(V.valor_total_diarias, 0) AS Valor_Diarias,
    (ISNULL(V.valor_total_diarias, 0) + ISNULL(V.valor_total_passagens, 0)) AS Custo_Total,
    
    -- Camada de Inteligência: Regras de Negócio para Auditoria Visual
    CASE 
        WHEN (ISNULL(V.valor_total_diarias, 0) + ISNULL(V.valor_total_passagens, 0)) > 20000 
            THEN 'Alerta: Alto Custo Executivo'
        WHEN (ISNULL(V.valor_total_diarias, 0) + ISNULL(V.valor_total_passagens, 0)) BETWEEN 10000 AND 20000 
            THEN 'Análise: Custo Médio-Alto'
        WHEN V.situacao = 'Não realizada' AND (ISNULL(V.valor_total_diarias, 0) > 0 OR ISNULL(V.valor_total_passagens, 0) > 0) 
            THEN 'Crítico: Pago e Não Realizado'
        WHEN V.motivo IS NULL OR LEN(TRIM(V.motivo)) < 5 
            THEN 'Aviso: Justificativa Ausente/Incompleta'
        ELSE 'Conforme (Normal)'
    END AS Status_Auditoria
FROM Viagens V;
```

---
🚀 O Dashboard Interativo (Power BI)
Com a base perfeitamente mastigada pelas Views, o modelo de dados no Power BI ficou leve e responsivo. O painel foi desenhado seguindo conceitos rígidos de UI/UX Corporativo (utilizando a paleta de cores institucional da ANAC e do Governo Federal) e foi dividido em 3 visões estratégicas:

1. Resumo Executivo
Objetivo: Apresentar os principais KPIs financeiros para a diretoria.

Métricas: Custo Total Acumulado, Quantidade de Processos e Divisão Percentual de Investimento (Gráfico de Rosca de Diárias vs. Passagens).

Destaque Técnico: Uso do efeito Container Clean (caixas brancas com bordas arredondadas de 8px e sombras suaves) sobre fundo fosco para evitar a fadiga visual do usuário.

2. Análise Logística e Rotas
Objetivo: Mapear a distribuição geográfica da aplicação dos recursos públicos.

Tecnologia: Implementação do Azure Maps para plotagem dinâmica de bolhas de calor por estado de destino (destino_uf), permitindo identificar os polos que mais demandam orçamento de deslocamento.

3. Auditoria e Compliance (Malha Fina)
Objetivo: Atuar como um painel de controle para o setor de governança fiscalizar anomalias.

Componentes: Um gráfico de Treemap que funciona como um mapa de calor dos status gerados pela View SQL, cruzado com uma Tabela de Detalhes com formatação condicional.

Destaque Técnico: A tabela omite automaticamente os processos "Conformes" e foca o preenchimento de células (em tons pastéis de vermelho e laranja) estritamente nos alertas críticos gerados no banco, otimizando o tempo de resposta da equipe de auditoria.

📈 Lições Aprendidas e Hard Skills Desenvolvidas
Modelagem Relacional de Alta Performance: Compreensão prática de que o tratamento de dados eficiente deve ser feito o mais próximo possível da fonte (SQL) e não na camada de apresentação (BI).

Data Storytelling e UI/UX: Transição de gráficos genéricos de sistema para uma interface com padrão de portal corporativo de transparência.

Regras de Compliance com SQL: Criação de arquiteturas lógicas para transformar dados puramente transacionais em insights acionáveis de auditoria.
