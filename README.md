[Capside](http://twitter.com/capside): Usando Kinesis en el monte de las ánimas
========================================================

## Diapositivas 
* Aquí están las diapositivas de la charla [diapos](http://blabla.com/diaposyeso)

## Creación de una instancia EC2 para ejecutar el productor y el consumidor.
* La instancia ha de estar en una subnet con acceso a Internet y tener una IP pública para poder alcanzarla
* Se debe proteger con un security group que permita acceso por ssh y por http al puerto 8080 desde al menos la IP desde la que os encontréis
* Debe asumir un rol que le permita acceder a: Kinesis, CloudWatch, DynamoDB y S3
* Usaremos la imagen Amazon Linux AMI y una instancia de tipo m4.large
* En el script de user data eliminaremos la versión 1.7 de Java e instalaremos la 1.8, podemos usar el script de debajo
 ```
 #!/bin/bash
 yum install -y jq git
 yum remove -y java-1.7.0-openjdk
 yum install -y java-1.8.0
 ``` 
 
 ## Creación de un "Kinesis Stream" a través de la CLI de AWS (desde la instancia lanzada arriba)

Configuramos la CLI de AWS en la instancia
```bash
aws configure ##Solo hace falta configurar la region, usar la misma región que aquella donde se haya desplegado la instancia
```
Usamos la CLI para crear nuestro stream
```bash
aws kinesis create-stream --stream-name animas --shard-count 2
aws kinesis describe-stream --stream-name animas
aws kinesis describe-stream --stream-name animas --query StreamDescription.StreamStatus
aws kinesis get-shard-iterator --stream-name animas --shard-id shardId-000000000000 --shard-iterator-type TRIM_HORIZON --query ShardIterator
aws kinesis get-records --shard-iterator "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

## Descargar los binarios de productor y consumidor a la instancia
```bash
wget https://github.com/capside/aws-kinesis-animas/raw/master/AnimasProducer.jar
wget https://github.com/capside/aws-kinesis-animas/raw/master/AnimasConsumer.jar
```

## Ejecutar el productor
El productor recibe varios parámetros, entre ellos, el nombre del stream y la región donde éste se encuentra.
Además, le daremos la latitud y longitud donde lo queremos desplegar (monte de las ánimas; Lat:41.754994, Lon:-2.449176)
```
java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=1044 -jar AnimasProducer.jar --drone=5555 --stream=animas --region=us-east-2 --latitude=41.754994 --longitude=-2.449176
```

## Ejecutar el consumidor
```
java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=1045 -jar AnimasConsumer.jar --stream=animas --region=us-east-2
```

## Observar las ánimas
Ahora podemos conectarnos por http al puerto 8080 de la instancia en la que se ejecuta el consumidor, observaremos la aplicación web en la que aparecen todas las ánimas detectadas por nuestros drones

![ÁnimasEnPena](https://github.com/capside/aws-kinesis-animas/blob/master/MonteAnimas.gif)

## Limpieza final
Antes de eliminar la instancia, podemos usarla para borrar el stream de Kinesis así como la tabla de DynamoDB creada por el consumidor.
```
aws dynamodb delete-table --table-name Zombies
aws kinesis delete-stream --stream-name animas
```
Ahora podemos eliminar la instancia
