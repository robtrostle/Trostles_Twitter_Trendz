![](https://www.freepnglogos.com/uploads/twitter-logo-png/twitter-logo-with-birds-symbol-icon-24.png)

<span style="font-family:Trebuchet MS; font-size:3em; color:#fdd700; text-shadow: 3px 3px #000">Trostle's Twitter Trendz</span>

### <font color = "blue">Question: </font><font color="green">What topic is trending in the world, what's the sentiment, and where are they tweeting from?<br><br><font color = "red">Outputs: <font color = "dark red">word cloud, world tweet map, tweet locations csv file</font>


```python
#Book Citation/Reference:
""" Title: Intro to Python for Computer Science and Data Science
Author: Paul Deitel, Harvey Deitel
(C) Copyright 2019 by Deitel & Associates, Inc. and
Pearson Education, Inc. All Rights Reserved."""
```




    ' Title: Intro to Python for Computer Science and Data Science\nAuthor: Paul Deitel, Harvey Deitel\n(C) Copyright 2019 by Deitel & Associates, Inc. and\nPearson Education, Inc. All Rights Reserved.'



<h2><font color=REBECCAPURPLE>I. Import Tweepy & Access Keys</font><h2>


```python
#Tweepy is a Python library for accessing the Twitter API.
import tweepy
import keys
auth = tweepy.OAuthHandler(keys.consumer_key, keys.consumer_secret)
auth.set_access_token(keys.access_token, keys.access_token_secret)
api = tweepy.API(auth, wait_on_rate_limit=True,
                  wait_on_rate_limit_notify=True)
#we'll need the print_tweets function so we can see the tweets. 
from tweetutilities import print_tweets
```

<h2><font color=REBECCAPURPLE>II. Quick API Search Examples</font><h2>

### Let’s get starteed by searching for three recent tweets about Pittsburgh. The search method’s q keyword argument specifies the query string, which indicates what to search for and the count keyword argument specifies the number of tweets to return:


```python
tweets = api.search(q='Pittsburgh', count=3)
```


```python
print_tweets(tweets)
```

    sweetbabette: I just looked at the forecast for Pittsburgh over Christmas and YIKES. https://t.co/oATpIXqzZU
    
    MsAWeeks: Looking forward to how the Pittsburgh Cycle continues to play out in film  https://t.co/WfUAKOBHTo
    
    Heather_Werner: RT @DaveDiCello: If we don't have another snowstorm this season in #Pittsburgh, I may have enough snow photos from last week to last me unt…
    
    

### Let's try searching the latest tweets by Hashtag!


```python
#search by using hashtags
covid = api.search(q='#covid19vaccine', count=2)
print_tweets(covid)
```

    loudonCorkOne: RT @PHE_uk: Our specialist freezers have been storing the #COVID19vaccine at secure locations, ready for distribution to #NHS hubs across t…
    
    aminaibr_lon: RT @PHE_uk: Our specialist freezers have been storing the #COVID19vaccine at secure locations, ready for distribution to #NHS hubs across t…
    
    

<h2><font color=REBECCAPURPLE>III. What's Trending?!</font><h2>

### The Tweepy API’s trends_available method calls the Twitter API’s trends/available19 method to get a list of all locations for which Twitter has trending topics. Method trends_available returns a list of dictionaries representing these locations. 


```python
trends_available = api.trends_available()
len(trends_available)
```




    467




```python
#The dictionary in each list element returned by trends_available has various information, including the location’s name and woeid
trends_available[1]
```




    {'name': 'Winnipeg',
     'placeType': {'code': 7, 'name': 'Town'},
     'url': 'http://where.yahooapis.com/v1/place/2972',
     'parentid': 23424775,
     'country': 'Canada',
     'woeid': 2972,
     'countryCode': 'CA'}



### The Twitter Trends API’s trends/place method uses Yahoo! Where on Earth IDs (WOEIDs) to look up trending topics. The WOEID 1 represents worldwide.

### Let’s get today’s worldwide trending topics (your results will differ):

### Method trends_place returns a one-element list containing a dictionary. The dictionary’s 'trends' key refers to a list of dictionaries representing each trend:


```python
world_trends = api.trends_place(id=1)
```


```python
trends_list = world_trends[0]['trends']
```


```python
trends_list[0]
```




    {'name': '#النصر_الاتفاق_دوري_المحترفين',
     'url': 'http://twitter.com/search?q=%23%D8%A7%D9%84%D9%86%D8%B5%D8%B1_%D8%A7%D9%84%D8%A7%D8%AA%D9%81%D8%A7%D9%82_%D8%AF%D9%88%D8%B1%D9%8A_%D8%A7%D9%84%D9%85%D8%AD%D8%AA%D8%B1%D9%81%D9%8A%D9%86',
     'promoted_content': None,
     'query': '%23%D8%A7%D9%84%D9%86%D8%B5%D8%B1_%D8%A7%D9%84%D8%A7%D8%AA%D9%81%D8%A7%D9%82_%D8%AF%D9%88%D8%B1%D9%8A_%D8%A7%D9%84%D9%85%D8%AD%D8%AA%D8%B1%D9%81%D9%8A%D9%86',
     'tweet_volume': 43977}




```python
#use a list comprehension to filter the list so that it contains only trends with more than 10,000 tweets:
trends_list = [t for t in trends_list if t['tweet_volume']]
```


```python
#Next, let’s sort the trends in descending order by
from operator import itemgetter
trends_list.sort(key=itemgetter('tweet_volume'), reverse=True)
```


```python
#display the names of the top five trending topics:
for trend in trends_list[:5]:
    print(trend['name'])
```

    Nintendo
    الرائد
    Pearl Harbor
    #TDYAwards
    #ArtistoftheYear
    


```python
#Grab the number one trending topic from the trends lists and print it as a string
print(trends_list[0]['name'])
```

    Nintendo
    


```python
#pass the number one trending topic string into the query search and print the two most recent tweets. 
number_one = api.search(q=trends_list[0]['name'], count=2)
print_tweets(number_one)
```

    Scanman241: https://t.co/CoezmlVkG8 this is really sad, Nintendo wtf happened?
    
    senpai_elder: RT @AhavaPneumaChan: If only Nintendo was actually like the vision Iwata had for the company. https://t.co/9OCQdWqVFO
    
    

<h2><font color=REBECCAPURPLE>III. Export to a Word Cloud!</font><h2>


```python
from wordcloud import WordCloud
from IPython.display import Image, HTML
```


```python
#create a word cloud
topics = {}
for trend in trends_list:
    topics[trend['name']] = trend['tweet_volume']
```

### Topics dictionary’s key–value pairs, then output the word cloud to the image file TrendingTwitter.png. The argument prefer_horizontal=0.5 suggests that 50% of the words should be horizontal, though the software may ignore that to fit the content:


```python
wordcloud = WordCloud(width=1600, height=900,
    ...:     prefer_horizontal=0.5, min_font_size=10, colormap='prism',
    ...:     background_color='white')
    ...:
```


```python
wordcloud = wordcloud.fit_words(topics)
```


```python
wordcloud = wordcloud.to_file('World_TrendingTwitter.png')
```


```python
Image('World_TrendingTwitter.png')
```




![png](output_34_0.png)



<h2><font color=REBECCAPURPLE>IV. Listening for tweets in a Stream.</font><h2>

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQFIJAvAg8OEn3BOCu5qsN6mpB9AuRV6LVk2A&usqp=CAU)


```python
from tweetlistener import TweetListener
```


```python
tweet_listener = TweetListener(api)
```


```python
tweet_stream = tweepy.Stream(auth=api.auth,
   ...:                              listener=tweet_listener)
   ...:
```

### The Stream object’s filter method begins the streaming process. Let’s track tweets about the number one trend. Here, we use the track parameter to pass a list of search terms:


```python
tweet_stream.filter(track=[trends_list[0]['name']], is_async=True)
```


```python
tweet_stream.running=False #this will terminate the stream. 
```

#### Without the is_async=True argument, filter initiates a synchronous tweet stream. In this case, IPython would display the next In [] prompt after the stream terminates.

<h2><font color=REBECCAPURPLE>V. Determine the sentiment of this trend. Negative? Positive? Neither?</font><h2>


```python
from sentimentlistener import SentimentListener
import sys
import sentimentlistener
```


```python
# create the StreamListener subclass object
#listen and pull in the last 10 tweets
sentiment_dict = {'positive': 0, 'neutral': 0, 'negative': 0}#keeps tally of the results in a dict. 
sentiment_listener = SentimentListener(api, sentiment_dict, trends_list[0]['name'], 10)#last 10 tweets. Can be edited. 
```


```python
# set up Stream 
stream = tweepy.Stream(auth=api.auth, listener=sentiment_listener)
```


```python
# start filtering English tweets containing search_key
stream.filter(track=[trends_list[0]['name']], languages=['en'], is_async=False)
```

    Connection successful
    
    - Scanman241: this is really sad, Nintendo wtf happened?
    
      jwaNINER: Nintendo, please chill.
    
      bigwhitecockult: Nintendo, make it canon that Link has nipple piercing. This is your final warning
    
      Baxon__: Satoru Iwata would be ashamed @NintendoAmerica #boycottnintendo #savemelee #freemelee #nintendohatesyou
    
    + Katzedan: tfw I loved that PS4 cover but man, Nintendo really can be super based &lt;3
    
    - P1kaTyler: yknow id love to celebrate smash's 2 year aniversary. but the timing couldnt be worse. fuck you nintendo
    
    + LambdaCore5: @OrangeofJuice @Michael_Brown89 @jultar1000 @huntgbunt Nintendo is most definitely old fashioned, they're still stuck in the late 2000s.
    
    - Dominik_Kocka: @Saberspark Nintendo fans denying the slow death of Nintendo
    
    + littlesmegger: @BoundaryBreak Yeah Nintendo didn’t make this claim by looks of it so as much as they’re an easy target right now s…
    
      Yvng_Hov_remix: @Lock409 @InfernoOmni Nintendo been frauds tbh
    
    - ManMalding: @paciencia_diogo @JakobFel even nintendo still crunches (despite a pledge to stop) have fun never playing a video game
    
    


```python
print("Tweet sentiment for ",trends_list[0]['name'])
print('Positive:', sentiment_dict['positive'])
print(' Neutral:', sentiment_dict['neutral'])
print('Negative:', sentiment_dict['negative'])
```

    Tweet sentiment for  Nintendo
    Positive: 3
     Neutral: 4
    Negative: 4
    


```python
from tweetutilities import get_API
api = get_API()
```

<h2><font color=REBECCAPURPLE>VI. Where in the world are they tweeting from?!</font><h2>

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSlgwzcGeDhiEj7oc8PORJ11AEuvsC0cCVuBw&usqp=CAU)

### Our LocationListener class requires two collections: A list (tweets) to store the tweets we collect and a dictionary (counts) to track the total number of tweets we collect and the number that have location data:


```python
tweets = []
counts = {'total_tweets': 0, 'locations': 0}
```


```python
from locationlistener import LocationListener
```


```python
location_listener = LocationListener(api, counts_dict=counts,
   ...: tweets_list=tweets, topic=trends_list[0]['name'], limit=50)
   ...:
```

### The LocationListener will use our utility function get_tweet_content to extract the screen name, tweet text and location from each tweet, place that data in a dictionary.


```python
stream = tweepy.Stream(auth=api.auth, listener=location_listener)
stream.filter(track=[trends_list[0]['name']], languages=['en'], is_async=False)
```

    NERUPLUSHIE: mutliple??? nintendo are you ok https://t.co/roYTPTNQZ1
    
    bigboipascal: @OnThisDayGaming Sakurai and Sora inc. deverses praise, fuck nintendo though
    
    Diam_ond10: @Pincan18 @RouGhBartL1 @_EyeAm @PoorlyAgedStuff This has nothing to do with that. Nintendo didn't pull anything aft… https://t.co/8zwvafBj7C
    
    KelvinA313: @PaulTassi @Forbes Nintendo needs to get their heads out of their asses.
    
    honeyspacetrash: @Saberspark Nah, this is big facts and the main issue I have with them as a Nintendo fan.
    
    DunkCheckston: @tydanskrrtskrrt HELLA FOUL FOR THAT NINTENDO
    
    SuperNelsdoHD: @Cibles_ Unless Nintendo tried to take down CTGP videos or Wiimmfi videos that doesn't necessary mean CTGP, or Wiim… https://t.co/4zFKcfzLc2
    
    StewpsLoL: Why does Nintendo hate us :(
    
    makromolecule: nintendo makes such good games that they think they can’t fail and honestly they’re sort of right
    
    Amysel_JinkusuP: @Saberspark I wouldn't say Nintendo is ruined, its just that Nintendo's policies are absolutely enforced whatever t… https://t.co/uDbYDCMxEU
    
    helladventurers: LRT i hate the realization that as much as I love their game, Nintendo as a company are a gross pile of shit but, y… https://t.co/Lo7kUiirsw
    
    BlakeUnleashed: glad to see nintendo celebrating it by cancelling a chairty project and 2 fan tournaments for supporting melee
    
    bigwhitecockult: Nintendo, make it canon that Link has nipple piercings. This is your final warning
    
    Kn1vezz: i want a nintendo switch so bad
    
    Im_Just_Storo: Fun fact: Nintendo is more greedy than Squeak Squad!
    
    ...Now excuse me while I celebrate Super Smash Bros. 2nd Anniversary. Good day.
    
    VictsWasTaken: @DLilwilly nintendo gud when they release a console
    
    PiCubed3: @IntroSpecktive remember that even though Nintendo is doing shitty things rn doesn’t mean we can support the develo… https://t.co/DqpsQMH9W9
    
    inkiestgrove: might reconcile the joy i feel towards nintendo games and the spite i feel towards nintendo as a modern company by… https://t.co/Lp9Z3llqul
    
    clothmanz: @MarcheX800 It’s not even specifically executives, it’s mostly Nintendo of America’s legal team
    
    Thedoor3dark: @Terry_S_S @ahoodatheguy @DSzostkiewicz @Sora_Sakurai Nintendo also has gay characters dumb ass. Guess you can’t pl… https://t.co/TtuDkyI0es
    
    thegeektcmyt1: @KingOfGuuh better roast those pussy ass nintendo niggas
    
    FalKoopa_: @XinguIarity Nintendo was shutting down fan projects and stuff even when Iwata was in charge.
    
    I don't think what w… https://t.co/YWjN9inqW4
    
    Aveliix: say it with me
    FUCK NINTENDO
    
    doofus_dr: @_Solid @Cptn_Alex That's sweet! Still shitty what Nintendo did but hopefully the campaign can continue!
    
    etheraltxt: what did Nintendo do this time
    
    the_comming: Nintendo after someone buys a switch from Walmart. despite Walmart not being owned by Nintendo https://t.co/Um8eW6lxYG
    
    beefbottom: i think imma treat myself to a nintendo switch
    
    NintendoSwitch2: Nintendo Serves Cease and Desist to Your Friend that Pronounces it “May-Rio” https://t.co/ExddLn3HGL
    
    splattown: @AlfredMarcelo @Scykoh Like yea I agree with you people should be protective of their work but Nintendo does it to the nth degree
    
    radarssbm: Fully respect people who make the choice to boycott Nintendo and not buy their games, but also fully support people… https://t.co/V4pnfaBWYp
    
    cmath192: Retweet to scare Nintendo https://t.co/kCkrz0ew5f
    
    lex288: I don't know who's salt is more entertaining, Nintendo's at the Splatoon tournament or Cyberpunk fans getting up in… https://t.co/LrSg8BQk9E
    
    MyOhMyke: @BoundaryBreak I don't think Nintendo had anything to do with the claim; it's from "Universal Studios" and I wouldn… https://t.co/7pQzPU0pIc
    
    KaraCorvus: @Saberspark I love Nintendo, but they have had some pretty bad takes in recent history.
    
    dealingindeals: Kohl’s #ad : Nintendo Switch Lite &amp; Super Smash Bros Bundle $270 + $60 Kohl’s Cash https://t.co/pkEGGTyMEB
    
    beeeeemboyNEW: @UnashamedRebel Nintendo sent a cease and desist to a TO for The Big House tournament, for using a mod called Slipp… https://t.co/qlI4q8tZ9m
    
    Pokerocks297: https://t.co/Jwfq4nPzGS Screw @NintendoAmerica @NintendoUK @NintendoEurope @Nintendo this is ridiculous
    
    FoxyknoxyLewis: https://t.co/uwfhl6IXBA
    Come watch me play old school nintendo!!
    
    VileMK2: Meaning do you think Nintendo should get as much hate right now as the hate people have for companies that are bein… https://t.co/fN0E6myiDf
    
    AnaHern42406414: @ColourPopCo My wish list is a big box of LIPPIE  STIX VAULT,  NINTENDO SWITCH and a TARGET GIFT CARD  🎅🎄🎅🎄🎅🎁🤞🤞… https://t.co/kHx58vC040
    
    AnthoShaffer487: Speaking of Smash Bros, Nintendo is still under hot water. Not only are they making Mario the strongest being to ev… https://t.co/iuK1xS1QD0
    
    AnthoShaffer487: Yeah. Not a good light Nintendo. I just worry for you if you ever come across Semjax because honestly if he found t… https://t.co/lSNhKTE3ND
    
    RustysBandana: This anger towards Nintendo just reminds me of that guy who chose to disown them just cause the Xbox 360 port/remas… https://t.co/T4TgAewNYl
    
    GameWar64: God Nintendo as a hole is on so much fire right now.
    
    colamusa209: Check out NINTENDO PUNCH-OUT!! MACHO MAN TOPPS SCRATCH OFF #1 YEAR 1989 EMC GRADED 10 MINT  https://t.co/fbWnAU0ILY via @eBay
    
    RykerDotJPG: @Asuvir_Bleats Nintendo don't do something shitty every week challenge
    
    MxAmericanPi: Waiting for Nintendo to come chop off my arm for having an unlicensed Zelda tattoo...
    
    dealingindeals: Kohl's #ad : Nintendo Switch Lite &amp; Super Smash Bros Bundle $270 + $60 Kohl’s Cash
    
    SHOP HERE!
    
    Pay: $269.99
    Get Ba… https://t.co/mCXfV0SPFQ
    
    TheMightyBitBit: @Saberspark Just because you CAN, doesn't mean you should... Nintendo should remember that
    
    BloodWar7: @spl_ayo Ah so that's why I've been seeing nintendo frustration this morning
    
    GameOHolic1: 5 mins Away!!! https://t.co/auuKKawlJ9 #homealone #christmas #videogames #nintendo https://t.co/DKpXNYEWMM
    
    

### Type Ctrl + C to terminate the previous snippet then try again with a different search term.


```python
counts['total_tweets']#how many tweets we got
```




    86




```python
counts['locations']#how may tweets had locations. 
```




    51




```python
#the percentage that had locations:
print(f'{counts["locations"] / counts["total_tweets"]:.1%}')
```

    59.3%
    

### Now, let’s use our get_geocodes utility function from tweetutilities.py to geocode the location of each tweet stored in the list tweets:

### As you’ll soon see, for each tweet with a valid location, the get_geocodes function adds to the tweet’s dictionary in the tweets list two new keys—'latitude' and 'longitude'. For the corresponding values, the function uses the tweet’s coordinates that OpenMapQuest returns.


```python
from tweetutilities import get_geocodes
```


```python
bad_locations = get_geocodes(tweets)#Sometimes the OpenMapQuest geocoding service times out, meaning that it cannot 
#handle your request immediately and you need to try again.
```

    Getting coordinates for tweet locations...
    Done geocoding
    

### Before we plot the tweet locations on a map, let’s use a pandas DataFrame to clean the data. When you create a DataFrame from the tweets list, it will contain the value NaN for the 'latitude' and 'longitude' of any tweet that did not have a valid location. We can remove any such rows by calling the DataFrame’s dropna method:


```python
import pandas as pd
df = pd.DataFrame(tweets)
df = df.dropna()#drops the bad locations
```

### Now, let’s create a folium Map on which we’ll plot the tweet locations:


```python
import folium
usmap = folium.Map(location=[39.8283, -98.5795],
tiles='Stamen Terrain',
zoom_start=5, detect_retina=True)
#The values above are the geographic center of the continental United States
```

### Next, let’s iterate through the DataFrame and add to the Map folium Popup objects containing each tweet’s text. In this case, we’ll use method itertuples to create tuples from each row of the DataFrame. Each tuple will contain a property for each DataFrame column:


```python
df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>screen_name</th>
      <th>text</th>
      <th>location</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NERUPLUSHIE</td>
      <td>mutliple??? nintendo are you ok https://t.co/r...</td>
      <td>(•̀ᴗ•́)و</td>
      <td>17.626349</td>
      <td>-63.249002</td>
    </tr>
    <tr>
      <th>1</th>
      <td>bigboipascal</td>
      <td>@OnThisDayGaming Sakurai and Sora inc. deverse...</td>
      <td>Germany</td>
      <td>51.083420</td>
      <td>10.423447</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Diam_ond10</td>
      <td>@Pincan18 @RouGhBartL1 @_EyeAm @PoorlyAgedStuf...</td>
      <td>Chicago, IL</td>
      <td>41.875555</td>
      <td>-87.624421</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Let's see the location's values and try to determine where in the world has the most tweets
df.location.value_counts()
```




    Canada                         2
    Los Angeles, CA                2
    Searcy, AR                     2
    Maryland, USA                  2
    Vancouver, British Columbia    1
    Skiatook, OK                   1
    Montréal, Québec               1
    (•̀ᴗ•́)و                       1
    New York, USA                  1
    your moms house                1
    idk                            1
    United States                  1
    Lost                           1
    Centerville, OH                1
    Gran Tesoro                    1
    Salt Lake City, UT             1
    Chicago, IL                    1
    Arizona USA                    1
    Germany                        1
    San Pedro, Los Angeles         1
    Melbourne, Victoria            1
    Margate, FL                    1
    NewMoon island                 1
    instagram                      1
    Michigan                       1
    Curitiba, Brasil               1
    Liverpool, England             1
    Name: location, dtype: int64




```python
#Export our DataFrame to a csv file for observation
import xlsxwriter
df.to_csv('Tweet_Locations.csv', sep = '-')
```


```python
for t in df.itertuples():
    text = ': '.join([t.screen_name, t.text])
    popup = folium.Popup(text, parse_html=True)
    marker = folium.Marker((t.latitude, t.longitude),popup=popup)
    marker.add_to(usmap)
```


```python
#outputs a map of the world, focused on the US. Zoom out to see the whole world. 
usmap.save('tweet_map.html')
```

<span style="font-family:Trebuchet MS; font-size:3em; color:#fdd700; text-shadow: 3px 3px #000">Tweet me if you need me! @NightHawk</span>
