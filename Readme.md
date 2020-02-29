---
title: "Social Media Data Collection"
author: "Karl Ho"
date: "`r Sys.Date()`"
output:
pdf_document: default
html_document:
df_print: paged
---


# I. Using API Method to collect Twitter data

Acquire API key and token from [Twitter developer website](http://dev.twitter.com) 
1. Login using your Twitter account (create one if none exists)
2. Click on Apps, create an app and apply for a developer account.  Give detail on your purpose (e.g. personal research)

Sample description:

1. Using API to conduct public opinion research.
2. Analyze Tweet contents, trends and transactional data in social networks.
3. Focus on Tweeting, favorites/likes, following and retweeting will be involved
4. Aggregate data will be presented to the public and reviewing agency targeting publications in academic journals and presentations in academic conferences.

Once approved, Twitter will provide API detail in four keys/secret/tokens. Open an R session and store the API data:

```
## Create token for direct authentication to access Twitter data

token <- rtweet::create_token(
  app = "Your App name",
  consumer_key <- "YOURCONSUMERKEY",
  consumer_secret <- "YOURCONSUMERSECRET",
  access_token <- "YOURACCESSTOKEN",
  access_secret <- "YOURACCESSSECRET")

## Check token

rtweet::get_token()
```

With API methods, there are plenty of R packages for collecting Twitter data.  Examples include twitteR, vosonSML and rtweet.  The following illustration uses rtweet, which gives most detail in twitter variables (almost 90). 

```
## Install packages need for Twitter data download

install.packages(c("rtweet","igraph","tidyverse","ggraph","data.table"), repos = "https://cran.r-project.org")

## Load packages

library(rtweet)
library(igraph)
library(tidyverse)
library(ggraph)
library(data.table)
```


```
## Search for 1,000 tweets in English
# Not run: 
rdt <- rtweet::search_tweets(q = "realDonaldTrump", n = 1000, lang = "en")
# End(Not run)

## preview users data
users_data(rdt)

## Boolean search for large quantity of tweets (which could take a while)
rdt <- rtweet::search_tweets(
  "Trump OR president OR potus", n = 10000,
  retryonratelimit = TRUE
)

## plot time series of tweets frequency
ts_plot(rdt, by = "mins")

```
[image](twittertimeseries.png)

To explore the network structure of the Twitter data, [igraph](http://kateto.net/networks-r-igraph) and [ggraph](https://www.data-imaginist.com/2017/ggraph-introduction-layouts/) packages are recommended for network plots 

```

## Create igraph object from Twitter data using user id and mentioned id.
## ggraph draws the network graph in different layouts (12). 

filter(rdt, retweet_count > 0 ) %>% 
  select(screen_name, mentions_screen_name) %>%
  unnest(mentions_screen_name) %>% 
  filter(!is.na(mentions_screen_name)) %>% 
  graph_from_data_frame() -> rdt_g
V(rdt_g)$node_label <- unname(ifelse(degree(rdt_g)[V(rdt_g)] > 20, names(V(rdt_g)), "")) 
V(rdt_g)$node_size <- unname(ifelse(degree(rdt_g)[V(rdt_g)] > 20, degree(rdt_g), 0)) 
ggraph(rdt_g, layout = 'kk') + 
  geom_edge_arc(edge_width=0.1, aes(alpha=..index..)) +
  geom_node_label(aes(label=node_label, size=node_size),
                  label.size=0, fill="#ffffff66", segment.colour="light blue",
                  color="red", repel=TRUE, family="Apple Garamond") +
  coord_fixed() +
  scale_size_area(trans="sqrt") +
  labs(title="Title", subtitle="Edges=volume of retweets. Screenname size=influence") +
  theme_graph(base_family="Apple Garamond") +
  theme(legend.position="none") 
```

# II. Using non-API Method to collect Twitter data

Twitter API is not without limits. These limits vary over time and it currently allows one week's data.  Some packages can reach data within a shorter period due to data size.  Other methods have been developed to collect historical Twitter data.  [Jefferson Henrique](https://github.com/Jefferson-Henrique/GetOldTweets-python) and [Dimtry Mottl](https://github.com/Mottl/GetOldTweets3) python packages are illustrated here.  This non-API method scrapes Twitter data based on Twitter search results by parsing the result page with a scroll loader, then calling to a JSON provider. While theoretically it can search through oldest tweets and collect data accordingly, the number of variables are limited to the layout of search results.

Prerequesites: 

1. Python3
2. Bash/terminal command line tool
3. Python pip package installer

Note: Windows system has not been tested in this workshop.

Illustration using [GetOldTweets3](https://pypi.org/project/GetOldTweets3/)
Install Python 3.x (e.g. Anaconda3) and run the following preparation steps (creating virtual environment, install GetOldTweets3 package using pip):
```
python3 -m venv env
source ./env/bin/activate 
python3 -m pip install GetOldTweets3
```

Alternatively, 

```
pip3 install -e git+https://github.com/Mottl/GetOldTweets3#egg=GetOldTweets3
```

There are two methods of collecting Twitter data.  The GetOldTweets3 command method is recommended since the data collection process can be time-consuming.

Examples:

```
## Keyword search
GetOldTweets3 --querysearch "Trump Kim" --since 2018-01-01 --until 2019-01-16 --output trumpkim.csv

## username search with time period and size limit
GetOldTweets3 --username "realDonaldTrump" --since 2016-11-01 --until 2019-03-08 --maxtweets 20000 --output rdt_2016_now.csv
```



