## Identifying Twitter communities: Asimple approach.

Twitter users choose the users they follow and the topics they tweet about carefully. This has led to a growing interest among Data Scientist and business owners trying to identify and categorize users based on their interests so that they may target them with products or issues of interest to them. It has also attracted academicians and they have published many [academic papers](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0210689) outlining different strategies to use in discovering twitter communities. However, many of such papers are overly complex and fail to meet the needs of beginner [Data Scientists](https://www.edureka.co/blog/what-is-data-science/#datascientist). This project seeks to present a very basic and simple approach to identifying twitter communities using [hashtags](https://www.takeflyte.com/blog/hashtags-explained) in users' tweets.

[Hashtags](https://www.takeflyte.com/blog/hashtags-explained) are used in Digital Social Networks to identify the topic of a social media post, start a conversation about an issue or create a channel. Social media campaigners or marketters use them a lot in their posts to draw users attention to their campaigns or products.
## What you need to follow through.
_You need to have python 3 installed or any other platform that will allow you to write python code and install relevant python packages for this tutorial. You also need to have a Twitter Developer Account so that you can obtain the Twitter API credentials to use Twitter API. Some knowledge in writing python codes and working with json files is a plus!_

Step 1: [Install Anaconda](https://www.anaconda.com/products/individual)
Step 2: [Get your Twitter Developer Account](https://developer.twitter.com/en)
Step 3: [Take a review on json format](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON)

## Initial steps and importing of relevant libraries.
Open your python editor of choice ( you can use Jupyter notebook from Anaconda if you installed it) and import the following which we will use.You may add other libraies of your choice if you want to try out different things.
```
from tweepy import API
from tweepy import OAuthHandler
from tweepy import Cursor
import json
from collections import Counter
import pandas as pd
import csv
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
```
## Get your Twitter Developer Account
In order for us to be able to get user tweets and profiles from Twitter using the Twitter API, we will need a consumer key, consumer secret, an access token and access token secret. We need these because we will need to authenticate the app that we will be using to get tweets from Twitter. Just in case you are new to APIs, don't worry. [API](https://www.mulesoft.com/resources/api/what-is-an-api) stands for Application Programming Interface. Just like the name suggests, reading it from right to left, think of it this way: "You have an Interface where you can tell a Program(you made) to run an Application(Twitter made) to do some specific task for you." Simple, isn't it?
Before you can get Twitter's programs to run and do something for you, the app you are using to do that must be authenticated. So go ahead and apply for the developer account by following the link provided above if you don't have it already. You need to have a Twitter user account first so go ahead and create one if you don't have one already. I must tell you that getting your developer account approved can take as short as 3 minutes after you have submitted your application or as long as never if you do not do it right. Carefully provide answers to the questions they ask and do not forget to uncheck the boxes that talk about doing something for the government if you don't intend to. By default, those options are ticked so if you leave them as such, Twitter will follow up with emails asking you to answer the same questions you just did. When you get your account approved, login and create an app. Get your credentials and let's take the next step!

These two functions will help you connect to Twitter and get data. Thanks to this [YouTube video](https://www.youtube.com/watch?v=CVYazfbbcZ8&list=PLmcBskOCOOFW1SNrz6_yzCEKGvh65wYb9&index=18&pbjreload=101)  tutorial that made those functions for us easily. Fill in the spaces with your api credentials.


```# Get user tweets as a json file
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

Note that it is best practice to store your credentials as environment variables so you don't expose them to the public. If you need to do that, you will need to import os and sys as well. [This](http://jonathansoma.com/lede/foundations-2019/classes/apis/keeping-api-keys-secret/) may be a good place to look for how to do that.

## Getting the users we want to download their tweets to analyze
To make it interesting, we are going to download tweets of users within a certain geographic location and based on the common hashtags found in their tweets, we can put them into clusters as communities. Start by first identifying geographic boundary you want to identify the communities. You can use your country for example. Go ahead and identify some people in that geographic boundary as seed member. Search their names on Twitter and grab their twitter usernames if they are present on twitter. Drop the '@' symbol, it is not necessary. You can write these names to a csv file to be used late or just keep them in a list within your program. Note that, if there exist such data somewhere, you could just grab it the usernames from there!
It is important that you get as many users as you can. To get our target number of users, we are going to scrawl through twitter and grab the friends i.e people a user is following who are in the same geographic region. For my case, I used 38 seed users and expanded the total to 1038 by grabbing the friends of all 38 users without duplicates. Think of ways to control duplicates in your user list. I handled this by using a python set to store them as set does not allow duplicates. Here is a function to get the names of the user's friends. 

```# user friends
usrset = set()

def userFriends(user = 'RuwaOntheGo'): #if you call the function without a parameter, the default user is 'RuwaOntheGo'
    client = TwitterClient()
        for friends in Cursor(client.friends_ids,screen_name=user).pages(max_pages): 
            for chunk in paginate(friends, 100):
                users = client.lookup_users(user_ids=chunk)
                for usr in users:
                    if usr.location =='your_location':
                        usrset.add(usr.screen_name) #get the user name of each friend and storing them in a set
            if len(friends) == 400:
                print("More results available. Sleeping for 1 minute to avoid rate limit..")
                time.sleep(120)
```

You can call this function in a for loop and pass your list of users you want to get their friends. For my case, I want to get the friends of the 38 seed users so I call this for each user in that list of 38. Because this function always adds to a global set called usrset, there will be no duplicates!
Go ahead and get as many user names as you want for this exercise if you have a powerful machine. You can use [google colab](https://colab.research.google.com/notebooks/intro.ipynb#recent=true) to save your machine some processing power. Once you are done getting the user names, you may want to write it into csv file or some database for persistence. 
_Hint: segment your input into different list of smaller sizes say, 20 users to avoid delays that may lead to a timed out error. Use different sets to hold the output and you can then unite the sets after you are done to avoid duplicates_

## Downloading the tweets of all the users.
Now that we have the user names of the users we want, we will go ahead and download their tweets. Tweets will be download as json objects and save in a json file. Eeach line in this file represent a tweet object. Remember this as we will be needing it to be able to get hashtags from the tweets. You can visit the Twitter developers site to learn more about the [structure of a tweet](https://developer.twitter.com/en/docs/tweets/data-dictionary/overview/intro-to-tweet-json#:~:text=Each%20Tweet%20has%20an%20author,most%20often%20an%20account%20bio.).
This function will help us download the tweets of a user as a json file

```def getUserTweets(user='AvokaAyebilla'):
    client = TwitterClient()
    fname = "user_timeline_{}.jsonl".format(user)
    with open(fname, 'w') as f:
        for page in Cursor(client.user_timeline, screen_name=user,count=200).pages(2):
            for status in page:
                f.write(json.dumps(status._json) + "\n")
 ```
