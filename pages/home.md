\---

title: Home

\---



\# Welcome



This is your new project's homepage. Edit this file to get started.

\## 1. Visão Estratégica: Economia e Eficiência Energética



```sql kpis\_estrategicos

SELECT 

   SUM(economia\_dimerizacao\_brl) AS economia\_total,

   ROUND(AVG(CASE WHEN mttr\_minutos > 0 THEN mttr\_minutos END), 1) AS mttr\_medio,

   COUNT(CASE WHEN status\_alerta != 'Normal' THEN 1 END) AS total\_alertas

FROM fato\_telemetria

```



{% big\_value data="kpis\_estrategicos" value="sum(economia\_total)" title="Economia com Dimerização" fmt="brl2" /%}



{% big\_value data="kpis\_estrategicos" value="avg(mttr\_medio)" title="MTTR Médio (min)" fmt="num1" /%}



{% big\_value data="kpis\_estrategicos" value="sum(total\_alertas)" title="Alertas Críticos Ativos" /%}



```sql economia\_por\_bairro

SELECT 

 d.bairro,

 SUM(f.economia\_dimerizacao\_brl) AS economia\_brl,

 SUM(f.consumo\_kwh) AS consumo\_total\_kwh

FROM fato\_telemetria f

JOIN dim\_dispositivos d ON f.device\_id = d.device\_id

GROUP BY d.bairro

ORDER BY economia\_brl DESC

```



{% bar\_chart 

 data="economia\_por\_bairro" 

x="bairro" 

y="sum(economia\_brl)" 

title="Economia Orçamentária por Bairro (R$)"

y\_fmt="brl2"

/%}



\---



\## 2. Visão Tática: Desempenho de Manutenção (MTTR)



```sql tatico\_manutencao

SELECT 

   d.tipo AS infraestrutura,

   d.bairro,

   ROUND(AVG(f.mttr\_minutos), 1) AS tempo\_medio\_atendimento,

   COUNT(\*) AS ocorrencias

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

   f.data\_hora,

   d.device\_id AS dispositivo,

   d.tipo,

   d.bairro,

   d.latitude,

   d.longitude,

   f.status\_alerta,

   f.nivel\_preenchimento\_pct

FROM fato\_telemetria f

JOIN dim\_dispositivos d ON f.device\_id = d.device\_id

WHERE f.status\_alerta != 'Normal'

```



{% map title="Alertas Críticos por Localização" initial\_position=\[-3.733, -38.510] zoom=12 %}

   {% point\_layer 

       data="alertas\_mapa" 

       lat="latitude" 

       lng="longitude" 

       point\_title="dispositivo" 

       color\_value="status\_alerta"

       tooltip\_fields=\["bairro", "tipo", "status\_alerta"]

   /%}

{% /map %}



{% table data="alertas\_mapa" search=true /%}

