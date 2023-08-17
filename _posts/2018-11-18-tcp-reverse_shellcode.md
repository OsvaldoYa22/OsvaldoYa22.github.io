---
layout: single
title: Geospatial Visualizations with Dash and Folium
date: 2023-08-17 
classes: wide
header:
  teaser: /assets/images/Captura_folium_02.PNG
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - Python
tags:
  - Folium
  - Dash
---
![](/assets/images/Captura_folium_02.PNG)
In this article, a brief explanation is provided about creating an interactive application that combines geographical data with powerful visualization tools. The application allows filtering points or polygons according to user needs. Additionally, the `Geocoding API` and `Street View Static API` are used to enhance the user's visual experience. As a result, it's possible to download the map with the selected filters into an HTML file, utilizing Python.




<iframe width="600" height = "420"
src="https://youtu.be/fwHeeGhOidw">
</iframe>




**Importing Libraries and Preparation**

In this section, we start by importing the necessary libraries for our project. Dash and Folium are the cornerstones of this application, while other libraries such as Pandas, KeplerGL, and Geopandas will assist us in working with data and creating visualizations.

```python
from dash import Dash, dcc, html, Input, Output, callback
import base64
from dash.dependencies import Input, Output, State
import pandas as pd
import folium
from keplergl import *
import dash
import dash_bootstrap_components as dbc
import geopandas as gpd
import folium
from folium import plugins
from fun_style import *
```

**Setting Up the Dash Application**

In the following code snippet, we are using the `dash.Dash` class to create our application.

```python
app = dash.Dash(__name__, external_stylesheets = [dbc.themes.BOOTSTRAP], suppress_callback_exceptions = True)
```

`external_stylesheets`: Here we are passing a Bootstrap stylesheet using `dbc.themes.BOOTSTRAP`. This allows us to apply an appealing and responsive visual design to our application.

`suppress_callback_exceptions`: Setting this to True allows us to handle callback exceptions without disrupting the entire application in case of errors in the callbacks.

**Creating the Sidebar of Options**

Here, we are defining the sidebar of options that will enable users to interact with the geospatial visualization in an intuitive manner. This part is crucial to provide a smooth user experience and facilitate the exploration of geographical data.

```python
sidebar = html.Div(
    [
        html.Div(
            children = [
                html.Img(src = "/assets/Logo.PNG", height = 150, style = {"width": "100%"}),
            ]
        ),
        html.Hr(),
        .....
        .....
        .....
```

```python
# Logo Section
html.Div([
    html.Img(src = "/assets/Logo.PNG", height = 150, style = {"width": "100%"}),
]),

```
The image is loaded from the path `/assets/Logo.PNG` and a style is applied to adjust its height to 150 pixels and a width of 100%.

```python
# Clear Filters Button
html.Div([
    dbc.Button("Limpiar filtros", outline = True, id = "f5-button", color = "primary", className = "me-1", n_clicks = 0),
], className = "d-grid gap-2"),

```
Here, a button is created using the `dash_bootstrap_components` library from Bootstrap. The button is used to reset the filters applied in the interface. It is assigned a unique ID `f5-button`, which can be used to perform actions in response to user clicks.

```python
# Classification Section with Checkboxes
html.H3('Clasificación', style = CONTENT_BOX_TITLES),
html.Div(
    dcc.Checklist(
        id = "classification",
        options = [
            {'label': 'Bares', 'value': 'Bares'},
            {'label': 'Centros nocturnos', 'value': 'Centros nocturnos'},
        ],
        inputStyle = {'margin-right': '4px'},
    ),
    style = {
        'background-color': '#F1F2F2',
        'border-radius': '15px',
        'margin': '5px',
        'padding': '7px',
        'position': 'relative',
        'overflow-y': 'auto',
    },
),
```
This section displays a checklist `checkboxes` that allows the user to select different sorting options, such as `Bars` and `Nightclubs`. The checklist is created using the `dcc.Checklist` component of Dash. The section has a style that defines the background, rounded borders, and scrolling style if there are too many items to fit in the available space.

```python
# HTML Map Download Section
html.Div([
    html.Div([
        dbc.Button(
            "Descargue el mapa en HTML",
            id = "open-lg",
            outline = True,
            color = "primary",
            n_clicks = 0,
        ),
    ], className = "d-grid"),
    dbc.Modal(
        [
            # Modal Content
        ],
        id = "modal-lg",
        size = "sm",
        is_open = False,
        keyboard = False,
        backdrop = "static",
    ),
]),

```
In this section, a button is displayed that, upon clicking, will open a modal providing instructions for generating and downloading a map in HTML format. The modal is defined using `dbc.Modal` and contains informative content on how to generate the HTML file of the map.

**Implementing Callbacks to Switch Tabs**

In this part of the code, we are utilizing callback functionality with `@callback` to change the content of a section based on the tab selected by the user.

```python
@callback(Output('tabs-content-inline', 'children'),
          Input('tabs-styled-with-inline', 'value'))
def render_content(tab):
    if tab == 'tab-1':
        return html.Div([            
            html.Div(
                children = [                        
                    html.Div(
                        style = {
                            'background-color': '#FFFFFF',
                            'margin': '15px',
                            'padding': '0px', 
                            'position': 'relative',
                            'box-shadow': '4px 4px 4px 4px lightgrey',
                            'width': '100vw',
                            'height': '55vh'
                        },
                        children = [
                            html.Div(id = 'right-column_map'),                          
                        ]
                    )            
                ],
                style = CONTENT_STYLE 
            ),               
        ])

```

`Output('tabs-content-inline', 'children')`: This sets that the content of the section with the identifier `tabs-content-inline` will change according to the selected tab.

`Input('tabs-styled-with-inline', 'value')`: This takes the value of the tab selected by the user from the component with the identifier `tabs-styled-with-inline`.

**Structuring the Overall Layout of the Application**

Here, the sidebar of options, the tabs, and the container where content will change based on the selected tab are included.

 ```python
 app.layout = html.Div([
    sidebar,                       
    html.Div([
        dcc.Tabs(id = "tabs-styled-with-inline", value = 'tab-1', children = [
            dcc.Tab(label = 'CONSUMO DE ALCOHOL', value = 'tab-1', style = style_tabs, selected_style = tab_selected_style),
            dcc.Tab(label = 'VENTA DE ALCOHOL', value = 'tab-2', style = style_tabs, selected_style = tab_selected_style),
        ], style = CONTENT_STYLE),
        html.Div(id = 'tabs-content-inline'),    
    ])
])
 ```
**Function to Generate the Interactive Map**
```python
@app.callback(
    Output('right-column_map', 'children'),
    [Input('alcaldias', 'value'), 
     Input('classification', 'value'),    
     Input('capas', 'value'),
     Input('poligonos-carga','value'), 
     Input('colonia-dropdown','value'),    
     ]     
)
def generate_map(selected_location, selected_classification,
                 selected_capas, poli, selected_colonia):
```
Within the `generate_map` function, data processing and filtering are performed based on user selections. For example:

```python
if selected_location:
    data = data[data['municipio'].isin(selected_location)]
if selected_classification:
    data = data[data['nombre_act'].isin(selected_classification)]
```
These lines filter the dataset based on the `alcaldías` (boroughs) and `clasificaciones` (classifications) selected by the user.

Then, a `Folium` map is created based on the user's layer selection.

```python
if selected_capas == "1":
    m = folium.Map(tiles = "OpenStreetMap")
elif selected_capas == "2":
    m = folium.Map(tiles = "Cartodb Positron")
# ...
```
Markers are generated on the map for each location in the data:
```python
for index, row in data.iterrows():
    longitude = row['longitud']
    latitude = row['latitud']
    # ... 
```

To incorporate images and Google Maps links into our map, we use Google APIs, specifically the `Geocoding API` and the `Street View Static API`, using a short code snippet in the `R` language.

```r
library(dplyr)        
library(httr)         
library(stringr)      
library(tidyr)  

base <- read.csv("C:/Users/.csv", fileEncoding = "Latin1",header = T,stringsAsFactors = F)

api_Key <- "Your API key"

generate_streetview_link <- function(lat, lon, key) {
  base_url <- "https://maps.googleapis.com/maps/api/streetview"
  query_params <- list(
    location = paste(lat, lon, sep = ","),
    size = "640x480",
    key = key
  )
  url <- modify_url(base_url, query = query_params)
  return(url)
}

base$IMAGEN <- mapply(generate_streetview_link, base$latitud, base$longitud, api_Key)

generate_streetview_url <- function(lat, lon) {
  base_url <- "https://www.google.com/maps/@"
  coords <- paste(lat, lon, sep = ",")
  url <- paste(base_url, coords, ",3a,75y,114.26h,90t/data=!3m4!1e1!3m2!1s", coords, "!2e0", sep = "")
  return(url)
}
base$MAPS <- mapply(generate_streetview_url, base$latitud, base$longitud)

```
Now we can integrate the images and links into our map.
```python
for index, row in data.iterrows():
        longitude = row['longitud']
        latitude = row['latitud']

        especialidad = row['nombre_act']
        id_bct_o = row['nom_estab']
        web_df = row['www_2']
        imagen_url = row['IMAGEN']  
        link_url = row['MAPS']  
        color = tipo_poste_colors.get(especialidad, 'gray')

        popup_content = f'<a href="{link_url}" target="_blank">' \
                        f'<img src="{imagen_url}" alt="{especialidad}" width="200" height="150"></a><br>' \
                        f'Clasificación: {especialidad}<br>' \
                        f'Nom. establecimiento: {id_bct_o}<br>'\
                        f'Sitio Web: <a href="{web_df}" target="_blank">{web_df}</a><br>'
        
        tooltip_content = f'Clasificación: {especialidad}<br>' \
                         f'Nom. establecimiento: {id_bct_o}<br>'\
                         f'Sitio Web: <a href="{web_df}" target="_blank">{web_df}</a><br>'

        folium.CircleMarker(
            [latitude, longitude],
            radius = 4,
            color = color,
            fill = True,
            fill_opacity = 1,
            weight = 1,

            popup = folium.Popup(popup_content, max_width = 300),
            tooltip = folium.Tooltip(tooltip_content),
        ).add_to(m)
```

Adding the Legend to the Map
```python
for type, count in camras_count.items():
        color = tipo_colors.get(type, 'gray')
        legend_html += f'<tr><td><span style="background-color:{color}; padding: 6px; border-radius: 50%; display: inline-block;"></span> {type}</td><td>{count}</td></tr>'

    legend_html += '</table></div>'
    m.get_root().html.add_child(folium.Element(legend_html))
```
This `for` loop iterates through each key-value pair in the `camras_count` dictionary.

`type` represents the type of element, and `count` represents the quantity associated with that type of element. The code searches the `tipo_colors` dictionary for the color associated with the type of element. If a color is not found for that type of element in the dictionary, the default color 'gray' is assigned. The dictionary is defined in the `fun_style.py` file as follows:
```python
tipo_colors = {
        'Bares' : '#FF0000',
        'Centros nocturnos' : '#FF6E00',
    }
```   
A `span` element with a background color representing the color associated with the element type is used. The name of the element type is displayed alongside this colored element, and the quantity is also displayed. This is added to the `legend_html` string, and the string is appended as an HTML element to the interactive map `m` using the `add_child()` method.

Defining the boundaries of the visible map area;
```python
sw = data[['latitud', 'longitud']].min().values.tolist()
ne = data[['latitud', 'longitud']].max().values.tolist()
m.fit_bounds([sw, ne])
```
These points are based on the minimum and maximum values of the `latitude` and `longitude` columns in the `data` DataFrame. This means that the most extreme coordinates in terms of `latitude` and `longitude` of all points in the `data` dataset are being determined.

Finally, the map is rendered as an HTML `iframe` element and returned as the result of the callback.
```python
map_html = m.get_root().render()
return html.Iframe(srcDoc = map_html, style = {'width': '100%', 'height': '600px'})
```
The `srcDoc` attribute is used to specify the HTML content that will be displayed inside the `<iframe>`. In this case, the HTML content is the previously generated map stored in the `map_html` variable.

**Function to Download the Map in HTML Format**

```python
@app.callback(
    Output('download-map', 'href'),  
    Input('btn_map', 'n_clicks'),
    State('alcaldias', 'value'),
    State('classification', 'value'),
    State('capas', 'value'), 
    State('poligonos-carga','value'), 
    State('colonia-dropdown','value'),
    prevent_initial_call = True,
)
def download_map(n_clicks,selected_location,selected_classification,  
                 selected_capas,poli,selected_colonia):
    
    map_iframe = generate_map(selected_location,selected_classification,
                 selected_capas,poli,selected_colonia)
    map_html = map_iframe.srcDoc
    encoded_map = base64.b64encode(map_html.encode()).decode()
    href = f"data:text/html;charset=utf-8;base64,{encoded_map}"

    if n_clicks:
        return href

    return dash.no_update
```
`map_iframe`: Here, the `generate_map` function we discussed earlier is called to generate the interactive map.

`map_html`: The HTML content of the generated map is obtained.

`encoded_map`: The HTML content is encoded in base64 to generate a text string that can be embedded in the download link.

`href`: The complete download link is formed here, with the content encoded in `base64` and the `MIME` type specified.

**Function to Toggle Modal Visibility**

A modal is a popup window that typically displays additional information or options to the user.

```python
def toggle_modal(n1, is_open):
    if n1:
        return not is_open
    return is_open

app.callback(
    Output("modal-lg", "is_open"),
    Input("open-lg", "n_clicks"),
    State("modal-lg", "is_open"),
)(toggle_modal)

```

**Function to Reset Filters to Their Initial Values**

This function implements a callback that resets the values of various components in your Dash application to their initial states. It allows the user to reset filters and selections to their initial settings.

```python
@app.callback(
    [Output('alcaldias', 'value'), 
     Output('classification', 'value'),    
     Output('capas', 'value'), 
     Output('poligonos-carga','value'), 
     Output('colonia-dropdown','value'),      
     ], 
    Input("f5-button", "n_clicks"),
    prevent_initial_call = True,
)
def redirect_to_self(n):
    selected_location = []
    selected_classification = []
    selected_colonia = []
    selected_capas = []
    poli = []
    return (selected_location,selected_classification,
                 selected_capas,poli,selected_colonia)
```

**Function to Update Dropdown Menu Options for 'colonias'**

```python
@app.callback(
    Output("colonia-dropdown", "options"),
    Input("alcaldias", "value")     
)
def update_sector_dropdown_02(selected_location):
    data = pd.read_csv("assets/BASES/BASE.csv", encoding='latin-1', low_memory=False)

    if selected_location is None or selected_location == []:
        unique_colonias = data['nomb_asent'].unique()
    else:
        data = data[data['municipio'].isin(selected_location)]
 
        unique_colonias = data['nomb_asent'].unique()
```

```python
options = [{'label': colonia, 'value': colonia} for colonia in unique_colonias]
```

A list of options is created in the format required by the dropdown menu `{'label': ..., 'value': ...}` using the unique neighborhoods obtained.
```python
return options
```
The updated list of options is returned so that the neighborhood dropdown menu reflects the neighborhoods corresponding to the selected boroughs.

**Main Execution Block of the Application**
```python
if __name__ == "__main__":
    app.run_server(debug=True, port=9050)
```

`debug=True` enables the debug mode, which means detailed messages will be displayed in case of errors.

`port=9050` sets the port number on which the server will run. In this case, port 9050 is being used.
