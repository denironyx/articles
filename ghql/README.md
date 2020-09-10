How did I find out about ghql?
------------------------------

After spending days researching about the R packages that can interact
with a GraphQL API, I found three packages. I picked `ghql` over the
others because, it was an rOpenSci package.

So what is GraphQL?
-------------------

GraphQL is a query language for application programming interfaces
(APIs) that prioritizes giving clients exactly the data they request.
It’s designed to make APIs flexible, fast and friendly. Basically, it is
used to load data from a server to a client and it does this in a much
more efficient manner than traditional methods and services.

And then, there is R
--------------------

R is a free software environment for statistical computing and graphics.
Well, that’s how it’s officially defined. But, trust me its gradually
evolving to do more than statistical computing and making of beautiful
graphics.

Moving on
---------

In order for R to interact with any GraphQL API, it requires a GraphQL
client. That’s where `ghql`, a GraphQL client for R, developed by Scott
Chamberlain comes into play. Still confused? I hope the flow chart below
helps you make more sense of it. ![GraphQL client and R connection
flow.](GHQL.png)

Working with `Countries List`, a GraphQL public API
---------------------------------------------------

The [Countries GraphQL API](https://github.com/trevorblades/countries)
is a public GraphQL API for information about countries, continents, and
languages. This public API uses [Countries
List](https://annexare.github.io/Countries/) and
[provinces](https://github.com/substack/provinces) as data sources, so
the schema follows the shape of that data, with a few exceptions: - The
codes used to key the objects in the original data are available as a
code property on each item returned from the API. - The
country.continent and country.languages are objects and arrays of
objects, respectively. - Each Country has an array of states populated
by their states/provinces, if any.

Loading the libraries

    library(ghql)
    library(jsonlite)
    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

Link to the GraphQL schema api

    link <- 'https://countries.trevorblades.com/'

Create a new graphqlClient object

    conn <- GraphqlClient$new(url = link)

Define a Graphql Query

    query <- '
    query($code: ID!){
      country(code: $code){
        name
        native
        capital
        currency
        phone
        languages{
          code
          name
        }
      }
    }'

The `ghql` query class and define query in a character string

    new <- Query$new()$query('link', query)

Inspecting the schema

    new$link

    ##  
    ##  
    ## query($code: ID!){
    ##   country(code: $code){
    ##     name
    ##     native
    ##     capital
    ##     currency
    ##     phone
    ##     languages{
    ##       code
    ##       name
    ##     }
    ##   }
    ## }

Define a variable as a named list

    variable <- list(
      code = "DE"
    )

Making a request, passing in the query and then the variables. Then you
convert the raw object to a structured json object

    result <- conn$exec(new$link, variables = variable) %>% 
      fromJSON(flatten = F)
    result

    ## $data
    ## $data$country
    ## $data$country$name
    ## [1] "Germany"
    ## 
    ## $data$country$native
    ## [1] "Deutschland"
    ## 
    ## $data$country$capital
    ## [1] "Berlin"
    ## 
    ## $data$country$currency
    ## [1] "EUR"
    ## 
    ## $data$country$phone
    ## [1] "49"
    ## 
    ## $data$country$languages
    ##   code   name
    ## 1   de German

Convert the json data into a tibble object

    country_data <- result$data$country %>% 
      as_tibble()
    country_data

    ## # A tibble: 1 x 6
    ##   name    native      capital currency phone languages$code $name 
    ##   <chr>   <chr>       <chr>   <chr>    <chr> <chr>          <chr> 
    ## 1 Germany Deutschland Berlin  EUR      49    de             German

More examples
-------------

Working with a GraphQL API without a defined variable named list

    link <- 'https://countries.trevorblades.com/'

    conn <- GraphqlClient$new(url = link)

    qry <- Query$new()


    qry$query('x', '{
    continent(code: "AF") {
        countries{
        code
        name
        native
        capital
        currency
        phone
        languages {
          name
        }
        }
    }
      }
              ')

    res <- conn$exec(qry$queries$x)

    res <- jsonlite::fromJSON(res, 
                              flatten = TRUE)

    res_data <- res$data$continent$countries %>% 
      as.data.frame()

    head(res_data)

    ##   code                             name                           native
    ## 1   AO                           Angola                           Angola
    ## 2   BF                     Burkina Faso                     Burkina Faso
    ## 3   BI                          Burundi                          Burundi
    ## 4   BJ                            Benin                            Bénin
    ## 5   BW                         Botswana                         Botswana
    ## 6   CD Democratic Republic of the Congo République démocratique du Congo
    ##       capital currency phone                                     languages
    ## 1      Luanda      AOA   244                                    Portuguese
    ## 2 Ouagadougou      XOF   226                                  French, Peul
    ## 3   Bujumbura      BIF   257                               French, Kirundi
    ## 4  Porto-Novo      XOF   229                                        French
    ## 5    Gaborone      BWP   267                               English, Tswana
    ## 6    Kinshasa      CDF   243 French, Lingala, Kongo, Swahili, Luba-Katanga
