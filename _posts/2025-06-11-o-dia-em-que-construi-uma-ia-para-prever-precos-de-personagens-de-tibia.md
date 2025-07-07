---
layout: post
title:  "O Dia em que Construí uma IA para Prever Preços de Personagens de Tibia"
date:   2025-06-11 01:00:00 -0300
categories: [ai, machine-learning]
tags: [pt-BR]
permalink: /posts/o-dia-em-que-construi-uma-ia-para-prever-precos-de-personagens-de-tibia
---

Uma história sobre TCAQS: Tibia Character Automated Quotation System

> English version: [Link](https://www.alvaroinckot.com/posts/the-day-i-built-an-ai-to-predict-tibia-character-prices)

## O início (final de 2022)

Imagine um domingo chuvoso em 2022. Eu estava procrastinando tarefas da vida real navegando pelo novíssimo auction house do [Tibia](https://www.tibia.com/news/?subtopic=latestnews), olhando para chars precificados como propriedades à beira-mar. _Certamente_ existe um padrão aqui, pensei. Quinze anos jogando Tibia haviam programado meu cérebro para achar negócios sub ou supervalorizados.

Então abri o VS Code e escrevi a primeira linha do que viria a ser o TCAQS: `async def crawl(url): ...`  

## Três semanas, 650.000 arquivos HTML, um laptop sobrecarregado

O rate-limiter do Tibia é o equivalente digital a um peso no tornozelo. Três requisições por segundo não é uma sugestão — é a lei. Multiplique isso por meio milhão de páginas e você tem **seis dias** de crawling constante (adicione quedas constantes de rede e algum tempo de inatividade do site do Tibia também). Quando a pasta `results/` finalmente atingiu 650 mil arquivos, compactei-a, dei tapinhas nas costas e... prontamente me mudei para os Estados Unidos.

A vida aconteceu: vistos, caixas, uma onda de calor na Flórida. `tcaqs/` juntou poeira em um disco de backup.

## Redescoberta (Maio de 2024)

Enquanto revia backups. Estou dando uma de Marie-Kondo em drives antigos quando um arquivo de 14 GB rotulado `tibia_auctions_2022.zip` me encara. Does it spark joy? _Claro_. E as ferramentas de AI explodiram desde 2022, então por que não terminar o trabalho?

## Morte por mil <div>s

Analisar aqueles arquivos HTML foi como lutar lidar contra 1.000 cortes de papel. Escrevi um pequeno snippet, **html-scalpel**, que dividiu cada página em um dict, despejou-o em Parquet e fez inserções em bulk no SQLite. Sete horas depois, eu tinha:

- 650 mil linhas
- Mais de 40 campos brutos (level, vocation, outfits, mounts, quest flags...)
- Dois alvos cruciais: `final_bid`, `bid_status` (vendido vs. não-vendido com lance mínimo)

![cpu goes brrrr](https://github.com/alvaroinckot/tcaqs/blob/main/assets/cpu-usage-result.png?raw=true)

## Temporada de modelos: XGBoost vs. meu ego

Prometi a mim mesmo **90% R²** em dados de teste, o suficiente para passar em meu próprio teste de olfato Tibiano. Veja como isso se desenrolou:

### v1: Baseline e Checagem de Realidade

Comecei com as estatísticas óbvias: level, vocation, world type e um punhado de outros atributos numéricos, trinta colunas no total. Uma rápida busca em grade no XGBoost básico empurrou o R² de treinamento para além de 0,94, mas o R² de teste estacionou em 0,89 enquanto o MAPE oscilava por toda parte, especialmente em personagens de baixo level. O modelo estava memorizando o conjunto de treinamento e aprendendo pouco sobre preços do mundo real, útil apenas como base para trabalhos futuros.

### v2: Empacotando o Lore

Em seguida, adicionei as features que realmente impulsionam os preços do Tibia, como quest flags, mounts raros, outfits, hirelings e server type, todos codificados one-hot e reequilibrados. O ajuste de hiperparâmetros elevou o R² de teste para 0,94 e suavizou previsões para chars de médio a alto level. A nova fraquesa, no entanto, eram personagens de baixo level em servidores recém-abertos. Eles começam superfaturados porque os early adopters têm excesso de TC investido, depois caem rapidamente à medida que a economia do servidor amadurece, então seus valores reais flutuam mais rápido do que minha coleta estática poderia capturar, mantendo o MAPE alto nessa fatia.

### v3: Eliminando um problema crucial

O verdadeiro avanço veio de dados mais limpos, não de features extras. Muitos auctions terminando no lance mínimo não tinham comprador, então seus preços listados eram apenas um chute. Filtrando essas linhas, e reajustando o XGBoost, a curva de erro se achatou: o MAPE geral caiu para 0,42 (abaixo de 0,30 para chars abaixo de 100) e o R² de teste estabilizou em 0,935. O CatBoost pontuou quase o mesmo, mas consumiu mais VRAM, então XGBoost ainda é o campeão por enquanto.

![Top Features](https://github.com/alvaroinckot/tcaqs/blob/main/assets/top-features.png?raw=true)

## Alguns resultados

A evolução da engenharia de features em números:

| Model       | Split    | MSE        | RMSE     | MAE      | MAPE   | R2     | Explained_Variance|
|-------------|----------|------------|----------|----------|--------|--------|-------------------|
| results-v1  | Training | 2289526.25 | 1513.12  | 703.37   |        | 0.9393 |                   |
| results-v1  | Testing  | 4209464.5  | 2051.7   | 857.07   |        | 0.8874 |                   |
| results-v2  | Training | 543840.75  | 737.46   | 376.31   |        | 0.9856 |                   |
| results-v2  | Testing  | 2172790.25 | 1474.04  | 582.8    |        | 0.9419 |                   |
| results-v2-es | Training | 333324.59 | 577.3427 | 315.0486 | 0.4379 | 0.9912 | 0.9912           |
| results-v2-es | Testing  | 2140074.75 | 1462.8994 | 577.0405 | 0.5454 | 0.9427 | 0.9427         |
| results-v3  | Training | 49717.34   | 222.9739 | 149.1993 | 0.3372 | 0.9954 | 0.9954            |
| results-v3  | Testing  | 725773.25  | 851.9233 | 358.3552 | 0.4243 | 0.9349 | 0.9349            |

## Uma bola de cristal ao vivo

- **Demo**: <https://huggingface.co/spaces/alvaroinckot/tcaqs>  
- **Code**: <https://github.com/alvaroinckot/tcaqs>

Preencha as informações do personagem e obtenha uma estimativa. Sem mais adivinhações antes de gastar.

## O que vem a seguir?

1. **Nova coleta** – Nova vocation (Monk) e gold ajustado à inflação significa que o snapshot de 2022 está envelhecendo. Um crawler distribuído em SQS (ou qualquer outra fila) deve reduzir o scraping de semanas para dias.
2. **Contexto de mercado** – Injetar CPI de level mundial para que o modelo acompanhe a inflação do gold.
3. **Competição de modelos** – LightGBM, TabPFN, talvez um pequeno transformer tabular — porque não?
4. **Dashboard público** – Índice diário de preço justo por vocation/world.

## XP final

Lançar o **TCAQS** me lembrou que o glamour da AI repousa em fundações nada glamorosas: scraping obstinado, limpeza implacável e muitas barras de espera. Mas unir duas paixões de longa data: Tibia e machine learning, valeu cada `<div>`.

