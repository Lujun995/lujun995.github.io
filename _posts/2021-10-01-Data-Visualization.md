---
layout: post
title: Data Visualization Challenge
subtitle: by Lucas Zhang
tags: [Data Visualization Challenge]
---

Create 3 informative visualizations about [malaria](https://github.com/rfordatascience/tidytuesday/tree/master/data/2018/2018-11-13) using Python in a Jupyter notebook. Where appropriate, make the visualizations [interactive](https://jupyterbook.org/interactive/interactive.html).

**Solution**:

The Malaria Dataset comes from a [Data Challenge](https://www.synapse.org/#!Synapse:syn16788291/wiki/583310) held by the Wellcome Trust and Sage Bionetworks, which was hosted in 2018. This dataset includes 3 subdatasets containing information of malaria incidence, overall deaths rates of all ages and death rates in different age groups across different countries and time all over the world.

Here I used the Python packages `pandas` to preprare the datasets and `bokeh` to create interative figures decribing the malaria conditions in Asian countries in the recent years. [bokeh](https://bokeh.org/) is a widely used open-source library to create interactive and shareable figures. It comes with a powerful toolbos which allows moving, zooming, reseting and saving the figures. 

![bokeh_illustration](/assets/img/bokeh_illustration.gif){: .mx-auto.d-block :}

if bokeh is not installed, we need to first install it using the following code:

{% highlight python %}
! python3 -m pip install --quiet bokeh
{% endhighlight %}

After the installation, we need first to import required libraries and read in the datasets using `pandas`. I also recommend a [21-color palette](https://tradeblotter.wordpress.com/2013/02/28/the-paul-tol-21-color-salute/) to display a large dataset. In the following figures, we need a [relationship table](https://datahub.io/JohnSnowLabs/country-and-continent-codes-list) mapping coutries to their continents. 
 
{% highlight python %}
#read in the data and import required modules
import pandas as pd
from bokeh.plotting import figure, show, output_notebook

country_continent_code_url = ("https://pkgstore.datahub.io/JohnSnowLabs/country-and-continent-codes-list/"
                              "country-and-continent-codes-list-csv_csv/data/b7876b7f496677669644f3d1069d3121/"
                              "country-and-continent-codes-list-csv_csv.csv")
country_continent_code = pd.read_csv(country_continent_code_url)
country_continent_code = country_continent_code[["Continent_Name","Three_Letter_Country_Code"]]
country_continent_code.rename(columns={"Three_Letter_Country_Code": "Code"}, inplace=True)

tol21rainbow = ("#771155", "#AA4488", "#CC99BB", "#114477", "#4477AA", "#77AADD", 
                "#117777", "#44AAAA", "#77CCCC", "#117744", "#44AA77", "#88CCAA", 
                "#777711", "#AAAA44", "#DDDD77", "#774411", "#AA7744", "#DDAA77", 
                "#771122", "#AA4455", "#DD7788")

data_death = pd.read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2018/2018-11-13/malaria_deaths.csv")
data_age = pd.read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2018/2018-11-13/malaria_deaths_age.csv")
data_incidence = pd.read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2018/2018-11-13/malaria_inc.csv")
{% endhighlight %}

We then prepare our datasets for the data visualization. To achieve our aim, we first need to explore the dataset using *pandas.DataFrame.head()* function; thereafter, we need to merge information from malaria datasets with country-continent mapping table.

{% highlight python %}
#to show the first several rows
data_death.head()
country_continent_code.head()
#drop those without a country code (or assigned to "Global"?)
data_death = data_death[~(data_death["Code"].isna())]
#add continent information
data_death = data_death.merge(right = country_continent_code, how ='left',
                              left_on = "Code", right_on = "Code")
data_death.head()
#the last column name is tooooooooooo long, I change the name
data_death.rename(columns={"Deaths - Malaria - Sex: Both - Age: Age-standardized (Rate) (per 100,000 people)": "Death_rate"},
                 inplace=True)
data_death.head()
{% endhighlight %}

Finally, we will get a table like the follwing:

![table](/assets/img/1633223052(1).png){: .mx-auto.d-block :}

