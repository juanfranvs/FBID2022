# FBID2022
Proyecto Realizado para la asignatura BDFI/FBID del MUIT/MUIRST de la ETSIT-UPM en el curso 2022/2023 por Juan Francisco Vara Sánchez y Jose Javier Mata de la Fuente.
Para el desarrollo de este proyecto tomamos como base el repositorio proporcionado: https://github.com/ging/practica_big_data_2019.

Realizaremos un despliegue en local partiendo y otro sobre Google Cloud utilizando la tecnología Docker para virtualizar los servicios en ficheros Dockerfile y lanzarlos con un fichero Docker Compose.

# Despliegue en Local
Primeramente descargamos el repositorio y accedemos a él:

    git clone https://github.com/ging/practica_big_data_2019.
    
Y nos movemos al repositorio:

    cd practica_big_data_2019

# Instalación de SW
Antes de realizar este despliegue nos centramos en instalar todas las herramientas software necesarias y que procedemos a detallar:
Por simplicidad, vamos a generar una carpeta software dentro del proyecto donde iremos colocando de forma ordenada el software necesario

- **Intellij**
Visitamos la página de JetBrains y nos descargamos el código asociado a la versión Linux que necesitamos https://www.jetbrains.com/idea/download/#section=linux
- **Python3**
 
        sudo apt install python 3.7    
- **PIP**

        sudo apt-get install python3-pip
- **SBT**

        sudo apt install sbt
- **MongoDB**

        wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
        sudo apt-get install gnupg
        echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
        sudo apt-get update
        sudo apt-get install -y mongodb-org
        echo "mongodb-org hold" | sudo dpkg --set-selections
        echo "mongodb-org-database hold" | sudo dpkg --set-selections
        echo "mongodb-org-server hold" | sudo dpkg --set-selections
        echo "mongodb-mongosh hold" | sudo dpkg --set-selections
        echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
        echo "mongodb-org-tools hold" | sudo dpkg --set-selections
Para arrancar el servicio de mongo y acceder a la shell de Mongo ejecutamos:
        sudo systemctl start mongod
        mongosh
- **Spark y Scala**
Descargamos el paquete https://mirrors.estointernet.in/apache/spark/spark-3.1.2/ (Versión 3.1.2 de Spark que incluye Hadoop 3.2)

        cd /practica_big_data_2019/software
        spark-3.1.2-bin-hadoop3.2.tgz
        tar -xzf spark-3.1.2-bin-hadoop3.2.tgz   
- **Kafka**
  Instlación de la  versón kafka_2.12-3.0.0 a través de la URL: https://kafka.apache.org/quickstart
  
        cd /practica_big_data_2019/software 
        kafka_2.12-3.0.0.tgz
        tar -xzf kafka_2.12-3.0.0.tgz
        
## Downloading Data
Nos movemos al **directorio de la práctica**

        cd practica_big_data_2019
Descargamos los datos ejecutando:

        ./resources/download_data.sh
## Install Python libraries

        pip install -r requirements.txt
## Start Zookeeper
Arrancamos en un terminal el servidor de Zookeeper:

        bin/zookeeper-server-start.sh config/zookeeper.properties
## Start Kafka
Arrancamos en un terminal el servidor de Kafka:

        bin/kafka-server-start.sh config/server.properties
Creamos en un nuevo terminal un **Tópico**

        bin/kafka-topics.sh \
            --create \
            --bootstrap-server localhost:9092 \
            --replication-factor 1 \
            --partitions 1 \
            --topic flight_delay_classification_request
Nos devuelve el siguiente mensaje:

         Created topic "flight_delay_classification_request".
Una vez creado mostramos la **Lista de Tópicos**:

         flight_delay_classification_request
Opcionalmente podemos generar un **Consumidor**:

         bin/kafka-console-consumer.sh \
            --bootstrap-server localhost:9092 \
            --topic flight_delay_classification_request \
            --from-beginning
## Import the distance records to MongoDB
Comprobamos si la máquina está activa ejecutando:

         sudo service mongod status
Nos devuelve:

     mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor pr>
     Active: active (running) since Tue 2022-11-15 16:53:03 CET; 23h ago
     Docs: https://docs.mongodb.org/manual
     Main PID: 18011 (mongod)
     Memory: 50.4M
     CGroup: /system.slice/mongod.service
            └─18011 /usr/bin/mongod --config /etc/mongod.conf
            
Ejecutamos el script import_distances.sh

          ./resources/import_distances.sh
Nos devuelve:

     2019-10-01T17:06:46.957+0200	connected to: mongodb://localhost/
     2019-10-01T17:06:47.035+0200	4696 document(s) imported successfully. 0 document(s) failed to import.
     MongoDB shell version v4.2.0
     connecting to: mongodb://127.0.0.1:27017/agile_data_science?compressors=disabled&gssapiServiceName=mongodb
     Implicit session: session { "id" : UUID("9bda4bb6-5727-4e91-8855-71db2b818232") }
     MongoDB server version: 4.2.0
        {
	        "createdCollectionAutomatically" : false,
	        "numIndexesBefore" : 1,
	        "numIndexesAfter" : 2,
	        "ok" : 1
        }

   
            
## Train and Save the model with Pyspark mllib
Dentro del directorio de la práctica exportamos las variables de entorno:

          export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
          export SPARK_HOME=/opt/spark
Entrenamos el modelo dentro de practica_big_data_2019/resources:

          python3 resources/train_spark_mllib_model.py .
Vemos el modelo generado moviendonos al directorio models
        
          ls ../models
## Run Flight Predictor
Añadimos la siguiente sentencia dentro del fichero MakePrediction.scala:

          cd /flight_prediction/src/main/scala/es/upm/dit/ging/predictor
          val base_path="/home/upm/practica_big_data_2019"
Una vez realizado el cambio abrimos Intellij compilamos y ejecutamos el fichero MakePrediction

	  cd /opt/idea-IC-222.4345.14/bin 
	  ./idea.sh

## Start the prediction request Web Application
Añadimos la variable de entorno PROJECT_HOME que contendrá la ruta de la práctica:

          export PROJECT_HOME="/home/upm/practica_big_data_2019
Vamos a la ruta web del proyecto y ejecutamos el fichero predict_flask.py:

          cd practica_big_data_2019/resources/web
          python3 predict_flask.py
Este fichero nos permitirá abrir la web predictora de vuelos que arranca la aplicación sobre el puerto 5000 de localhost: http://localhost:5000/flights/delays/predict_kafka y ejecutar el botón Submit

## Check the predictions records inserted in MongoDB
Una vez realizamos la predicción con el interfaz web abrimos la shell de MongoDB para ver que los datos realizados por la predicción son almacenados de forma correcta.

           > mongosh
           > use agile_data_science
           > db.flight_delay_classification_response.find()
Esto nos devuelve como respuesta una query de Mongo que contiene los datos que se han almacenado de dicha predicción.

             {
                _id: ObjectId("6373cc430e39684b17440328"),
                Origin: 'SFO',
                DayOfWeek: 1,
                DayOfYear: 359,
                DayOfMonth: 25,
                Dest: 'ATL',
                DepDelay: 5,
                Timestamp: ISODate("2022-11-15T17:28:35.113Z"),
                FlightDate: ISODate("2018-12-24T23:00:00.000Z"),
                Carrier: 'AA',
                UUID: 'efab3289-49ce-4233-9c47-4983fb3acc91',
                Distance: 2139,
                Route: 'SFO-ATL',
                Prediction: 2
             }

## Ejecución con Spark-Submit
Una vez que hemos probado con Intellij la predicción, paramos el código de MakePrediction y procedemos a generar un fichero con extensión .jar que se encargará de compilar y ejecutar el código empleando el comando **spark-submit**.
Vamos a la ruta /flight_prediction/target/scala y ejecutamos:

            sbt clean
            sbt package
Con este fichero ejecutamos el siguiente comando:

            spark-submit --class es.upm.dit.ging.predictor.MakePrediction --master local --packages org.mongodb.spark:mongo-spark-connector_2.12:3.0.0,org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.1 /home/upm/practica_big_data_2019/flight_prediction/target/scala-2.12/flight_prediction_2.12-0.1.jar
Con ello, podremos volver a generar otra predicción que también se almacenará en la BBDD de MongoDB sin necesidad de utilizar Intellij.

## Apache Airflow(Opcional)
Con este SW de Apache podremos entrenar el modelo de otra manera. 
The version of Apache Airflow used is the 2.1.4 and it is installed with pip. For development it uses SQLite as database but it is not recommended for production. For the laboratory SQLite is sufficient.
Install python libraries for Apache Airflow (suggested Python 3.7)

	cd resources/airflow
	python3.7 -m pip install -r requirements.txt -c constraints.txt
Seleccionamos la variable de entorno PROJECT_HOME cogiendo la ruta del repositorio de la práctica
	
	export PROJECT_HOME=/home/upm/practica_big_data_2019
Establecemos un nuevo valor de la variable PATH:
	
	export PATH=$PATH:~/.local/bin
En un terminal, configuramos el environment de airflow:

	export AIRFLOW_HOME=~/airflow
	mkdir $AIRFLOW_HOME/dags
	mkdir $AIRFLOW_HOME/logs
	mkdir $AIRFLOW_HOME/plugins

	airflow users create \
    	--username admin \
    	--password admin \
    	--firstname Jack \
    	--lastname  Sparrow\
    	--role Admin \
    	--email example@mail.org
Iniciamos la base de datos:

	airflow db init
Iniciamos en otro terminal el scheduler y el webserver de airflow:

	airflow webserver --port 8080
	airflow scheduler
Nos conectamos a la página  http://localhost:8080/home y nos muestra la lista de DAGs accesibles
Si pinchamos sobre la solapa de **Play->Trigger** Dag nos entrena un modelo que aparece representado en la lista de DAGs **Run**

	
	
	

            
        
        
