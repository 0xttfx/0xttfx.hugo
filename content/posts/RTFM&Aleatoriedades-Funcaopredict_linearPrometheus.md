---
title: "RTFM & Aleatoriedades - Função  predict_linear() do Prometheus"
date: 2024-11-27T21:59:52-03:00
#layout: "simple"
# weight: 1
# aliases: ["/first"]
tags: ["Linux", "DevOps", "SRE", "Observabilidade", "Prometheus", "PromQL", "Alertmanager" ]
author: "Faioli a.k.a 0xttfx"
toc: true
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: ""
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

A função [predict_linear()](https://prometheus.io/docs/prometheus/latest/querying/functions/#predict_linear) do Prometheus permite prever o valor futuro de uma série temporal com base em uma regressão linear simples dos dados históricos. Isso é especialmente útil para antecipar tendências e comportamentos futuros de métricas monitoradas, auxiliando na detecção proativa de possíveis problemas.

>[!TIPS]
>predict_linear() deve ser usado somente com medidores

- `predict_linear(v range-vector, t scalar)`
  - prevê o valor das séries temporais `t` segundos a partir de agora, com base no vetor de intervalo `v`, usando [regressão linear simples](https://en.wikipedia.org/wiki/Simple_linear_regression). O vetor de intervalo deve ter pelo menos duas amostras para realizar o Cálculo. Quando `+Inf`ou a `-Inf`são encontrados no vetor da faixa, o valor de inclinação e deslocamento calculado será `NaN`
    - Em [estatística](https://en.wikipedia.org/wiki/Statistics "Estatísticas"), **a regressão linear simples** (**SLR**) é um modelo [de regressão linear](https://en.wikipedia.org/wiki/Linear_regression) com uma única [variável explicativa](https://en.wikipedia.org/wiki/Covariate). Ou seja, diz respeito a pontos amostrais bidimensionais com [uma variável independente e uma variável dependente](https://en.wikipedia.org/wiki/Dependent_and_independent_variables) (convencionalmente, as coordenadas *x* e *y* em um [sistema de coordenadas cartesianas](https://en.wikipedia.org/wiki/Cartesian_coordinate_system)) e encontra uma função linear (uma [linha reta](https://en.wikipedia.org/wiki/Straight_line) não vertical) que, tão precisamente quanto possível, prevê os valores das variáveis dependentes como uma função da variável independente.

## Tá, mas e na prática?

Deixa eu tentar explicar de forma pratica essa paradinha

- Essa é a sintaxe que precisamos ter em mente:

``` promql
predict_linear(vetor_intervalo, tempo_futuro_em_segundos)
```

- Onde:
  - `vetor_intervalo` é um range vector que representa a `time series` ao longo de um intervalo de tempo específico.
  - `tempo_futuro_em_segundos` é o escalar que indica quantos segundos no futuro você deseja prever o valor.

Agora suponha uma situação na qual seja necessário prever quando o espaço livre em um sistema de arquivos de um host servidor se esgotará, com base no fluxo de dados das últimas horas.

Agora que temos uma caso de uso, vamos criar uma 2 expressões que antecipe esse momento, com a intenção de ter tempo hábil para ações preventivas antes que o problema pata na nossa porta...

- 1 expressão:
  - a

  ```promql
  predict_linear(node_filesystem_free_bytes[1h], 4 * 3600) < 10737418240
  ```

  - b

  ```promql
  predict_linear(node_filesystem_free_bytes[1h], 4 * 3600) < 10 * 10^9
  ```

  - c

  ```promql
  predict_linear(node_filesystem_free_bytes[1h], 4 * 3600) < 10 * 1024^3
  ```
- 2 expressão:

  ```promql
  predict_linear(node_filesystem_free_bytes[1h], 4 * 3600) < 10
  ```

- Onde:
  - `node_filesystem_free_bytes[1h]` obtém os dados de espaço livre no disco nos últimos 60 minutos.
  - `4 * 3600` define o intervalo de 4 horas (4 * 3600 segundos).
  - `10 * 1024^3` ou `10 * 10^9` definem o limite de 10 GB ou `< 10` que define o limite de 10 bytes 
    - Para representar o valor mínimo de **10 GB** na expressão [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/), deve-se usar bytes como unidade base. E como 1 GB equivale a **1.073.741.824 bytes**, para um valor mínimo de **10 gigabytes (GB)**, seria:
      - para a unidade padrão
        - **10737418240** (10 * 1.073.741.824 = 10.737.418.240 bytes)
      - para a unidade simplificada:
        - **10 * 1024^3** resultando em: 10.737.418.240 bytes (1 GB como 1024\^3 bytes).
      - para a unidade simplificada, caso seja preciso uma valor mai exato:
        - **10 * 10^9** resultando em: **10.000.000.000** 10^9 bytes (1 GB = 1.000.000.000 bytes).
      - para representar uma valor minimo menor de **10 bytes**
        - **< 10**

Com essas expressões estamos verificando se a previsão do espaço livre no disco será menor que **10 GB** ou **10 bytes** dentro de 4 horas. E sendo verdade, com base na tendência atual da aferição, um alerta será acionado, informando que o espaço livre em disco estará abaixo do limite de 10 GB ou 10 bytes no futuro próximo de 4 horas...

Dando continuidade com o nosso cenário suposto. Vamos implementar um alarme usando nossas expressões

- Alarme para quando menor que**10 GB**
  - lembrando que podemos usar **`< 10 * 1024^3`** ou **`< 10737418240`** ou **`< 10 * 10^9`** conforme nossa necessidade de assertividade...
   
```yaml
groups:
- name: disk_alerts
  rules:
  - alert: SistemaDeArquivosCriticamenteBaixoEm4Horas
    expr: predict_linear(node_filesystem_free_bytes[1h], 4 * 3600) < 10 * 1024^3
    for: 10m
    labels:
    severity: warning
    annotations:
    summary: "A treta vai ser doida em 4 horas"
    description: "O sistema de arquivos no servidor {{ $labels.instance }} terá menos de 10 GBytes disponíveis em 4 horas."
```

- Alarme para quando menor que **10 bytes**

```yaml
groups:
- name: disk_alerts
  rules:
  - alert: SistemaDeArquivosCriticamenteBaixoEm4Horas
    expr: predict_linear(node_filesystem_free_bytes[1h], 4 * 3600) < 10
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: "A treta vai ser criticamente doida em 4 horas"
      description: "O sistema de arquivos no servidor {{ $labels.instance }} terá menos de 10 bytes disponíveis em 4 horas."
```

- Onde:
  - `alert`: nome do alerta.
  - `expr`: expressão que utiliza `predict_linear()` para prever o esgotamento do espaço em disco.
  - `for`: período durante o qual a condição deve ser verdadeira antes de acionar o alerta, para evitar falsos positivos.
  - `labels`: rótulos adicionais para categorizar o alerta.
  - `annotations`: informações adicionais que podem ser incluídas na notificação do alerta.

Essa e o método que me permitiu sair dos alarmes de limiares fixos que não atendem de maneira satisfatória crescimentos tão velozes que, no momento do disparo do do alerta, o tempo é tao curto ou tarde demais para uma atuação técnica...

