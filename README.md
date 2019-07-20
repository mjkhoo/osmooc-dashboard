# Open Science MOOC Dashboard

====================================================

IMPORTANT

This is NOT the current version of this tool. It is a version I cloned to experiment with formatting some of the interface elements in order to make the behavior of the dashboard more predictable when expanding and contracting the browser window horizontally.

The current working version of the dashboard is available at the link in the following section.

====================================================

Data and source code for [this dashboard](http://www.dataplanes.org/osmooc-dashboard/) on the Open Science MOOC's GitHub repository statistics and user activities. 

## How to collect GitHub data

### Setup

```{r}
# Install and load packages using pacman
if (!require("pacman")) install.packages("pacman")
library(pacman)

p_load(httr, jsonlite, tidyverse)
```

### Authentication

See e.g. [this article](https://towardsdatascience.com/accessing-data-from-github-api-using-r-3633fb62cb08) for instructions on how to set up your own GitHub app.

```{r}
# Set OAuth
oauth_endpoints("github")
gh_app <- oauth_app(appname = "[INSERT HERE]",
                   key = "[INSERT HERE]",
                   secret = "[INSERT HERE]")

# Get credentials and config
github_token <- oauth2.0_token(oauth_endpoints("github"), gh_app)
gtoken <- httr::config(token = github_token)
```

### Custom functions to retrieve data from GitHub

```{r functions}
# Function to submit API request, parse JSON content, and convert to data frame 
get_data <- function(url) {
  
  res <- httr::GET(url, query = list(state = "all", per_page = 100, page = 1), gtoken)
  stop_for_status(res)
  res_df <- jsonlite::fromJSON(content(res, type = 'text', encoding = "UTF-8"))
  
  return(res_df)
}

# Function to submit multiple API requests, parse JSON content, and convert to data frame 
get_data_multiple <- function(urls) {
  
  res <- lapply(urls, get_data)
  res_df <- map_df(res, ~as.data.frame(.x), .id = "df_id")
  
  return(res_df)
}
```

### Collect GitHub data on the Open Science MOOC

```{r}
# Retrieve Open Science MOOC repos (modules 1-10 only)
repos_df <- get_data("https://api.github.com/orgs/OpenScienceMOOC/repos")
repos_df_mod <- repos_df %>% 
  filter(stringr::str_detect(name, "Module-"))

# Retrieve contributors for each repo
contributors_df <- get_data_multiple(repos_df_mod$contributors_url)

# Retrieve stargazers for each repo
stargazers_df <- get_data_multiple(repos_df_mod$stargazers_url)

# Retrieve subscribers for each repo
subscribers_df <- get_data_multiple(repos_df_mod$subscribers_url)

# Export data
save(repos_df_mod, contributors_df, stargazers_df, subscribers_df, file = "osmooc-github.RData")
```


