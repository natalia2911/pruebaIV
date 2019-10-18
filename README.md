[![License: LGPL v3](https://img.shields.io/badge/License-LGPL%20v3-blue.svg)](https://www.gnu.org/licenses/lgpl-3.0)
# Proyecto Cloud Computing


Este es el repositorio relacionado con el proyecto realizado por Natalia María Mártir Moreno sobre la asignatura de Cloud Computing del Máster de Ingeniería Informática de Granada.



## Descripción del proyecto

Mi proyecto va a basarse en una serie de microservicios en la nube en los que aparezcan un microservicio encargado de la gestión de los exámenes que puede tener un alumno matriculado, otro microservicio que gestione los alumnos, y otro que se encargue de mandarle mensajes al alumno.

Estos microservicios están pensados para incluirse en un ***calendario personalizado para los alumnos de la ETSIIT***

[Información más detallada sobre el proyecto](https://github.com/natalia2911/Proyecto-CloudComputing/blob/master/Documentación/DescripcionProyecto.md)

## Arquitectura


Nuestro proyecto se va a basar en una arquitectura de microservicios.
Hemos pensado que cada uno de estos microservicios puedan estar implementados en lenguajes diferentes.

En conclusión tenemos los siguientes microservicios:

- Gestión de exámenes.
- Gestión de alumnos.
- Paso de mensajes.

El lenguaje que vamos a usar va a ser Ruby, aunque la intención principal es desarrollar cada microservicio en un lenguaje diferente pudiendo ser Python, Java, entre otros.

Por otro lado hemos comentado que necesitaremos el acceso a una base de datos, para obtener la información del alumno, aun no hemos concretado cuál será pero optamos por una base de datos no relacional, debido a que los datos están optimizados y se logra una gran accesibilidad en el tratamiento de los mismos, por lo que una de las posibles candidatas podría ser **MongoDB**.

La comunicación entre los microservicios se va a realizar mediante paso de mensajes, para ello tendremos que implementar un sistema broker como por ejemplo **RabbitMQ**
RabbitMQ es el intermediario de mensajes de código abierto más usado. Lo hemos elegido ya que es uno de los más ligeros y fáciles de implementar con microservicios en la nube, tiene un implementación distribuida y utiliza mensajería asíncrona.

[Más información sobre RabbitMQ](https://www.rabbitmq.com/)

### Diagrama de la arquitectura
---
![diagramaArquitectura](https://github.com/natalia2911/Proyecto-CloudComputing/blob/master/img/diagrama.png)


## Licencia

Este proyecto tiene la licencia ***GNU General Public License v3.0*** debido a que es una de las licencias con más permisividad en el ámbito de estudiar, compartir o modificar el software.
