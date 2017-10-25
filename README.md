Usando Kinesis en el monte de las ánimas
========================================================

## Somos [Capside](http://twitter.com/capside)

## Diapositivas 
* Aquí están las diapositivas de la charla [diapos](http://blabla.com/diaposyeso)

## Creación de una instancia EC2 para ejecutar el productor y el consumidor.
* La instancia ha de estar en una subnet con acceso a Internet y tener una IP pública para poder alcanzarla
* Se debe proteger con un security group que permita acceso por ssh y por http al puerto 8080 desde al menos la IP desde la que os encontréis
* Debe asumir un rol que le permita acceder a: Kinesis, CloudWatch, DynamoDB y S3
* Usaremos la imagen Amazon Linux AMI y una instancia de tipo m4.large
* En el script de user data eliminaremos la versión 1.8 de Java e instalaremos la 1.9, podemos usar el script de debajo
 ```
 #!/bin/bash
 yum install -y jq git
 yum remove -y java-1.7.0-openjdk
 yum install -y java-1.8.0
 ``` 
 
 ## Creación de un "Kinesis Stream" a través de la CLI de AWS (desde la instancia lanzada arriba)

```bash
aws kinesis create-stream --stream-name zombies --shard-count 2
aws kinesis describe-stream --stream-name zombies
aws kinesis describe-stream --stream-name zombies --query StreamDescription.StreamStatus
aws kinesis get-shard-iterator --stream-name zombies --shard-id shardId-000000000000 --shard-iterator-type TRIM_HORIZON --query ShardIterator
aws kinesis get-records --shard-iterator "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```
