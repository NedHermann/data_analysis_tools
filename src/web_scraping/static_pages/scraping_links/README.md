## WEB SCRAPING - STATIC LINKS

We will be going over how to use [rvest](https://github.com/tidyverse/rvest) to scrape links from [static websites](https://en.wikipedia.org/wiki/Static_web_page). Refer to the [static_pages/scraping_tables](./scraping_tables) readme for details of each functions.

### Loading Libraries
- tidyverse: data science
- rvest: web scraping

```
library(tidyverse)
library(rvest)
```

### Website
We will be scraping a list of articles from [MSFT's investor releations](https://www.microsoft.com/en-us/investor/events/events-recent.aspx). We want the date, title, subtitle, and all links.
We will save this link as the variable: `url`

```
url <- "https://www.microsoft.com/en-us/investor/events/events-recent.aspx"
```

The articles we will be scraping looks like this:  
![msft_tbl](images/msft.png)

### Reading HTML
- Used the `read_html()` function to read the page's html.
- The data we want is wrapped in a parent div with the following css selector: `div.PastEvents div.m-content-placement-item.f-size-small`.
 - Used the `html_nodes()` function to pull it and all of it's children elements
- Saved this element as `div_tags`.

```
div_tags <- read_html(url) %>%
  html_nodes("div.PastEvents div.m-content-placement-item.f-size-small") 
```

### Scraping the articles
The following code block is a bit to unpack, so we'll do it step-by-step. Essentially, I created an `lapply()` loop that goes through each article div and pulls for it's date, title, subtitle, link_names, and link_urls. What's unique about this scrape is that not every article has a link and there are differing numbers of supplemental links. We'll need to build something robust to handle the indifference and make sure all the data is where it should be.

```
msft_tbl <- lapply(1:length(div_tags), function(x){
  el <- div_tags[[x]] 
  
   title <- el %>%
    html_nodes("ul li:nth-child(3) h3") %>%
    html_text()
  
  subtitle <- el %>%
    html_nodes("ul li:nth-child(4)") %>%
    html_text()
  
  date <- el %>%
    html_nodes("ul li:nth-child(2)") %>%
    html_text() %>%
    as.Date(., format = "%B %d, %Y")
  
  if(length(html_nodes(el, "ul li:nth-child(1) a")) > 0){
    article <- html_nodes(el, "ul li:nth-child(1) a") %>%
      html_attr("href")
    
  } else {
    article <- NA
  }
  
  supl_links <- el %>%
    html_nodes("ul li:nth-child(5) a") %>%
    html_attr("href")
  
  supl_link_names <- el %>%
    html_nodes("ul li:nth-child(5) a") %>%
    html_text()
  
  tbl <- data.frame(
    date = date,
    title = title,
    subtitle = subtitle,
    link_name = janitor::make_clean_names(c("article", supl_link_names)),
    link = c(article, supl_links))
  
  return(tbl)
})
```

### Scraping the articles cont.
The way `lapply()` works is that it applies the function from the 2nd argument to our 1st argument's values. We used the `length()` argument to see how many articles were within our div_tag (76) and then created a numeric vector from 1 to 76 `1:length(div_tags)`. Each iteration of this vector is passed through our function as the variable `x`.  

The first part of the lapply function is within the code block below:
- Subset the articles with `[[x]]` and saved it as the variable `el`.
- Used the `html_nodes()` function to select the 3rd li child element and extracted it's child h3 text. This is where our titles are located.
- Saved the title as `title`.
- Do the same for the 4th li child element and extract it's html text. This is where our subtitles are located.
- Saved the subtitles as `subtitle`.

```
el <- div_tags[[x]] 

title <- el %>%
  html_nodes("ul li:nth-child(3) h3") %>%
  html_text()

subtitle <- el %>%
  html_nodes("ul li:nth-child(4)") %>%
  html_text()
```

### Scraping the articles cont.
The second part of the lapply function is within the code block below:
- Used the `html_nodes()` function to select the 2nd li child element and extracted it's text. This is where our dates are located.
- Used the `as.Date()` function and inputted the date's appropiate format with the `format` argument. This transforms the string to a date class.
- Saved the date as `date`.

```
date <- el %>%
  html_nodes("ul li:nth-child(2)") %>%
  html_text() %>%
  as.Date(., format = "%B %d, %Y")
```

### Scraping the articles cont.
As mentioned a [few sections ago](#scraping-the-articles), not every article has an article link.
- Used an `if else` statement to see if an anchor tag exists in the current article (where the image is displayed).
 - If so, then pull it's `href` attribute and save as `article`.
 - Else, save NA as `article`.

```
if(length(html_nodes(el, "ul li:nth-child(1) a")) > 0){
  article <- html_nodes(el, "ul li:nth-child(1) a") %>%
    html_attr("href") 

} else {
  article <- NA
}
```

### Scraping the articles cont.
- Pulled the 5th li child anchor tag's href (link) and text.
- Saved the link urls as `supl_links` and the link names as `supl_link_names`.

```
supl_links <- el %>%
  html_nodes("ul li:nth-child(5) a") %>%
  html_attr("href")

supl_link_names <- el %>%
  html_nodes("ul li:nth-child(5) a") %>%
  html_text()
```

### Scraping the articles cont.
Finally, created a dataframe with the columns `date`, `title`, `subtitle`, `link_name`, and `link`. Some articles will be differing in rows, hence a differing amount of supplemental links. Any links that are missing will be shown as an `NA`.  
We also used the `make_clean_names()` function from the `janitor` library to change the format of the link names to snake case. 

```
tbl <- data.frame(
  date = date,
  title = title,
  subtitle = subtitle,
  link_name = janitor::make_clean_names(c("article", supl_link_names)),
  link = c(article, supl_links))
```

### End of the lapply function
Once the lapply function goes through all the articles, it will save each article as a dataframe in a list called `msft_tbl`.  

Sample of one of the articles:
|date      |title                                       |subtitle                                                                  |link_name|link                                                         |
|----------|--------------------------------------------|--------------------------------------------------------------------------|---------|-------------------------------------------------------------|
|2023-02-07|New AI-powered Bing and Edge Conference Call|Amy Hood, EVP & CFO and Phil Ockenden, CVP & CFO Windows, Devices & Search|article  |/en-us/Investor/events/FY-2023/AI-Powered-Bing-Edge-Conf.aspx|
|2023-02-07|New AI-powered Bing and Edge Conference Call|Amy Hood, EVP & CFO and Phil Ockenden, CVP & CFO Windows, Devices & Search|webcast  |/en-us/Investor/events/FY-2023/AI-Powered-Bing-Edge-Conf.aspx|
|2023-02-07|New AI-powered Bing and Edge Conference Call|Amy Hood, EVP & CFO and Phil Ockenden, CVP & CFO Windows, Devices & Search|blog     |https://aka.ms/AAjd7x2                                       |

### Cleaning the table
As we can see from the table above, some of the links from the link column only has the path of the url, not the full link. This will be common in any anchor tag that redirects you to a different page in it's application. To create the full link, we just need to concatenate the domain url (`https://microsoft.com`) where the link starts with a `/`.
- Used the `bind_rows()` function to bind all articles from the msft_tbl list to a single dataframe.
- Used the `ifelse()` function to see if a link url starts with `/`, we will concatenate the domain url to it's beginning. Otherwise, leave it as is.
- Arranged by date in descending order.

```
msft_tbl_long <- bind_rows(msft_tbl) %>%
  mutate(link = ifelse(str_starts(link, "/"), paste("https://microsoft.com", link, sep = ""), link)) %>%
  arrange(desc(date))
```

### Sample of msft_tbl_long output
- date: date that the article was published
- title: title of the article
- subtitle: subtitle of the article
- link_name: link names
- link: link urls

|date      |title                                       |subtitle                                                                  |link_name|link                                                         |
|----------|--------------------------------------------|--------------------------------------------------------------------------|---------|-------------------------------------------------------------|
|2023-02-07|New AI-powered Bing and Edge Conference Call|Amy Hood, EVP & CFO and Phil Ockenden, CVP & CFO Windows, Devices & Search|article  |https://microsoft.com/en-us/Investor/events/FY-2023/AI-Powered-Bing-Edge-Conf.aspx|
|2023-02-07|New AI-powered Bing and Edge Conference Call|Amy Hood, EVP & CFO and Phil Ockenden, CVP & CFO Windows, Devices & Search|webcast  |https://microsoft.com/en-us/Investor/events/FY-2023/AI-Powered-Bing-Edge-Conf.aspx|
|2023-02-07|New AI-powered Bing and Edge Conference Call|Amy Hood, EVP & CFO and Phil Ockenden, CVP & CFO Windows, Devices & Search|blog     |https://aka.ms/AAjd7x2                                       |

### Transforming msft_tbl_long to a wide format
You can also transform this long table to a wider one by the link_name column. This avoids repeating data from columns like date, title, subtitle and can be useful in some analysis.

```
msft_tbl_wide <- msft_tbl_long %>%
  pivot_wider(names_from = "link_name", values_from = "link") %>%
  arrange(desc(date))
```

### Sample of msft_tbl_wide output

|date      |title                                       |subtitle                                                                  |article|webcast                                                      |blog                  |microsite                               |power_point                                                                                                                                                     |transcript                                                                                                                                                          |event_website                                                                 |
|----------|--------------------------------------------|--------------------------------------------------------------------------|-------|-------------------------------------------------------------|----------------------|----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------|
|2023-02-07|New AI-powered Bing and Edge Conference Call|Amy Hood, EVP & CFO and Phil Ockenden, CVP & CFO Windows, Devices & Search|https://microsoft.com/en-us/Investor/events/FY-2023/AI-Powered-Bing-Edge-Conf.aspx|https://microsoft.com/en-us/Investor/events/FY-2023/AI-Powered-Bing-Edge-Conf.aspx|https://aka.ms/AAjd7x2|https://news.microsoft.com/feb-2023-news|http://view.officeapps.live.com/op/view.aspx?src=https://c.s-microsoft.com/en-us/CMSFiles/InvestorPresentation.pptx?version=6ff579c4-103e-76c0-8a18-05085e0632c7|http://view.officeapps.live.com/op/view.aspx?src=https://c.s-microsoft.com/en-us/CMSFiles/Transcript223.docx?version=e69dc7f8-a0b8-7d09-ff82-f821891ad767           |NA                                                                            |
|2023-01-24|Microsoft Fiscal Year 2023 Second Quarter Earnings Conference Call|Satya Nadella, Chairman and CEO and Amy Hood, EVP & CFO                   |https://microsoft.com/en-us/Investor/events/FY-2023/earnings-fy-2023-q2.aspx|https://microsoft.com/en-us/Investor/events/FY-2023/earnings-fy-2023-q2.aspx|NA                    |NA                                      |http://view.officeapps.live.com/op/view.aspx?src=https://c.s-microsoft.com/en-us/CMSFiles/SlidesFY23Q2.pptx?version=8b9dc2d4-0847-8499-8c6e-02a4c377194e        |http://view.officeapps.live.com/op/view.aspx?src=https://c.s-microsoft.com/en-us/CMSFiles/TranscriptFY23Q2.docx?version=c501559f-1cc8-0347-51e3-215c864aca4f        |https://microsoft.com/en-us/Investor/earnings/FY-2023-Q2/press-release-webcast|
|2022-12-19|IR Fireside Chat: Advertising               |Virtual Meeting                                                           |https://microsoft.com/en-us/Investor/events/FY-2023/IR-Fireside-Chat-Advertising|https://microsoft.com/en-us/Investor/events/FY-2023/IR-Fireside-Chat-Advertising|NA                    |NA                                      |NA                                                                                                                                                              |https://view.officeapps.live.com/op/view.aspx?src=https://c.s-microsoft.com/en-us/CMSFiles/IRFireside_Advertising_.docx?version=dad7b600-6b7e-c1cb-ed4f-43a507c10ee7|NA                                                                            |
