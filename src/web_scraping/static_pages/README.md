## Web Scraping Static Pages

Scraping from static pages is easier than dynamic pages. This is because the html you receive once connected to the server has the data you want to scrape already loaded in the dom. We can just read the html from the page source and pull what we need.

Each website you want to scrape will be different. Some of the data you want are in different elements with different characteristics from the last website. You will need to already know about targetting the correct information with `css selectors`. 

### DIRECTORY:
- Scraping HTML Tables
  - Code: [static_pages/scraping_tables](static_pages/scraping_tables)
  - Description: Scrapes html tables with the `rvest` library. The table we scraped in this example wasn't paginated, but if it was, you would scrape the anchor tag that it would redirect you to (ie. `domain.com/table?pg=2`), read it's html, and then just scrape it's contents. You would create a loop to go through all the available pages. Sometimes, the pagination doesn't redirect you to another page, the data is already there but hidden. In this case, you can just scrape the single page.

- Scraping Links
  - Code: [static_pages/scraping_links](static_pages/scraping_links)
  - Description: Scrapes links with the `rvest` library. The function we used to scrape html tables is straight-forward but it only works for `table` elements. Sometimes the stuff you want to scrape from static websites are a bit more involved. In this case, I went to msft's investors releation website to scrape it's news releases. Again, didn't need to deal with pagination and we could've pulled more information on the articles than we did - but the techniques used can be leveraged to achieve both feats if needed.
 
