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
