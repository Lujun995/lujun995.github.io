---
layout: post
title: Dashboard using Python
subtitle: by Lucas Zhang
tags: [Data Visualization Challenge]
---

Is there life after graduate school? Download data of [Science and Engineering PhDs awarded in the US](https://ncses.nsf.gov/pubs/nsf19301/data). Do some analysis in `pandas`. Make a [dashboard visualization](https://pyviz.org/dashboarding/) of a few interesting aspects of the data.

**Solution**:

Here I visualize the tables [Doctorate recipients from U.S. colleges and 
universities: 1958–2017](https://ncses.nsf.gov/pubs/nsf19301/assets/data/tables/sed17-sr-tab001.xlsx) and [Doctorate-granting institutions, by state or location and major science and engineering fields of study: 2017](https://ncses.nsf.gov/pubs/nsf19301/assets/data/tables/sed17-sr-tab007.xlsx) to illustrate the overall tendencies of Doctoral recipients over years and across different institutes in the U.S.

To achieve the desired outcomes, we need first to set up a local Python environment for our app. Here I use [Anaconda3](https://www.anaconda.com/products/individual), which is a collection of Python packages for scientific programming, and it facilitates our package management. After the installation, we use the Spyder IDE in the Anaconda Navigator. Spyder IDE is an IPython development environment that provides many handy interfaces of, for example, editing, variable exploration and project management.

![Spyder_IDE](/assets/img/Spyder_IDE.png){: .mx-auto.d-block :}

![Spyder_IDE2](/assets/img/Spyder_IDE2.png){: .mx-auto.d-block :}

To set up our dashboard, we need several libraries including `pandas`, `plotly.express`, `dash` and `dash_bootstrap_components`. `pandas` is a library focusing on dataset management, `plotly.express` can plot fancy figures, `dash` is used to render a website for our contents and make the figures interactive, and `dash_bootstrap_components` can provide integrated themes for `dash`.

{% highlight python %}
#Initialization
work_directory = "C:\\Users\\zlj\\Desktop\\BIOSTAT 823\\HW4"

import os
import pandas as pd
import plotly.express as px  
from dash import Dash, dcc, html, Input, Output
import dash_bootstrap_components as dbc
{% endhighlight %}

Next, we use `pandas` to read in the data and prepare the data for more advanced visualization.

{% highlight python %}
#Data preparation
os.chdir(work_directory)
f_time = "Number of Doctorate recipients 1958–2017.xlsx"
f_instit_majo = "Doctorate-granting institutions, by location and major fields.xlsx"
df_time = pd.read_excel(f_time, header=[0], engine='openpyxl')
df_demograph = pd.read_excel(f_instit_majo, header=[0, 1, 2], engine='openpyxl')
df_demograph = df_demograph.set_index(df_demograph.columns[0])
df_demograph = df_demograph.stack(level=0).stack(level=0).stack(level=0).reset_index()
colnames = ["Location", "Sci/Eng", "Field", "Major", "Student Number"]
colnames_o = list(df_demograph)
df_demograph.rename(columns = {colnames_o[i]: colnames[i] 
                                 for i in range(0,len(colnames_o))},
                      inplace = True)
for i in range(2, df_demograph.shape[1]-1):
    index = df_demograph.iloc[:, i].str.contains("Unnamed:").values
    df_demograph.iloc[index, i] = df_demograph.iloc[index, i - 1]
states = ["Alaska", "Alabama", "Arkansas", "American Samoa", "Arizona", 
          "California", "Colorado", "Connecticut", "District ", "of Columbia", 
          "Delaware", "Florida", "Georgia", "Guam", "Hawaii", "Iowa", "Idaho", 
          "Illinois", "Indiana", "Kansas", "Kentucky", "Louisiana", "Massachusetts", 
          "Maryland", "Maine", "Michigan", "Minnesota", "Missouri", "Mississippi", 
          "Montana", "North Carolina", "North Dakota", "Nebraska", "New Hampshire", 
          "New Jersey", "New Mexico", "Nevada", "New York", "Ohio", "Oklahoma", "Oregon", 
          "Pennsylvania", "Puerto Rico", "Rhode Island", "South Carolina", "South Dakota", 
          "Tennessee", "Texas", "Utah", "Virginia", "Virgin Islands", "Vermont", "Washington", 
          "Wisconsin", "West Virginia", "Wyoming"]
df_states = df_demograph[df_demograph.Location.isin(states)]
df_states = df_states[df_states.Field != "All fields"]
df_states = df_states[df_states.Major != "Total"]
df_instit = df_demograph[ ~ df_demograph.Location.isin(states)]
df_instit = df_instit[df_instit.Field != "All fields"]
df_instit = df_instit[df_instit.Major != "Total"]
df_instit = df_instit[df_instit.Location != "All institutions"]
{% endhighlight %}

We will also need some texts for explanations in our dashboard.

{% highlight python %}
#text contents
text_header="""

**Homework 4 for BIOS 823**

*By Lucas Zhang*

Is there life after graduate school?
Download data of [Science and Engineering PhDs awarded in the US] 
(https://ncses.nsf.gov/pubs/nsf19301/data). 
Do some analysis in `pandas`. 
Make a [dashboard visualization](https://pyviz.org/dashboarding/) 
of a few interesting aspects of the data.

Here I visualize the tables [Doctorate recipients from U.S. colleges and 
universities: 1958–2017]
(https://ncses.nsf.gov/pubs/nsf19301/assets/data/tables/sed17-sr-tab001.xlsx) 
and [Doctorate-granting institutions, by state
or location and major science and engineering fields of study: 2017]
(https://ncses.nsf.gov/pubs/nsf19301/assets/data/tables/sed17-sr-tab007.xlsx)
to illustrate the overall tendencies of Doctoral recipients over years and 
across different institutes in the U.S.

Please also visit my [GitHub Page](https://lujun995.github.io/) and 
[GitHub repository](https://github.com/Lujun995/BIOS823)
"""
text_time = """
The figure below shows the number of PhD degree recipients rapidly increased from 1958 to 2017.
In 1958, there are less than 9,000 PhD graduates every year all over the United States. 
Six decades later, the number of PhD degree recipients increased to 55,000, four times more than
in 1958.

Feel free to use the slider bar below to specify a period of interest.
"""
text_instit = """
The figure below shows the number of PhD recipients in different majors in 2017. In the
dropdown box below, you can select institutes of interest. If the box is clear, you select
all the records.

What is the distribution of majors of PhD graduates from Duke, Harvard and 
MIT in 2017? Type Duke, Harvard and Massachusetts Institute of Technology in the box and 
select the corresponding institutes. Is what you find in line with your impression?

From the sunburst graph below, we can find that while a lot of PhD recipients graduated from 
all the three institutes in 2017, Harvard grants the most PhD degrees in life science and MIT 
is the winner in engineering.
"""
{% endhighlight %}

Thereafter, we can start to build the layout of our dashboard. In `dash`, we can use some Python-like code to build an HTML-based website. Here, we use an integrated theme "MINTY" in `dash_bootstrap_components` to help us build a beautiful dashboard. While full documentation of `dash` can be found [here](https://dash.plotly.com/), in a nutshell, we need 

1. Set up the server with "Dash" function and apply our theme MINTY
2. Set up the layout using an HTML-like Python
3. If we want to create interactive items, we need to add these items in the layout and also specify an "id" so the server can recognize these items. In `dash` dcc (dash core components), there are several items that can accept value from the users' input, such as "Slider", "RangeSlider", multiple-choice or single-choice "Dropdown" box. There are also some items that contain our text-rich contents and figures, such as "Markdown" and "Graph".
4. Update the interactive items with our specific Python function, which accepts input from "dash.dcc" and outputs them back into our dashboard.
5. Run the server and we can visit our website at http://127.0.0.1:8050/

Here comes the Python code for each step:

{% highlight python %}
#Step 1, Set up the server
app = Dash(__name__, external_stylesheets=[dbc.themes.MINTY])
{% endhighlight %}


{% highlight python %}
#Step 2 & 3, Set up the layout & Specify interactive items in the layout
app.layout = html.Div([
    dbc.Container([
        dbc.Row([
            dbc.Col(html.H1(children="How many PhDs students graduate in the U.S. every year?"), 
                    className="mb-2")
        ]),
        dbc.Row([
            dbc.Col(dcc.Markdown(children=text_header), 
                    className="mb-4")
        ]),
        html.Hr(), #a thematic break between paragraph-level elements
        
        dbc.Row([
            dbc.Col(html.H3(children="Number of PhD Graduates over Years in the U.S."), 
                    className="mb-2")
        ]),
        dbc.Row([
            dbc.Col(dcc.Markdown(children=text_time), 
                    className="mb-4")
        ]),
        dcc.Graph(id = 'fig_time'),
        dcc.RangeSlider(id = "year_range", step = 1, 
                        min = df_time['Year'].min(), max=df_time['Year'].max(),
                        value = [df_time['Year'].min(), df_time['Year'].max()],
                        updatemode = "drag", pushable = True,
                        marks = {str(year): str(year) 
                                 for year in range(df_time['Year'].min(),
                                                   df_time['Year'].max(), 10)}
                        ),
        html.Hr(),
        
        dbc.Row([
            dbc.Col(html.H3(children="Number of PhD Graduates over Different Majors in the U.S."), 
                    className="mb-2")
        ]),
        dbc.Row([dbc.Col(dcc.Markdown(children=text_instit), className="mb-4")]),
        dcc.Dropdown(id = "instit_choice", value=["Duke U."], multi=True,
                     options=[{"label": sch, "value": sch} for sch in df_instit.Location.unique()],
                     placeholder="All institutes"),
        dcc.Graph(id = 'fig_instit', style={'width': '90vh', 'height': '90vh'}),
        html.Hr(),
        
    ])
])
{% endhighlight %}

{% highlight python %}
#Step 4, Interactive Figures
@app.callback(Output("fig_time", "figure"), Input("year_range", "value"))
def ifig_time(years): #this seems directly related to the callback on the above
    df_time_ranged = df_time[(df_time["Year"] >= years[0]) &
                             (df_time["Year"] <= years[1])]
    fig_time = px.line(df_time_ranged, x="Year", y="Doctorate recipients", 
                       template = "simple_white", markers = True)
    fig_time.update_layout(transition_duration=500)
    return fig_time
#the number of graduates seems to be related to the GDP growth rate

@app.callback(Output("fig_instit", "figure"), Input("instit_choice", "value"))
def ifig_instit(schools):
    if len(schools) == 0:
        df_instit_s = df_instit
    else:
        df_instit_s = df_instit[df_instit.Location.isin(schools)]
    df_instit_s = df_instit_s.groupby(["Field", "Major"]).sum().reset_index()
    fig_instit = px.sunburst(df_instit_s, path = ['Field', 'Major'], 
                             values = "Student Number", color = "Field",
                             template = "simple_white")
    fig_instit.update_layout(transition_duration=500)
    return fig_instit
{% endhighlight %}

{% highlight python %}
#Step 5, Run the server and visit our website at http://127.0.0.1:8050/
if __name__ == '__main__':
    app.run_server(debug=True)
{% endhighlight %}

Run the Python file using "runfile", and the following is an illustration of our dashboard:


![Illustrate](/assets/img/zoom_1.gif){: .mx-auto.d-block :}
