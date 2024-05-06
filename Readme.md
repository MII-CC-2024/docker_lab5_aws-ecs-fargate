# Docker: python y Flask (Hola Mundo) en AWS ECS


## Crea la aplicación

Crea un entorno virtual, actívalo, instala Flask y desarrolla tu aplicación.
(aquí no se muestran estos pasos, partimos de la aplicación ya desarrollada y las librerías necesarias indicadas en el fichero requirements.txt)


Aplicación: web.py:

```python

# file web.py

from flask import Flask, render_template, request

app = Flask(__name__)

@app.route("/", methods=['GET'])
def saluda():
    return render_template('index.html', msg="Hola Mundo!")

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080, debug=True)


```

Vista: templates/index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Web App</title>
    <link rel= "stylesheet" type= "text/css" href= "{{ url_for('static',filename='css/main.css') }}" />
</head>
<body>
    <h1> {{ msg }}</h1>
    <img src="{{ url_for('static',filename='img/logo.png') }}" />
</body>
</html>
```

Estilo: css/main.css

```css
h1 {
    color: red;
}
```

Imagen: img/logo.png


Prueba la aplicación en local


```shell
$ python3 web.py
```



## Contruir la imagen

Crea el fichero Dockerfile

```dockerfile

FROM python:3.10-alpine

WORKDIR /app

EXPOSE 8080

COPY ./app .

RUN pip install --no-cache-dir -r requirements.txt

CMD [ "python", "./web.py" ]

```

Construye la imagem

```shell
$ docker build --tag jluisalvarez/hm-flask:2024 .
```

Prueba la imagen en local

```shell
$ docker run -d --name hmflask -p 8080:8080 jluisalvarez/hm-flask:2024
```

Sube la imagen a Docker Hub

```shell
$ docker login

...

$ docker push jluisalvarez/hm-flask:2024

```

# Despliega en AWS ECS

En la consola web de AWS, accede a Amazon Elastic Container Service.

## Crea un cluster

Un clúster en ECS es un concepto lógico que permite la agrupación de contenedores que se ejecutarán en máquinas virtuales específicas (EC2) o en infraestructura Serveless (Fargate). Para crearlo:

En la consola Web AWS, accede a Amazon Elastic Container Service.

Selecciona la opción "Clusters" y clic en "Crear Cluster"

Introduce un nombre para el clúster, por ejemplo: clusterECSFargate

En Infraestructura, selecciona la opción AWS Fargate (que estará seleccionada por defecto)

En Monitoreo y Etiquetas, lo dejaremos por defecto.


## Crea una definición de tarea

En la consola Web AWS, en Amazon Elastic Container Service.

Selecciona la opción "Definiciones de Tareas" y clic en "Crear una nueva definición de Tarea"

En el primer paso, establecemos un nombre, por ejemplo, hm-flask. 
En "Requisitos de infraestructura", establecemos la configuración del entorno, AWS Fargate, Linux, 0.5 CPU, Memoria 1Gb y en rol de tarea y de ejecución, seleccionamos LabRole, dejando el resto de opciones por defecto.

En contenedor 1, estableceremos un nombre para el contenedor, por ejemplo hmflask, y la imagen para el contenedor (registry.hub.docker.com/jluisalvarez/hm-flask:2024).
Establecemos el puerto, en nuestro caso 8080, el resto de opciones la dejamos por defecto y  pulsamo en "Crear".

## Ejecutar una tarea o un servicio

En la consola Web AWS, accede a Amazon Elastic Container Service, opción "Clusters" y accedemos a nuestro cluster (clusterECSFargate)

Accedemos a la pestaña Tareas y pulsamos en Ejecutar una nueva tarea.

En Configuración Informática, seleccionamos Tipo de lanzamiento.

En Configuración de implementación, seleccionamos Tarea.

En Familia, seleccionamos la definición de tarea a ejecutar y la versión deseada.

En Redes, seleccionamos la VPC y el grupo de seguridad, activando la opción de IP pública.

El resto de opciones pueden dejarse por defecto.

Una vez esté activa, podemos acceder a la tarea y en "Enlace de Red" podemos ver la URL de acceso.