---
layout: single
title: Ciw
excerpt: "Ciw es una biblioteca de Python para la simulación de redes de colas abiertas que tiene como objetivo habilitar las mejores prácticas dentro del dominio de la simulación de eventos discretos y producir rerultados reproducibles."
date: 2022-05-09
classes: wide
header:
  teaser: /assets/images/Captura_ciw.PNG
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - Python
  - Simulación
tags:
  - Redes de colas
  - Inventarios
---

![](/assets/images/Captura_ciw.PNG)

En este post encontraras un breve resumen del funcionamiento de la libreria Ciw de Python, asi como
algunos programas de simulación de eventos discretos y ejemplos teoricos de redes de colas.

La reducción y el cierre de las instituciones estatales de salud mental en Filadela en la década de 1990
llevaron al desarrollo de una red de atención continua de servicios residenciales. Aunque aumentó la diversidad de entornos de atención, la congestión en las instalaciones provocó que muchos pacientes pasaran innecesariamente días adicionales en instalaciones de cuidados intensivos. Este estudio aplica un sistema de red de colas con bloqueo para analizar dichos procesos de congestión. "Bloqueo"denota situaciones en las que los pacientes son rechazados de los alojamientos a los que fueron derivados y, por lo tanto, se ven obligados a permanecer en sus instalaciones actuales hasta que haya espacio disponible. Se presentan y comparan los resultados matemáticos y de simulación. Aunque los modelos de colas se han utilizado en numerosos estudios de atención médica, la inclusión del bloqueo aún es rara. Descubrimos que, en Filadela, la escasez de un tipo particular de instalaciones puede haber creado un "bloqueo aguas arriba". Por lo tanto, la eliminación de estos cuellos de botella especícos de las instalaciones puede ser la forma más eciente de reducir la congestión en el sistema en su conjunto Koizumi,Kuno y Smith,(2018).

## ¿Que es Ciw?
Ciw es una biblioteca de Python para simulación de redes de colas abiertas que tiene como objetivo habilitar las mejores prácticas dentro del dominio de la simulación de eventos discretos y producir resultados reproducibles
Palmer,Harper y Knight,(2018) mencionan que las características principales de esta biblioteca incluyen la
capacidad de simular redes de colas, múltiples clases de clientes y redes restrtingidas que muestran bloqueos.
Con las siguentes cualidades:

* El código fuente abierto y accesible promueve la comprensión, el desarrollo y la modicación. El rastreador
de problemas en línea y el entorno de desarrollo abierto fomentan el debate, la generación de ideas y
el desarrollo. La licencia permisiva permite ampliarlo, modicarlo y adaptarlo a las necesidades de los
usuarios.

* El ecosistema de Python permite que se use de manera exible dentro del lenguaje de programación, lo
que facilita la experimentación y la integración con otras herramientas cientícas. Los modelos se pueden
probar y controlar la versión.

* Los modelos son legibles y el paquete tiene una extensa documentación bilingüe para mejorar la comuni-
cación del modelo.

## Instalación 
Ciw actualmente es compatible y se prueba regularmente en las versiones de Python 3.6, 3.7 y 3.8.

```R
$ pip install ciw
```

## Líneas de espera
Suponga que es gerente de un banco y le gustaria saber cuanto tiempo esperan uss clientes en su banco.
* Los clientes llegan al azar aproximadamente 12/hora con una distribución Exponencial
* El tiempo de atención dura aproximadamente 10 minutos con una distribución Exponencial
* Cuenta con 3 servidores
¿En primedio cuanto esperan los clientes?
Solución:
Primero importamos nuestras librerias
```R
import ciw
```
Ahora debemos decirle a Ciw como se ve nuestro sistema
* number of servers = numero de servidores
* arrival distributions = distribucion de llegadas
* service distributions = distribucion de atencion

Tomar en cuenta que las unidades se consideran en minutos, por esta razón:
* Exponential(rate=0.2) describe nuestras llegadas (12/hora)
* Exponential(rate=0.1) describe el tiempo de atención del servidor
```R
N = ciw.create_network(arrival_distributions=[ciw.dists.Exponential(rate=0.2)],
	service_distributions=[ciw.dists.Exponential(rate=0.1)],
	number_of_servers=[3])
```

Ahora establecemos una semilla y creamos nuestro banco.
```R
#Establecemos una semilla 
ciw.seed(1)
#ahora creamos la simulacion
Q = ciw.Simulation(N)
```
Simulamos un día (1440 min.).
```R
Q.simulate_until_max_time(1440)
```
### Explorando nuestra simulación
Usando la comprensión de listas, podemos obtener listas de cualquier estadística que nos guste:
```R
#podemos obtener una lista de todos los registros con 
recs = Q.get_all_records()
# Una lista de tiempos del servidor
servicetimes = [r.service_time for r in recs]
# Una lista de espera
waits = [r.waiting_time for r in recs]
```
Puede obtener más información en el siguiente archivo [PDF](https://drive.google.com/file/d/1w9887JvFKDoik8pYWmqQjP0H-_FlJ6tP/view) y los regursos en el siguiente [repositorio](https://github.com/OsvaldoYa22/Ciw/tree/main)
