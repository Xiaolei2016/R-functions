# **R-functions**
## Get or Set work folder path
```
# get work folder path
getwd()

# set work folder path
setwd("C:/tmp/IR")
```
## Read file in R
```
# read the file directly if the file has already in the work folder path
data <- read.table("textfile.txt", header=FALSE, quote = "", sep="\n")

# read the file
data <- read.csv(file="c:/csvfile.csv", header=TRUE, sep=",")
```
## Transforming Text
```
library(tm)  
corpus <- Corpus(VectorSource(data))
```
## Pre-processing data
```
# tolower: convert text to lower case
cleanData <- tm_map(corpus, tolower)

# removePunctuation: remove punctuation symbols
cleanData  <- tm_map(cleanData , removePunctuation)

# removeWords: remove words like stopwords
cleanData <- tm_map(cleanData, removeWords, stopwords("english"))

# stemDocument: stem Document
cleanData <- tm_map(cleanData, stemDocument, language = "english")

# stripWhitespace: eliminate extra white-spaces
cleanData <- tm_map(cleanData, stripWhitespace)

# removeNumber: remove numbers
cleanData <- tm_map(cleanData, removeNumbers)
```
## Matrix with documents and terms
```
# matrix with documents as rows and terms as columns for the pre-processed data
dtm <- DocumentTermMatrix(cleanData)

# matrix with terms as rows and documents as columns for the pre-processed data
tdm <- TermDocumentMatrix(cleanData)
```
## tf-idf
```
# tf-idf
dtm_tfidf <- weightTfIdf(dtm)
```
## Calculate distance for documents
```
# method can be one of "euclidean", "maximum", "manhattan", "canberra", "binary" or "minkowski"
d_distance <- dist(dtm_tfidf,method = "euclidean")
```
## Cluster documents by using KMeans algorithm
```
# get clusters by using kmeans algorithm
results <- kmeans(d_distance, 3)
```
## Calculate similarity (cosine similarity or jaccard similarity)
```
library(text2vec)
# calculate cosine similarity for all documents
cos_sim = sim2(as.matrix(new_dtm_tfidf), method = "cosine")

# calculate cosine similarity for all documents
jac_sim = sim2(sparse_matrices, method = "jaccard", norm = "l2")
```
## Demo-Cluster sentences then summarize
```
data <- c("Best Toyota dealer in bay area. Drive out with a new car today",
                        "Largest Selection of Furniture. Stock updated everyday" , 
                        " Unique selection of Handcrafted Jewelry",
                        "Free Shipping for orders above $60. Offer Expires soon",
                        "XXXX is where smart men buy anniversary gifts",
                        "2012 Camrys on Sale. 0% APR for select customers",
                        "Closing Sale on office desks. All Items must go" )


library(tm)
library(text2vec)

# set work folder
setwd("C:/tmp/IR")

# represent a collection of text documents
corpus <- Corpus(VectorSource(data))

# Transformations on Corpora
# tolower: convert text to lower case
cleanData <- tm_map(corpus, tolower)

# removePunctuation: remove punctuation symbols
cleanData  <- tm_map(cleanData , removePunctuation)

# removeWords: remove words like stopwords
cleanData <- tm_map(cleanData, removeWords, stopwords("english"))

# stemDocument: stem Document
cleanData <- tm_map(cleanData, stemDocument, language = "english")

# stripWhitespace: eliminate extra white-spaces
cleanData <- tm_map(cleanData, stripWhitespace)

# matrix with documents as rows and terms as columns for the pre-processed data
dtm <- DocumentTermMatrix(cleanData)

# tf-idf
dtm_tfidf <- weightTfIdf(dtm)

# get distance for each document
d_distance <- dist(dtm_tfidf,method = "euclidean")

set.seed(1)
ncluster <- 2
# get clusters by using kmeans algorithm
results <- kmeans(d_distance, ncluster)

clusters <- 1:ncluster
varnames <- paste("cluster", clusters, sep="")
Summaries <- list()

for (i in clusters) {
  
  newCorpus <- list()
  newCleanData <- list()
  
  # get Corpus data for each cluster
  newCorpus <- corpus[results$cluster == i]
  
  # assign cluster name for each cluster
  assign(varnames[i], value = newCorpus)
  
  # get pre-processed data for each cluster
  newCleanData <- cleanData[results$cluster == i]
  
  # matrix with documents as rows and terms as columns for the pre-processed data for each cluster
  new_dtm <- DocumentTermMatrix(newCleanData)
  
  # tf-idf for each cluster
  new_dtm_tfidf <- weightTfIdf(new_dtm)
  
  # calculate cosine similarity for all documents in each cluster
  cos_sim = sim2(as.matrix(new_dtm_tfidf), method = "cosine")
  
  # calculate sum of each column
  col_sums <- colSums(cos_sim)
  
  # get the column which has max sum of column
  col_max <- max.col(t(col_sums), "first")
  
  # get the sentence which has highest cosine similarity in the cluster
  Summaries <- c (Summaries,newCorpus[col_max]$content)
  
  # output all clusters
  for (j in  1:length(newCorpus)){
    cat (varnames[i],":  ",get(varnames[i])$content[j],"\n")
  }
}

# output Summaries
cat ("Summaries: \n",as.String(Summaries))
```
### Output
```
cluster1 :   Best Toyota dealer in bay area. Drive out with a new car today 
cluster1 :   Largest Selection of Furniture. Stock updated everyday 
cluster1 :   Free Shipping for orders above $60. Offer Expires soon 
cluster1 :   XXXX is where smart men buy anniversary gifts 
cluster1 :   2012 Camrys on Sale. 0% APR for select customers 
cluster1 :   Closing Sale on office desks. All Items must go 
cluster2 :    Unique selection of Handcrafted Jewelry 

Summaries: 
 2012 Camrys on Sale. 0% APR for select customers
 Unique selection of Handcrafted Jewelry 
```
