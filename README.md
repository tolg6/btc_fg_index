# Get Bitcoin Price - Fear & Greed Index

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

In this notebook, bitcoin price of the last 90 days and the Bitcoin Fear&Greed Index will be taken 
with the API and the conversion into a data frame will be done.

<p>price link : <a href="https://min-api.cryptocompare.com/documentation" class="uri">https://min-api.cryptocompare.com/documentation</a>.</p>
<p>Fear&Greed Index = <a href="https://alternative.me/crypto/fear-and-greed-index/" class="uri">https://alternative.me/crypto/fear-and-greed-index/</a>.</p>

![btc](https://github.com/tolg6/btc_fg_index/blob/main/unnamed-chunk-8-1.png?raw=true)

## Libraries
```{r,warning = FALSE,message = FALSE}
library(httr)
library(jsonlite)
library(RCurl)
library(dplyr)
library(lubridate)
library(ggplot2)
```

First, lets determine the limit (number of days) parameter. I get 90 in this notebook and we set it to use it jointly in both links.

```{r,warning = FALSE,message = FALSE}
limit = 90
link_fg_index = paste0("https://api.alternative.me/fng/?limit=",limit)
link_fg_index = URLencode(link_fg_index)
linkbtc = paste0("https://min-api.cryptocompare.com/data/v2/histoday?fsym=BTC&tsym=USDT&limit=",limit)
linkbtc = URLencode(linkbtc)
```

Lets take the fear & greed data.

```{r,warning = FALSE,message = FALSE}
fg_data = GET(link_fg_index)
fg_data
```

We will convert it to JSON first and then to Dataframe format, respectively.

```{r,warning = FALSE,message = FALSE}
fg_data = prettify(rawToChar(fg_data$content))
fg_data = fromJSON(fg_data)
fg_data = fg_data$data%>%as.data.frame()
colnames(fg_data) = c("value","value_class","time","update")
fg_data
```

Now lets take the Bitcoin data.
Since the number of rows is limit+1 in the data from cryptocompare.com, 
lets remove the first row and equate the days of the price data with the Fear&Greed index.

```{r,warning = FALSE,message = FALSE}
btc = fromJSON(getURL(linkbtc))
btc = as.data.frame(btc$Data$Data)
btc = btc[-1,]
head(btc,10)
```


Lets combine the 2 datasets according to the time variable.

```{r,warning = FALSE,message = FALSE}
btc$time = as.character(btc$time)
btc_fg_data = btc%>%left_join(fg_data,"time")
head(btc_fg_data,10)
```

Lets change the time variable to the appropriate format.

```{r,warning = FALSE,message = FALSE}
class(btc_fg_data$time) = c('POSIXt','POSIXct')
btc_fg_data$time = btc_fg_data$time%>%lubridate::as_date()
btc_fg_data<-btc_fg_data%>%select(time,close,value,value_class)
btc_fg_data$value = as.numeric(btc_fg_data$value)
head(btc_fg_data,10)
```

### Basic Plotting

```{r,warning = FALSE,message = FALSE}
library(ggfortify)
btc_fg_ts = ts(btc_fg_data[,2:3],frequency = 365,start = c(2021,9,28))
autoplot(ts(btc_fg_data[,2:3],frequency = 365,start = c(2021,9,28)),facets = T)+ylab("")+theme_minimal()
```

