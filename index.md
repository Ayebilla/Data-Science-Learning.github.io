## Identifying Twitter communities: Asimple approach

Twitter users choose the users they follow and the topics they tweet about carefully. This has led to a growing interest among Data Scientist and business owners trying to identify and categorize users based on their interests so that they may target them with products or issues of interest to them. It has also attracted academicians and they have published many [academic papers](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0210689) outlining different strategies to use in discovering twitter communities. However, many of such papers are overly complex and fail to meet the needs of beginner [Data Scientists](https://www.edureka.co/blog/what-is-data-science/#datascientist). This project seeks to present a very basic and simple approach to identifying twitter communities using [hashtags](https://www.takeflyte.com/blog/hashtags-explained) in users' tweets.

[Hashtags](https://www.takeflyte.com/blog/hashtags-explained) are used in Digital Social Networks to identify the topic of a social media post, start a conversation about an issue or create a channel. Social media campaigners or marketters use them a lot in their posts to draw users attention to their campaigns or products.

_You need to have python 3 installed or any other platform that will allow you to write python code and install relevant python packages for this tutorial. You also need to have a Twitter Developer Account so that you can obtain the Twitter API credentials to use Twitter API. Some knowledge in writing python codes and working with json files is a plus!_

Step 1: [Install Anaconda](https://www.anaconda.com/products/individual)
Step 2: [Get your Twitter Developer Account](https://developer.twitter.com/en)
Step 3: [Take a review on json format](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON)

Now that you are set, lets get started!

Open your python editor of choice ( you can use Jupyter notebook from Anaconda if you installed it) and import the following which we will use.You may add other libraies of your choice if you want to try out different things.

```from tweepy import API
from tweepy import OAuthHandler
from tweepy import Cursor
import json
from collections import Counter
import pandas as pd
import csv
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
```
In order for us to be able to get user tweets and profiles from Twitter using the Twitter API, we will need a consumer key, consumer secret, an access token and access token secret. We need these because we will need to authenticate the app that we will be using to get tweets from Twitter. Just in case you are new to APIs, don't worry. [API](https://www.mulesoft.com/resources/api/what-is-an-api) stands for Application Programming Interface. Just like the name suggests, reading it from right to left, think of it this way: "You have an Interface where you can tell a Program(you made) to run an Application(Twitter made) to do some specific task for you." Simple, isn't it?
Before you can get Twitter's programs to run and do something for you, the app you are using to do that must be authenticated. So go ahead and apply for the developer account by following the link provided above if you don't have it already. You need to have a Twitter user account first so go ahead and create one if you don't have one already. I must tell you that getting your developer account approved can take as short as 3 minutes after you have submitted your application or as long as never if you do not do it right. Carefully provide answers to the questions they ask and do not forget to uncheck the boxes that talk about doing something for the government if you don't intend to. By default, those options are ticked so if you leave them as such, Twitter will follow up with emails asking you to answer the same questions you just did. When you get your account approved, login and create an app. Get your credentials and let's take the next step!

These two functions will help you connect to Twitter and get data. Thanks to [this](https://www.youtube.com/watch?v=CVYazfbbcZ8&list=PLmcBskOCOOFW1SNrz6_yzCEKGvh65wYb9&index=18&pbjreload=101) YouTube video tutorial that made those functions for us easily. Fill in the spaces with your api credentials.


```
def getTwitterOAuth():
    consumer_key = ''
    consumer_secret = ''
    access_token = ''
    access_token_secret = ''
    auth = OAuthHandler(consumer_key, consumer_secret)
    auth.set_access_token(access_token,access_token_secret)
    return auth

def TwitterClient():
    auth = getTwitterOAuth()
    client = API(auth)
    return client
```

Note that it is best practice to store your credentials as environment variables so you don't expose them to the public. If you need to do that, you will need to import os and sys as well. 

