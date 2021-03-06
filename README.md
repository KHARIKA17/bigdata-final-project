# BigData Final project

Below are steps showing how to process data using Spark and Python.

### Source Link:
[https://www.gutenberg.org/cache/epub/33355/pg33355.txt](https://www.gutenberg.org/cache/epub/33355/pg33355.txt)

### Published Databricks Link:
[Databricks Link](https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/566061547951977/1948253970437628/6799487382147066/latest.html)

### Tools and Languages:
- Tools:Databricks.
- Languages:Spark and python.

## Steps Involved:
### Step1:Gathering the data:
1.Importing the data from the source URL and saving it in temporary file.
```
#Fetching the data from source URL
import urllib.request
urllib.request.urlretrieve("https://www.gutenberg.org/cache/epub/33355/pg33355.txt","/tmp/harika.txt")
dbutils.fs.mv("file:/tmp/harika.txt","dbfs:/data/harika.txt")
```
2.Then move the temporary file to dbfs.
```
dbutils.fs.mv("file:/tmp/harika.txt","dbfs:/data/harika.txt")
```
3.Then transfer the file from dbfs to Spark.
```
sc_RDD = sc.textFile("dbfs:/data/harika.txt")
```
### Step2:Cleaning the data:
1.From the data file,we will change the Upper case letters to lower case letters,also removing the spaces and splitting the up the sentence to words.
```
# flatamp each line to words
wordsRDD = sc_RDD.flatMap(lambda line:line.lower().strip().split(" "))
```
2.Next,we will remove the punctuations using the regular expression.For this we need to import re-regular expression library
```
import re
# remove punctutation
clean_tokens_RDD = wordsRDD.map(lambda w: re.sub(r'[^a-zA-Z]','',w))
```
3.Then, we need to filter the data by removing all the stop words from the data and then create a new RDD with the obtained result
```
#prepare to clean stopwords
from pyspark.ml.feature import StopWordsRemover
remove =StopWordsRemover()
stopwords = remove.getStopWords()
clean_words_RDD=clean_tokens_RDD.filter(lambda wrds: wrds not in stopwords)
```
### Step3:Processing the data:
1.We will now prcess the data,as we are done with cleaning the data.Here we map the words to key value pairs.
```
IKVPairsRDD = clean_words_RDD.map(lambda word : (word,1) )
```

2.We will reduce by key to get word count result:
```
# reduceByKey() to get (word, count) results
resultsRDD = IKVPairsRDD.reduceByKey(lambda acc, value: acc+value)
```
3.Next,we will sort the words in descending order and print top 20 most used words.
```
mostused_results = resultsRDD.map(lambda x: (x[1], x[0])).sortByKey(False).take(20)
print(mostused_results)
```
4.Next,we will retrieve all the elements using collect funtion and printing the results.
```
results=resultsRDD.collect()
print(results)
```

### Charting the results:
Here we are going to display the output as in the form of charts,for this we need to import all the required libraries
```
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from collections import Counter

# prepare chart information
source = 'Oahu Travelers guide,by Bill Gleasner,Diana Gleasner'
title = 'Top Words in ' + source
xlabel = 'word'
ylabel = 'count'

# create Pandas dataframe from list of tuples
df = pd.DataFrame.from_records(mostused_results, columns =[xlabel, ylabel]) 
print(df)

# create plot (using matplotlib)
plt.figure(figsize=(10,3))
sns.barplot(xlabel, ylabel, data=df, palette="rainbow").set_title(title)
```
Then we can visualise the data as below:
![https://github.com/KHARIKA17/bigdata_final_project/blob/main/output1.JPG](https://github.com/KHARIKA17/bigdata_final_project/blob/main/output1.JPG)
### Image of Word cloud:
We need to import "Natural Language tool kit" and "word Cloud" libraries to show the highest word count for given the input file.Then, we need to define few functions to process the data and the result will shown in figure.
```
import wordcloud
import nltk
nltk.download('popular')
import matplotlib.pyplot as plt

from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
from wordcloud import WordCloud

class WordCloudGeneration:
    def preprocessing(self, data):
        # convert all words to lowercase
        data = [item.lower() for item in data]
        stop_words = set(stopwords.words('english'))
        # concatenate all the data with spaces.
        paragraph = ' '.join(data)
        # tokenize the paragraph using the inbuilt tokenizer
        word_tokens = word_tokenize(paragraph) 
        # filter words present in stopwords list 
        preprocessed_data = ' '.join([word for word in word_tokens if not word in stop_words])
        print("\n Preprocessed Data: " ,preprocessed_data)
        return preprocessed_data

    def create_word_cloud(self, final_data):
        # initiate WordCloud object with parameters width, height, maximum font size and background color
        # call the generate method of WordCloud class to generate an image
        wordcloud = WordCloud(width=1600, height=800, max_words=10, max_font_size=200, background_color="black").generate(final_data)
        # plt the image generated by WordCloud class
        plt.figure(figsize=(12,10))
        plt.imshow(wordcloud)
        plt.axis("off")
        plt.show()

wordcloud_generator = WordCloudGeneration()
# you may uncomment the following line to use custom input
# input_text = input("Enter the text here: ")
import urllib.request
url = "https://www.gutenberg.org/cache/epub/33355/pg33355.txt"
request = urllib.request.Request(url)
response = urllib.request.urlopen(request)
input_text = response.read().decode('utf-8')

input_text1 = input_text.split('.')
clean_data = wordcloud_generator.preprocessing(input_text1)
wordcloud_generator.create_word_cloud(clean_data)
```
![https://github.com/KHARIKA17/bigdata_final_project/blob/main/output2.JPG](https://github.com/KHARIKA17/bigdata_final_project/blob/main/output2.JPG)

#### References:
- [https://www.tutorialspoint.com/pyspark/pyspark_introduction.htm](https://www.tutorialspoint.com/pyspark/pyspark_introduction.htm)
- [https://github.com/GUNDUPOOJA/Pooja_big_data_project](https://github.com/GUNDUPOOJA/Pooja_big_data_project)
- [https://github.com/Rajeshwari-Rudra/bigData-finalProject](https://github.com/Rajeshwari-Rudra/bigData-finalProject)




