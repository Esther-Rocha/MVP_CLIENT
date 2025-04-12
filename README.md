# Yelp Review Pipeline com Databricks

Este projeto tem como objetivo construir um pipeline de dados utilizando a plataforma em nuvem **Databricks**, focando na análise de avaliações de usuários de estabelecimentos presentes no **Yelp Open Dataset**.

---

## Objetivo

Analisar os reviews de clientes em estabelecimentos comerciais (com foco em restaurantes) a partir das seguintes perguntas:

1. Quais os tipos de estabelecimentos mais bem avaliados em diferentes regiões?
2. Quais palavras-chave aparecem com maior frequência em avaliações positivas e negativas?
3. Há correlação entre número de avaliações e nota média?
4. A localização influencia na nota?
5. Quais estabelecimentos se destacam acima da média?

---

##  Fonte de Dados

- **Yelp Open Dataset**  
  [https://www.yelp.com/dataset](https://www.yelp.com/dataset)

Arquivos utilizados:
- `business.json`
- `review.json`
- `user.json`

---

## Pipeline

O pipeline foi implementado em notebooks Databricks divididos nas seguintes etapas:

### 1. Coleta
Leitura dos arquivos JSON no DBFS.

### 2. Transformação
- Foco na categoria "Restaurants"
- Criação de modelo Estrela (fato_review + dimensões)
- Normalização de dados e eliminação de nulos

### 3. Carga
- Escrita em formato Delta Table no Databricks

### 4. Análise
- SQL e PySpark para responder às perguntas
- WordClouds para analisar sentimentos nos textos

---

## 📊 Tecnologias

- Apache Spark (via Databricks)
-  Python 3
- Pandas, PySpark, Matplotlib, WordCloud
- SQL (Databricks SQL)

---




# MVP_CLIENT
df_business = spark.read.json("/FileStore/yelp/business.json")
df_review = spark.read.json("/FileStore/yelp/review.json")

# Filtrando apenas restaurantes
df_restaurants = df_business.filter(df_business.categories.contains("Restaurants"))

# Juntando com reviews
df_joined = df_review.join(df_restaurants, "business_id")

# Criando tabela Delta
df_joined.write.format("delta").mode("overwrite").saveAsTable("fato_reviews")


SELECT categories, state, ROUND(AVG(stars),2) as avg_rating
FROM fato_reviews
GROUP BY categories, state
ORDER BY avg_rating DESC

from wordcloud import WordCloud
# Separar por estrelas >=4 e <=2 e gerar nuvem de palavras

df_user_review = df_review.groupBy("user_id").agg(avg("stars").alias("avg_rating"), count("*").alias("review_count"))
df_user_review.corr("avg_rating", "review_count")

SELECT city, ROUND(AVG(stars), 2) as avg_rating, COUNT(*) as num_reviews
FROM fato_reviews
GROUP BY city
ORDER BY avg_rating DESC

SELECT name, city, ROUND(AVG(stars), 2) as avg_rating, COUNT(*) as num_reviews
FROM fato_reviews
GROUP BY name, city
HAVING COUNT(*) > 30 AND AVG(stars) > 4.5
ORDER BY avg_rating DESC
