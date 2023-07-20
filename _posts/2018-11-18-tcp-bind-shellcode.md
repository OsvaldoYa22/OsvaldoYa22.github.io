---
layout: single
title: Robos - app 
excerpt: "Es una libreria de interfaz de usuario/framework de Python para crear aplicaciones web analíticas, por ejemplo el análisis de datos, exploración de datos, visualización, modelado, control de instrumentos e informes.  Dash se basa en Flask, React y Plotly, y combina la potencia de estas tecnologías para proporcionar una manera sencilla de crear aplicaciones web interactivas con visualizaciones de datos dinámicas."
date: 2023-07-10
classes: wide
header:
  teaser: /assets/images/slae32.png
categories:
  - Python
tags:
  - Python
  - Dash
  - Plotly
---
## Requirements
```python
from dash import Dash, dcc, html
from dash.dependencies import Input, Output
import dash_bootstrap_components as dbc
```

## Estilos externos
```python
external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css',
                        [dbc.themes.BOOTSTRAP]]
```
Se definen dos estilos externos para la aplicación Dash. Uno de ellos es un enlace a una hoja de estilos CSS alojada en CodePen 'https://codepen.io/chriddyp/pen/bWLwgP.css'. El otro estilo externo es de Bootstrap (dbc.themes.BOOTSTRAP) que se utilizará para darle estilo y formato a la aplicación.
## Inicialización de la aplicación Dash
```python
app = Dash(__name__, external_stylesheets = external_stylesheets, suppress_callback_exceptions = True)
```
Se crea una instancia de la clase Dash y se asigna a la variable app, los parámetros proporcionados son:
* `__name__` es el nombre del módulo actual.
* `external_stylesheets` se pasasn los estilos externos definidos anteriormente
* `suppress_callback_exceptions` se establece esta opcion como `True`por lo que la aplicación suprimirá las excepciones de devolución de llamada. Esto permite manejar de manera más fexible los errores en las devoluciones de llamada.

## Diseño de la aplicación 
```python
app.layout = html.Div([
    html.Div([
        html.H1('ROBOS EN CDMX', style = {'textAlign': 'center', 'height': '75px', 'margin': '0px -10px 10px', 'background-color': '#FFFFFF', 'border-radius': '2px', 'display': 'block'})]),
    dcc.Tabs(id = "tabs", value = 'tab-1', children = [
        dcc.Tab(label = 'ANALISIS DESCRIPTIVO', value = 'tab-2'),
        dcc.Tab(label = 'ANALISIS ECONOMETRICO', value = 'tab-1')]),
    html.Div(id = 'tabs-content')
])
```
El diseño de la aplicación se define utilizando componentes de la librería `html` y `dcc` de Dash. En este caso, se crea un diseño con dos pestañas `dcc.Tabs` y un contenedor de contenido `html.Div` donde se mostrará el contenido correspondiente a la pestaña seleccionada.
* Se crea un encabezado con un título ("ROBOS EN CDMX") dentro de una `html.Div`.
* Luego, se definen las pestañas con etiquetas "ANALISIS DESCRIPTIVO" y "ANALISIS ECONOMETRICO".
* El contenido de cada pestaña se establece mediante un callback

## Callbacks
```python
@app.callback(Output('tabs-content', 'children'),
              Input('tabs', 'value'))
def render_content_tab(tab):
    return render_content_table(tab)
```
En Dash, los callbacks se utilizan para actualizar dinámicamente el contenido de la aplicación en respuesta a cambios en los componentes de entrada. Los callbacks están definidos utilizando el decorador `@app.callback.`
* El primer callback se activa cuando se cambia de pestaña `Input('tabs', 'value')` y su función asociada es `render_content_tab(tab)`. Esta función se encargará de mostrar el contenido correspondiente a la pestaña seleccionada en el contenedor de contenido `tabs-content`.

## Funciónes
### Estructura de cada tabla
Llamamos a nuestras librerias
```Python
from dash import dcc, html
import dash_bootstrap_components as db
```
Definimos a nuestro `render_content_table(tab)` el cual su contenido va a depender del valor de `tab`
```python
def render_content_table(tab):
    if tab == 'tab-2':
        return html.Div([])
    elif tab == 'tab-1':
        return html.Div([])
```
Dentro de cada `html.Div` se mostrara el analsis "Econometrico" o "Descriptivo" según seleccione el usuario
* Descriptivo

  ```python
  html.Div([
                ## agregamos un espacio
                html.Br(),
                ## agregamos texto
                html.P('Seleccione el origen de los datos',
                        style = {'font-weight': 'bold','background-color': '#F2F2F2',
                                'border-radius': '3px','margin': '5px',
                                'padding': '7px','position': 'relative','display': 'inline-block',
                                'box-shadow': '4px 4px 4px 4px lightgrey',
                                'overflow-y': 'auto'
                            }),
                ## agreamos las opciones de nuestros ogiren de datos, como la imagen correspondiente
                dcc.RadioItems(id = 'origing_data',
                               labelStyle = {'display': 'inline-block',"align-items": "center", 'textAlign': 'center'},
                               options = [
                                   {'label': [
                                       html.Img(src="/assets/FGJ.png", height=60),                                       
                                   ], 'value': 'Data_FGJ'},
                                   {'label': [
                                       html.Img(src="/assets/C5.png", height=60),                                       
                                   ], 'value': 'Data_C5'}
                               ], value = 'Data_FGJ',                               
                               ),
                
                html.Br(),
                ## agregamos un Slider para seleccionar el año de la información que necesitamos
                dcc.Slider(
                    id="slider-circular", min=2016, max=2023,
                    marks={str(year): str(year) for year in range(2016, 2024)},
                    value=2016, step=1
                ),
                html.Div(id = 'tabs-content02')
                               
                ]),
  ```
  * Se crea un contenedor `Div` que agrupa varios elementos dentro de él
  * Se agrega un salto de línea `<br>` para introducir un espacio entre elementos.
  * Se agrega un párrafo `<p>`que muestra el texto "Seleccione el origen de los datos". El texto tiene algunos estilos CSS definidos, como negrita `'font-weight': 'bold'`, un fondo de color `'background-color': '#F2F2F2'`, un borde redondeado `'border-radius': '3px'`, margen `'margin': '5px'` y relleno `'padding': '7px'`. El párrafo también tiene un sombreado `'box-shadow'` para dar una apariencia visual de relieve.
  * Se crea un conjunto de elementos de opción `RadioItems` que permite al usuario seleccionar una de las dos opciones: "Data_FGJ" o "Data_C5". Cada opción tiene una etiqueta que incluye una imagen `html.Img` que se mostrará junto al texto de la opción. El valor predeterminado seleccionado es "Data_FGJ".
  * Se agrega un control deslizante `Slider` que permite al usuario seleccionar un año entre 2016 y 2023. El valor predeterminado es 2016. Se muestran marcas en el control deslizante para cada año entre 2016 y 2023.
* Econometrico
    ```python
    html.Div(
            style = {'display': 'flex', 'flex-wrap': 'wrap'},
            children = [
                html.Div(
                    style = {'width': '50%'},
                    children = [
                        html.Div(
                            style = {'background-color': '#F2F2F2','border-radius': '15px',
                                'margin': '5px','padding': '7px','position': 'relative',
                                'box-shadow': '4px 4px 4px 4px lightgrey','overflow-y': 'auto'},
                            children = [
                                dcc.Graph(id = 'Mi_grafica_ST_2', figure = {}),
                            ]
                        )
                    ]
                )
    ```
    * Se crea un componente de gráfica `Graph` utilizando Dash. Se asigna el identificador `'Mi_grafica_ST_2'` a la gráfica y se inicializa con una figura vacía `figure = {}`. Este identificador será útil para actualizar el contenido de la gráfica más adelante mediante un callback.
## Graficas
1. ```python
   def update_graph_01_2016(value):
    fig = None
    if value == 'Data_FGJ':
        fig = px.treemap(conteo_colonia_2016, path=[px.Constant("Ciudad De México"), 'NOMDT'], values='ROBOS',
                 color='NOMDT',
                 color_continuous_scale='orrd',
                 hover_data={'NOMUT': True, 'ROBOS': ':,.0f'})
        
        fig.update_traces(marker=dict(cornerradius=5))
        fig.update_traces(hovertemplate='<b>%{label}<br>ROBOS: %{customdata[1]:,.0f}')

        fig.update_layout(margin = dict(t = 15, l = 7, r = 7, b = 5,)) 
    else:
        x = np.linspace(0, 10, 100)
        y = 2 * x  
        fig = px.line(x = x, y = y)
    return fig
   ```
2. ```python
    def update_graph_02_2016(value):
      if value == 'Data_FGJ':
          fig = px.treemap(conteo_fecha_2016, path = [px.Constant("Robos CDMX 2016"), 'mes','dia'], values = 'Frecuencia',
                           color = 'mes', 
                           color_continuous_scale = 'orrd',
                           hover_data={'Frecuencia': ':,.0f'},)
          fig.update_traces(marker=dict(cornerradius=5), 
                    hovertemplate='<b>%{label}<br>ROBOS: %{customdata[0]}',
                    insidetextfont=dict(color='white'),  # Opcional: para que el texto dentro de las cajas sea blanco
                    textfont_color='black',  # Opcional: para que el texto fuera de las cajas sea negro )
          fig.update_layout(margin = dict(t = 10, l = 7, r = 7, b = 5))   
      else:
          x = np.linspace(0, 10, 100)
          y = 6 * x  
          fig = px.line(x = x, y = y)    
      return fig
   ```
3. ```python
   def update_graph_04_2016(value):
    if value == 'Data_FGJ':
        fig = px.choropleth_mapbox(shape_2016, geojson = shape_2016.geometry, locations = shape_2016.index, color = "ROBOS",
                                   color_continuous_scale = 'orrd',
                                   mapbox_style = "open-street-map",
                                   zoom = 9.7, center = {"lat": 19.4326, "lon": -99.1332},
                                   opacity = 0.5,)
        fig.add_trace(
            go.Scattermapbox(
                lat=shape_2016.geometry.centroid.y,
                lon=shape_2016.geometry.centroid.x,
                mode='markers',
                hovertext=shape_2016['NOMDT'] + '<br>' + shape_2016['NOMUT'] + '<br>ROBOS: ' + shape_2016['ROBOS'].apply(lambda x: f'{int(x):,}'),
                hoverinfo='text',
                marker=dict(
                    size=5,
                    opacity=0),
                hoverlabel=dict(
                    bgcolor='#FEDDDD')))
        fig.update_layout(
            margin=dict(t=10, l=7, r=7, b=5),
            coloraxis_showscale=False)
    else:
        x = np.linspace(0, 10, 100)
        y = 6 * x
        fig = px.line(x=x, y=y)
   ```
