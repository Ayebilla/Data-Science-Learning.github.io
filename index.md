## Identifying Twitter communities: Asimple approach.

Twitter users choose the users they follow and the topics they tweet about carefully. This has led to a growing interest among Data Scientists and business owners trying to identify and categorize users based on their interests so that they may target them with products or issues of interest to them. It has also attracted academicians and they have published many [academic papers](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0210689) outlining different strategies to use in discovering twitter communities. However, many of such papers are overly complex and fail to meet the needs of beginner [Data Scientists](https://www.edureka.co/blog/what-is-data-science/#datascientist). This project seeks to present a very basic and simple approach to identifying twitter communities using [hashtags](https://www.takeflyte.com/blog/hashtags-explained) in users' tweets.

[Hashtags](https://www.takeflyte.com/blog/hashtags-explained) are used in Digital Social Networks to identify the topic of a social media post, start a conversation about an issue or create a channel. Social media campaigners and marketters use them a lot in their posts to draw users attention to their campaigns or products.
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
from mpl_toolkits.mplot3d import Axes3D
```
## Get your Twitter Developer Account
In order for us to be able to get user tweets and profiles from Twitter using the Twitter API, we will need a consumer key, consumer secret, an access token and access token secret. We need these because we will need to authenticate the app that we will be using to get tweets from Twitter. Just in case you are new to APIs, don't worry. [API](https://www.mulesoft.com/resources/api/what-is-an-api) stands for Application Programming Interface. Just like the name suggests, reading it from right to left, think of it this way: "You have an Interface where you can tell a Program(you made) to run an Application(Twitter made) to do some specific task for you." Simple, isn't it?
Before you can get Twitter's programs to run and do something for you, the app you are using to do that must be authenticated. So go ahead and apply for the developer account by following the link provided above if you don't have it already. You need to have a Twitter user account first so go ahead and create one if you don't have one already. I must tell you that getting your developer account approved can take as short as 3 minutes after you have submitted your application or as long as never if you do not do it right. Carefully provide answers to the questions they ask and do not forget to uncheck the boxes that talk about doing something for the government if you don't intend to. By default, those options are ticked so if you leave them as such, Twitter will follow up with emails asking you to answer the same questions you just did. When you get your account approved, login and create an app. Get your credentials and let's take the next step!
## Connect to Twitter

These two functions will help you connect to Twitter and get data. Thanks to this [YouTube video](https://www.youtube.com/watch?v=CVYazfbbcZ8&list=PLmcBskOCOOFW1SNrz6_yzCEKGvh65wYb9&index=18&pbjreload=101)  tutorial that made those functions for us easily. Fill in the spaces with your api credentials.

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

Note that it is best practice to store your credentials as environment variables so you don't expose them to the public. If you need to do that, you will need to import os and sys as well. [This](http://jonathansoma.com/lede/foundations-2019/classes/apis/keeping-api-keys-secret/) may be a good place to look for how to do that.

## Getting the users we want to download their tweets to analyze
To make it interesting, we are going to download tweets of users within a certain geographic location and based on the common hashtags found in their tweets, we can put them into clusters as communities. Start by first identifying geographic boundary you want to identify the communities. You can use your country for example. Go ahead and identify some people in that geographic boundary as seed member. Search their names on Twitter and grab their twitter usernames if they are present on twitter. Drop the '@' symbol, it is not necessary. You can write these names to a csv file to be used late or just keep them in a list within your program. Note that, if there exist such data somewhere, you could just grab it the usernames from there!
It is important that you get as many users as you can. To get our target number of users, we are going to scrawl through twitter and grab the friends i.e people a user is following who are in the same geographic region. For my case, I used 38 seed users and expanded the total to 1038 by grabbing the friends of all 38 users without duplicates. Think of ways to control duplicates in your user list. I handled this by using a python set to store them as set does not allow duplicates. Here is a function to get the names of the user's friends. 

```
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

This function will help us download the tweets of a user as a json file. After you run it, check in your working directory (if you did not specify any) and you will see the json file there! Just like we did with the userFriends function, you will call this code in a loop and pass all the users you want to download their tweets.
```
def getUserTweets(user='RuwaOntheGo'):
    client = TwitterClient()
    fname = "user_timeline_{}.jsonl".format(user)
    with open(fname, 'w') as f:
        for page in Cursor(client.user_timeline, screen_name=user,count=200).pages(2):
            for status in page:
                f.write(json.dumps(status._json) + "\n")
```
## Getting hashtags from the tweets
Now that you have the tweets of all the users you considered, next thing is to search for hashtags of interest in each tweet. For my case, I needed to put these users into clusters based on their interest in **economy, social, culture and health** issues of my country. To do this, I went through twitter and harvested some popular hashtags used by people in my country on twitter. By reviewing people's post with that hashtag and my own knowledge the issue, each hashtag was place in one of the four categories. In all, I used 9 hashtags for each category. See below.

```
htags = {'economy': ['yearofreturn','ghanaianbusiness','localbusiness',
                     'ghanabeyondaid','2020budgetreview',
                     'sdgs','beyondthelockdown','farmersdayghana'], 
         'social':['ghtourism','ghweddings','imwithher','tadigirls',
                   'justiceforgeorgefloyd','girlchildeducation',
                   'lutterodt','hushpuppi','ugandavsghana'], 
         'culture': ['independenceday','ghanamostbeautiful','ghana',
                     'saynotocorruption', 'goodmorningremix','gmb2020',
                     'nsmq','registertovote2020', 'mokobe'],
         'health':['stayhome','socialdistancing','spreadcalmnotfear',
                   'ghanacovid19','wearyourmask','covid19','ghhealth',
                   'globalhealth','staysafe']}

```
Now go through all tweets of a each user and count the occurrences of these hashtags, updaing the count of each category accordingly. I used a python dictionary for this.
```
stats = {'economy': 0, 'social':0, 'culture': 0,'health':0}
```
The entire function for getting hashtags from a user's tweets is shown next

```
def getHashtags(tweet):
    entities = tweet.get('entities',{})
    hashtags = entities.get('hashtags',[])
    return [tag['text'].lower() for tag in hashtags]

def tags(user='RuwaOntheGo', dictTags = {}): #dicTags is a dictionary containing the keywords as keys and the hashtags of interest as a list of values
    stats = {'economy': 0, 'social':0, 'culture': 0,'health':0} #initialize all feature count to zeros.
    labels = ['economy','social','culture','health']  
    fname = "user_timeline_{}.jsonl".format(user)
    with open(fname, 'r') as f:
        hashtags = [] #Counter()
        for line in f:
            tweet = json.loads(line)
            tags_in_tweet = getHashtags(tweet)
            if len(tags_in_tweet)>0:
                hashtags.append(tags_in_tweet)
        for tag in hashtags:
            for tt in tag:
                for key in labels:
                    if tt in dictTags[key]:
                        stats[key] =stats[key] + 1 
    #print(hashtags)
    return {user: list(stats.values())}
                    
```
Just like the other functions, call this function many times to get the hashtag count for each user. 

## Preparing a pandas data frame for your data
If you called the tags() function above many times, storing the result in one big list, you will end up with a list containing dictionaries whose values are a list! Employ your basic python skills here to manipulate it into a form that you can simply convert it to a pandas data frame. I used these functions below to manipulate mine. Go ahead and do it your way!
```
def get_dictkeys(listdic):
    return list(listdic.keys())

#returns the values of the dictionary as a list
def get_dictvals(listdic):
     return list(listdic.values())

def dataframe():
    keysList = []
    valList = []
    data = []
    fkeyList = []
    for item in uList:
        keysList.append(get_dictkeys(item))
        valList.append(get_dictvals(item))
    for it in valList:
        data.append(it[0])
    for n in keysList:
        fkeyList.append(n[0])  
    df = pd.DataFrame(data, columns = ["economy", "social","culture","health"])
    df.insert(0,'user',fkeyList)
    return df
```
Here is how the first 5 entries of my data fram looks like.
user | economy | social | culture | health
---  | ------  | ------ | ------- | ------
sambahflex |0    | 0      |     1   |  0
CoOwusu           0          0      0
AshesiIX | 0  |     0   |       0   |   0 
askrashida | 0    |  0    |      0   |   0
Ghanasoccernet | 0 |  0   |      0   |   0
TransformingGh | 0 |   0   |     29    | 0 

Good job! if you have your data frame ready. Now let's do the last bit of it which is using the K-means to make a cluster of our users. I know you are exhausted already so I will not bore you by explaining what K-means is. You can take a quick read about it [here](https://en.wikipedia.org/wiki/K-means_clustering)

```
X = dff.iloc[:,1:5]
kmeans = KMeans(4)
identified_cluster = kmeans.fit_predict(X)
identified_cluster

data_wt_clusters = dff.copy()
data_wt_clusters['clusters'] = identified_cluster
fig =plt.figure()
#ax = Axes3D(fig, rect=[0, 0, .95, 1], elev=48, azim=134)
#ax = fig.add_subplot(111,projection='3d')
ax = Axes3D(fig, rect=[1, 1, 1, 1], elev=48, azim=200)

x = data_wt_clusters['economy']
y = data_wt_clusters['social']
z = data_wt_clusters['culture']
c = data_wt_clusters['health']

img = ax.scatter(x,y,z, c=c, cmap=plt.hot())
fig.colorbar(img)
plt.show()
```
Here is how my cluster looks like. Go ahead and play with varying K, setting the axes and see how it changes. 
<img src="https://user-images.githubusercontent.com/40761614/88488565-94a62700-cf7d-11ea-9c98-d9af65184678.png" alt="cluster">

## Conclusion
Thank you for making it this far! We have come to end of this tutorial and this is what I have to say;
This was meant present an easy way for beginners to engage and understand how to identify twitter communities and I hope you enjoyed it?  

Before you go, just read the limitations so that you may produce much better results.
This project has limitations as you might have noticed. Firstly, the data set(1038 users) was too small to make meaningful clusters as we want. You can see the effect of that on the cluster graph. Also, notice that I used just a few hashtags which were not chosen based on any principle either than my own discretion to represent each feature. This makes the data prone to biases and not a good representation of the features. I think we can improve this by using more hashtags and asking experts and our friends to help group them into each feature for us.
