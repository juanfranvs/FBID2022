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
        tar -xzf kafka_2.13-3.3.1.tgz   
- **Kafka**
  Instlación de la  versón kafka_2.12-3.0.0 a través de la URL: https://kafka.apache.org/quickstart
  
        cd /practica_big_data_2019/software 
        tar -xzf kafka_2.13-3.3.1.tgz
        
