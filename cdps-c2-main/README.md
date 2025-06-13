<img  align="left" width="150" style="float: left;" src="https://www.upm.es/sfs/Rectorado/Gabinete%20del%20Rector/Logos/UPM/CEI/LOGOTIPO%20leyenda%20color%20JPG%20p.png">
<img  align="right" width="60" style="float: right;" src="https://www.dit.upm.es/images/dit08.gif">


<br/><br/>

# Solución de la Práctica creativa 2 de CDPS

## 1. Despliegue de la aplicación en máquina virtual pesada  (2 puntos)

En el ordenador host relazar las siguiente operaciones:

1. Clonar el repositorio

```
git clone https://github.com/CDPS-ETSIT/practica_creativa2.git
```

```
cd ./practica_creativa2/bookinfo/src/productpage
```

2. Exportar la variable de entorno:

```
export GROUP_NUMBER=49
``` 
Comprobar que el valor se ha almacenado correctamente

```
echo ${GROUP_NUMBER}
```

3. Crear el entorno virtual en pyhton

```
python3 -m venv cdps-c2-env

```
Activar el entorno
```
source cdps-c2-env/bin/activate

```

4. Instalar las dependencias con pip3

```
pip3 install -r requirements.txt
```

5. Levantar el servidor indicando el puerto

```
python3 productpage_monolith.py 9080
```

**Modificaciones:** Los ficheros modificados para esta parte de la práctica son `productpage_monolith.py` añadiendo:

```
línea 50: groupNumber = "N/D" if (os.environ.get("GROUP_NUMBER") is None) else os.environ.get("GROUP_NUMBER")
línea 304: groupNumber=groupNumber,

```

y el fichero `templates/productpage.html` añadiendo:

```
línea 98:  {{groupNumber}} dentro de la etiqueta h3 
```

## Despliegue de una aplicación monolítica usando docker (2 puntos).

1. Construir la imagen a partir del fichero Dockerfile ubicado en la ruta `bookinfo/src/productpage/`:

```
 sudo docker build -t g49/product-page -f Dockerfile-monolith .
```

2. Ejecutar el contenedor con la imagen creada indicando la variable de entorno:

```
docker run --name g49-product-page -p 9080:9080 -e GROUP_NUMBER=49 -d g49/product-page
```

## Segmentación de una aplicación monolítica en microservicios utilizando docker-compose ( 2 puntos)

1. Ejecutar el ficher `build-services.sh` para construir todas las imagenes a partir de sus respectivos Dockerfiles:

```
cd sol_practica_creativa2/bookinfo/src
./build-services.sh
```

2. Levantar todos los contenedores a partir del fichero docker-compose:

```
docker-compose up
```

**Nota:** En esta solución no se ha considerado el formato de nombres que se les ha pedido a los estudantes. Sin embargo esos cambios son mínimos a efectos de la práctica.

## Despliegue de una aplicación basada en microservicios utilizando Kubernetes (4 puntos, opcional) 

1. Crear el cluster de Kubernetes, dependiendo del sistema a utilizar (KGE, minikube,etc).
2. Ejecutar el fichero `bookinfo.yaml` donde se encuentran todos los comandos para levantar los pods de cada uno de los microservicios:
```
cd sol_practica_creativa2/bookinfo/platform/kube/
```

```
kubectl apply -f bookinfo.yaml
```

3. Verificar los pods y servicios levantados y que se expone el servicio de productpage:

```
kubectl get pods
``` 

```
kubectl get svc
``` 

```
kubectl get deployments
```

4. Verificar que se puede puede cambiar las versiones de los ratings, modificando el fichero `bookinfo.yaml` en la sección de reviews y aplicando los cambios:

Seccion de Reviews con versión 2:

```
apiVersion: v1
kind: Service
metadata:
  name: reviews
  labels:
    app: reviews
    service: reviews
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: reviews
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v1
  labels:
    app: reviews
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v1
  template:
    metadata:
      labels:
        app: reviews
        version: v1
    spec:
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v2:1.16.2
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
        securityContext:
          runAsUser: 1000
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
```


Seccion de Reviews cambiada a versión 3:

```
apiVersion: v1
kind: Service
metadata:
  name: reviews
  labels:
    app: reviews
    service: reviews
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: reviews
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v3
  labels:
    app: reviews
    version: v3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v3
  template:
    metadata:
      labels:
        app: reviews
        version: v3
    spec:
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v3:1.16.2
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
        securityContext:
          runAsUser: 1000
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
```

```
kubectl apply -f bookinfo.yaml
```
Ahora las reviews deben aparecer con estrellas rojas


