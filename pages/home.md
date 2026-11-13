\---

title: Home

\---



\# Welcome



This is your new project's homepage. Edit this file to get started.

\## 1. Visão Estratégica: Economia e Eficiência Energética



```sql kpis\_estrategicos

SELECT 

&#x20;   SUM(economia\_dimerizacao\_brl) AS economia\_total,

&#x20;   ROUND(AVG(CASE WHEN mttr\_minutos > 0 THEN mttr\_minutos END), 1) AS mttr\_medio,

&#x20;   COUNT(CASE WHEN status\_alerta != 'Normal' THEN 1 END) AS total\_alertas

FROM fato\_telemetria

```



{% big\_value data="kpis\_estrategicos" value="sum(economia\_total)" title="Economia com Dimerização" fmt="brl2" /%}



{% big\_value data="kpis\_estrategicos" value="avg(mttr\_medio)" title="MTTR Médio (min)" fmt="num1" /%}



{% big\_value data="kpis\_estrategicos" value="sum(total\_alertas)" title="Alertas Críticos Ativos" /%}



```sql economia\_por\_bairro

SELECT 

&#x20;   d.bairro,

&#x20;   SUM(f.economia\_dimerizacao\_brl) AS economia\_brl,

&#x20;   SUM(f.consumo\_kwh) AS consumo\_total\_kwh

FROM fato\_telemetria f

JOIN dim\_dispositivos d ON f.device\_id = d.device\_id

GROUP BY d.bairro

ORDER BY economia\_brl DESC

```



{% bar\_chart 

&#x20;   data="economia\_por\_bairro" 

&#x20;   x="bairro" 

&#x20;   y="sum(economia\_brl)" 

&#x20;   title="Economia Orçamentária por Bairro (R$)"

&#x20;   y\_fmt="brl2"

/%}



\---



\## 2. Visão Tática: Desempenho de Manutenção (MTTR)



```sql tatico\_manutencao

SELECT 

&#x20;   d.tipo AS infraestrutura,

&#x20;   d.bairro,

&#x20;   ROUND(AVG(f.mttr\_minutos), 1) AS tempo\_medio\_atendimento,

&#x20;   COUNT(\*) AS ocorrencias

FROM fato\_telemetria f

JOIN dim\_dispositivos d ON f.device\_id = d.device\_id

WHERE f.mttr\_minutos > 0

GROUP BY d.tipo, d.bairro

```



{% table data="tatico\_manutencao" /%}



\---



\## 3. Visão Operacional: Alertas Georreferenciados



```sql alertas\_mapa

SELECT 

&#x20;   f.data\_hora,

&#x20;   d.device\_id AS dispositivo,

&#x20;   d.tipo,

&#x20;   d.bairro,

&#x20;   d.latitude,

&#x20;   d.longitude,

&#x20;   f.status\_alerta,

&#x20;   f.nivel\_preenchimento\_pct

FROM fato\_telemetria f

JOIN dim\_dispositivos d ON f.device\_id = d.device\_id

WHERE f.status\_alerta != 'Normal'

```



{% map title="Alertas Críticos por Localização" initial\_position=\[-3.733, -38.510] zoom=12 %}

&#x20;   {% point\_layer 

&#x20;       data="alertas\_mapa" 

&#x20;       lat="latitude" 

&#x20;       lng="longitude" 

&#x20;       point\_title="dispositivo" 

&#x20;       color\_value="status\_alerta"

&#x20;       tooltip\_fields=\["bairro", "tipo", "status\_alerta"]

&#x20;   /%}

{% /map %}



{% table data="alertas\_mapa" search=true /%}

