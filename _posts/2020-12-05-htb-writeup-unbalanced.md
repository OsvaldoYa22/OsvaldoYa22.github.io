---
layout: single
title: Leaflet 
excerpt: " Leaflet es una paquetería altamente versátil y poderosa que permite crear visualizaciones de mapas interactivos de manera fácil y eficiente en R. Puedes agregar capas de información geoespacial, como puntos, polígonos, rutas y etiquetas, y personalizarlos con estilos y colores atractivos. La interactividad proporciona una experiencia inmersiva para los usuarios, ya que pueden explorar los mapas, hacer zoom, hacer clic en los elementos y obtener información adicional."
date: 2023-05-20
classes: wide
header:
  teaser: /assets/images/Captura15.PNG
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - R
  - Mapas
tags:  
  - R
  - Mapas
  - HTML
  - Leaflet
---

![](/assets/images/Captura15.PNG)

Leaflet es una paquetería altamente versátil y poderosa que permite crear visualizaciones de mapas interactivos de manera fácil y eficiente en R. Puedes agregar capas de información geoespacial, como puntos, polígonos, rutas y etiquetas, y personalizarlos con estilos y colores atractivos. La interactividad proporciona una experiencia inmersiva para los usuarios, ya que pueden explorar los mapas, hacer zoom, hacer clic en los elementos y obtener información adicional.

## Motivación

<div style="text-align: center;">
  <div style="position: relative; padding-bottom: 86.25%; height: 0;">
    <iframe style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;" src="https://c5projects.github.io/" frameborder="0" allowfullscreen></iframe>
  </div>
</div>

## Cargamos los datos y capas para nuestro mapa
```R
base <- read.csv("THALES_BASE_2019_2020_parte_1.csv", header = T,stringsAsFactors = F)
#SHP alcaldias
ALCALDIAS <- spTransform(shapefile("POLIGONOS/ALCALDIAS.shp"), "+proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0")
#SHP vialidades primarias
VIALIDADES_PRIMARIAS <- spTransform(shapefile("POLIGONOS/VIALIDADES CATEGO_PRIMARIA.shp"), "+proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0")
#Ordenamos las variables de los tipos de delitos
orden <- order(unique(base$incidente_c4))
casos <- unique(base$incidente_c4)
casos[orden]
Robo_Vehiculo_con_Violencia <- base[which((base$incidente_c4) == "Robo-Vehiculo con Violencia"),]
```
## Diseño del mapa

Titulo 
```R
tag.map.title <- tags$style(HTML("
  .leaflet-control.map-title { 
    transform: translate(-50%,20%);
    position: fixed !important;
    left: 50%;
    text-align: center;
    padding-left: 10px; 
    padding-right: 10px; 
    background: TRANSPARENT;
    font-weight: bold;
    font-size: 25px;
    color: #9F2241}"))
title <- tags$div(tag.map.title, HTML(paste("Robo a vehículos 2019<br>" )))
```
Simbologia

```R
colors <- c("transparent","green","YELLOW","ORANGE","red") ## Definimos los colores de nuestras figuras

labels <- c("<b>INCIDENCIA</b>",   ## Definimos la descripcion de cada uno de nuestros simbolos
            "BAJA",
            "MEDIA",
            "ALTA",
            "MUY ALTA"
            )  

sizes <- c(0,10,10,10,10)  ## Definimos el tamaño de nuestras figuras
shapes <- c("circle","square","square","square","square") ## Definimos la forma de nuestra figura
borders <- c("transparent","green","YELLOW","ORANGE","red") ## Definmos el color del borde de nuestra figura
```

Agregamos nuestros vectores definidos anteriormente al mapa
```R
addLegendCustom <- function(map, colors, labels, sizes, shapes, borders, opacity = 0.5){
  make_shapes <- function(colors, sizes, borders, shapes) {
    shapes <- gsub("circle", "50%", shapes)
    shapes <- gsub("square", "0%", shapes)
    paste0(colors, "; width:", sizes, "px; height:", sizes, "px; border:3px solid ", borders, "; border-radius:", shapes)
  }
  make_labels <- function(sizes, labels) {
    paste0("<div style='display: inline-block;height: ", 
           sizes, "px;margin-top: 4px;line-height: ", 
           sizes, "px;'>", labels, "</div>")
  }
  
  legend_colors <- make_shapes(colors, sizes, borders, shapes)
  legend_labels <- make_labels(sizes, labels)
  
  return(addLegend(map, colors = legend_colors, labels = legend_labels, opacity = opacity,position = "bottomleft"))
}
```
## Generamos nuestro mapa 
Usamos la siguientes lineas de comando para iniciar nuetro mapa con la libreria  `leaflet`

```R
leaflet()  %>%
```

### Capas
Agremamos fierentes capas de estilo a nustro mapa 
Cada `addTiles()` que se agrega le asignamos un grupo para posteriormente poder agregar ese filtro al addlayer
```R
  addTiles() %>% addProviderTiles(providers$Esri.WorldGrayCanvas, group = "B&W") %>%
  addTiles(urlTemplate = "http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", 
           attribution = NULL, layerId = NULL, options = tileOptions(),group = "Color")%>%
  addTiles(urlTemplate = "https://mts1.google.com/vt/lyrs=s&hl=en&src=app&x={x}&y={y}&z={z}&s=G", 
           attribution = NULL, layerId = NULL, options = tileOptions(),group = "Satelital")%>%
```
### Poligonos
De la misma forma que agregamos distintos estilos de mapa podemos agregar nuestras propias capas castograficas 
```R
## agregamos los poligonos de alcaldias de la CDMX aquí tambien se modifica el color, la opacidad del color como y se agrega al grupo que va a pertencer
  addPolygons(data=ALCALDIAS,color = "black",fillColor = "transparent",fillOpacity =0.01,weight = 1,popup = ALCALDIAS$ALCALDIA,
              highlightOptions = highlightOptions(color = "red", weight = 1) , group="Alcaldias",options = pathOptions(pane="polygons"))%>% 
## agregamos el poligono de vialidades principales de la CDMX
  addPolylines(data=VIALIDADES_PRIMARIAS,color = "#8B0000",weight = 0.8, group="Primarias",options = pathOptions(pane="polygons"))%>%
```
### Puntos
De una froma similar agregamos nuestros puntos, al ser puntos nosotros contamos con longitud y latitud en nuestra base de datos, por la anto le decimos a nuestra función como se muestran esas celdas para que las interprete y las muestre en el mapa

```R
  addCircles(data = Robo_Vehiculo_con_Violencia,lng = Robo_Vehiculo_con_Violencia$longitud,lat = Robo_Vehiculo_con_Violencia$latitud,color = "#9AC0CD" ,radius = 5,fillOpacity = T,
             popup = paste("<b>","Tipo : ","</b>",as.character(Robo_Vehiculo_con_Violencia$clas_con_f_alarma),"<br>",
                           "<b>","Categoria : ","</b>",as.character(Robo_Vehiculo_con_Violencia$incidente_c4),"<br>"),
             group = paste("Puntos","(",nrow(Robo_Vehiculo_con_Violencia),")"),options = pathOptions(pane="li"))%>%
```
### Mapa de calor 
Con la función `addWebGLHeatmap` agreamos a nuestro mapa un acumulado de esos puntos mediante una gama de colores, es decir se crea un mapa de calor
```R
addWebGLHeatmap(data = Robo_Vehiculo_con_Violencia,lng = Robo_Vehiculo_con_Violencia$longitud,lat = Robo_Vehiculo_con_Violencia$latitud, group = "Mapa de calor",size=700,gradientTexture = "skyline",
                  opacity = 0.7 )%>%
```
### Filtros
Agregamos nuestro addLayer, estos son los grupos que creamos anteriormente para que se puedan filtrar los datos según lo necesitemos
```R
addLayersControl(overlayGroups = c( "&nbsp; <b>Vista </b> &nbsp; ", ## Esto se ve como un titulo ya que no pertenece a ninguna clase
                                      "Mapa de calor",
                                      paste("Puntos","(",nrow(Robo_Vehiculo_con_Violencia),")"),
                                      ## con la función paste nosotros podemos ver cual es el acumulado de nuestros puntos
                                      "&nbsp; <b>Vialidades</b> &nbsp; ", ## Esto se ve como un titulo ya que no pertenece a ninguna clase
                                      "Primarias",
                                      
                                      "&nbsp; <b>División </b> &nbsp; ", ## Esto se ve como un titulo ya que no pertenece a ninguna clase
                                      "Alcaldias",
                                      
                                      "&nbsp; <b>Capas</b> &nbsp; ",## Esto se ve como un titulo ya que no pertenece a ninguna clase
                                      "B&W",
                                      "Color",
                                      "Satelital"
                                      
  ),
  options = layersControlOptions(collapsed = T))%>% 
```
Ahora necesitamos darle un fromato a nuestro addLayer, para que nuestros "titulos" tengan un fondo del color que nosotros le indiquemos y la posición que necesitemos para las descripciones de nuestros addLayers
```R
htmlwidgets::onRender(jsCode = htmlwidgets::JS("function(btn,map){ 
                                                 var lc=document.getElementsByClassName('leaflet-control-layers-overlays')[0]
                                                 
                                                 lc.getElementsByTagName('input')[0].style.display='none';
                                                
                                                 lc.getElementsByTagName('div')[0].style.fontSize='160%';
                                                 lc.getElementsByTagName('div')[0].style.textAlign='center';
                                                 lc.getElementsByTagName('div')[0].style.color='white';
                                                 lc.getElementsByTagName('div')[0].style.backgroundColor='#9F2241';
                                                 
                                                 lc.getElementsByTagName('input')[3].style.display='none';
                                                
                                                 lc.getElementsByTagName('div')[3].style.fontSize='160%';
                                                 lc.getElementsByTagName('div')[3].style.textAlign='center';
                                                 lc.getElementsByTagName('div')[3].style.color='white';
                                                 lc.getElementsByTagName('div')[3].style.backgroundColor='#9F2241';
                                                 
                                                 lc.getElementsByTagName('input')[5].style.display='none';
                                                
                                                 lc.getElementsByTagName('div')[5].style.fontSize='160%';
                                                 lc.getElementsByTagName('div')[5].style.textAlign='center';
                                                 lc.getElementsByTagName('div')[5].style.color='white';
                                                 lc.getElementsByTagName('div')[5].style.backgroundColor='#9F2241';
                                                 
                                                 lc.getElementsByTagName('input')[7].style.display='none';
                                                
                                                 lc.getElementsByTagName('div')[7].style.fontSize='160%';
                                                 lc.getElementsByTagName('div')[7].style.textAlign='center';
                                                 lc.getElementsByTagName('div')[7].style.color='white';
                                                 lc.getElementsByTagName('div')[7].style.backgroundColor='#9F2241';
                                                 
                                                 
   ;
                                                 }
                                                 ")) %>%
```
Obtenga el código completo [aquí](https://github.com/OsvaldoYa22/leaflet)

