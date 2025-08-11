# Container for ML task

## Docker

### Launch and Operate ML Dev Environments with Docker

#### MLFlow

1. `docker image ls` to show the current image installed
2.  Use `docker pull io/library/tool:version` to pull the image you want
3. When you run some packages, you want to have access the UI, therefore you need to open the port. `docker run -p hostport:containerport <image_name> <The command you want to run inside the container> ` An example: `docker run -p 5555:5000 ghcr.io/mlflow/mlflow:v3.2.0 mlflow server --host 0.0.0.0`
4. By default, if you run the above command, it would go through the following procedures:  pull -> create -> attach -> connect -> start. The only want you can get out to the bash is to ctrl + c, but this would stop the container. 
5. `docker ps` to show the current running container.  `docker ps -l ` `docker ps -n 4` can show the previously run container. `docker ps -a`  to show all the containers. 
6. `docker start xxx` can start a stopped container.  When you  start the docker, you might want to check its history by using `docker logs xxx`. You might even want to go inside using `docker exec -it xxx sh`. And you can run something like `ps aux`. 
7.  But you do not have to run the container always in attached mode!  `docker run -dit alpine sh` here the d stands for detach model. the it stands for interactive shell.  You can enter `docker exec -it XXX sh` to interact with it. 
8. `docker rm -f xxx` to stop and remove container.  `docker rmi xxx` to remove image.
9. So to formally run mlflow in the backend, use `docker run -d -p 5555:5000 --name mlflow ghcr.io/mlflow/mlflow:v3.2.0 mlflow server --host 0.0.0.0`. 

#### Jupyter notebook

1. The difference this time is it needs to have a mounted place so that even if the container is deleted, something still remains. `docker run -d -p 8888:8888 --name notebook -v ~/ml-docker/notebooks:/home/jovyan/work jupyter/scipy-notebook` What does this mean? -d background. ip the port exports. -name defines the name -v is the volume it mounts   localhost:container  Finally, this is from dockerhub.io, and tag is latest. The complete version should be `docker.io/jupyter/scipy-notebook:latest` 
2. Use `docker logs notebook` to find the token. A bit of tricky I guess. 

#### Connect notebook with MLFlow Container

1. We want both of them to be running, and we want to run the code in jupyter container. The MLflow exports 5000 to the local host 5555 port. We need the jupyter container to connect to that port.  This can be done using `mlflow.set_tracking_uri("http://host.docker.internal:5555")`. 
2. After running the experiment in jupyter notebook, you can easily check the localhost:5555, to see the actual result of the experiment.
3. Now stop by using `docker stop xxx ` Delete by using `docker rm xxx` or simply use `docker container prune`  

### Packaging ML Apps as Container Images with Dockerfiles

You can create a container based on prebuilt image, copy the source code and install the command manually. You can also use dockerfile to do all these automatically.

#### Manually 

1. We pick the python311 inside the debian as the base image. `docker run -idt --name dev -p  7860:7860 python:3.11-slim bash`  The reason to use 7860 is gradio would be using that port to gave a web interface. 
2. Let us copy the source code directly inside the docker. `docker cp . dev:/app`
3. Go inside the container and do something about it: `docker exec -it dev bash`  . Inside it find the requirement.txt and install it.
4. We can convert this commit and transform it into an image, first ctrl + c to stop the app and ctrl + d to exit out of the container.  Use this command. `docker container commit dev docker.io/xxxxxx/tech-stack-advisor:v1` Here xxx is the user name of your docker hub. 
5. If you want to publish it, you can use the following method: `docker login`  and `docker image push docker.io/lzabry1/tech-stack-advisor:V1`

#### Dockerfile

> 1. FROM python:3.11-slim
> 2.  
> 3. WORKDIR /app
> 4.  
> 5. COPY requirements.txt .
> 6. RUN pip install --no-cache-dir -r requirements.txt
> 7.  
> 8. COPY . .
> 9.  
> 10. EXPOSE 7860
> 11.  
> 12. CMD ["python", "app.py"]

1. `cat > Dockerfile` and copy all the things above. 
2. `docker build -t docker.io/xxxxxx/tech-stack-advisor:v2 .`   The same as earlier. 
3. `docker run --name tsa -p 7860:7860 docker.io/xxxxxx/tech-stack-advisor:v2` This would start the container under the name of tsa.
4. Remember to tag the image to the latest:  `docker tag docker.io/xxxxxx/tech-stack-advisor:v2 docker.io/xxxxxx/tech-stack-advisor:latest` 

#### Difference

1. Dockerfile would create a smaller sized container
2. It can maintain some commands and more layers. 

#### The writing of dockerfile

1. `FROM ubuntu:latest` Build the basic image
2. `RUN pip install --no-cache-dir -r requirements.txt` defines what do you want to do during the build process
3. `WORKDIR /app` All the subsequent commands happen inside
4. `COPY . .` Happens while the build happens. 
5. `EXPOSE` defines the port . Other user can use `docker image history xxx` to find the exported port
6. `CMD` You want to run something during the running stage. You can only have ONE command. 

### Simulating Production Grade ML Systems in Dev with Docker Compose

#### Task

1. We need to package the prediction model using fastapi, which is going to be called by streamlit.
2. We also need MLflow so that we can train and make experiments on the model. 

#### Docker Compose

1.  If we want to manually run the container, we can use `docker run -d --name mlflow -p 5555:5000 \
     ghcr.io/mlflow/mlflow:latest \
     mlflow server --host 0.0.0.0 \`

2. Of course this might be too complex. Why no just use code? That is the idea of Docker Compose. 

3. Go inside  the mlflow directory, write the compose file as following: 

   ```
   services:
     mlflow:
       image: ghcr.io/mlflow/mlflow:latest
       ports:
         - 5555:5000
       command: mlflow server --host 0.0.0.0
     
   ```

   4.  `docker compose ps` `docker compose up -d`  `docker compose down` `docker compose up` `docker compose down`

   5.  Right now the project looks like this

      ```
      .
      ├── Dockerfile
      ├── LICENSE
      ├── README.md
      ├── configs
      │   └── model_config.yaml
      ├── data
      │   ├── processed
      │   │   └── README.md
      │   └── raw
      │       └── house_data.csv
      ├── mlflow
      │   └── compose.yaml
      ├── models
      │   └── trained
      │       └── README.md
      ├── notebooks
      │   ├── 00_data_engineering.ipynb
      │   ├── 01_exploratory_data_analysis.ipynb
      │   ├── 02_feature_engineering.ipynb
      │   └── 03_experimentation.ipynb
      ├── requirements.txt
      ├── run_pipeline.sh
      ├── src
      │   ├── api
      │   │   ├── README.md
      │   │   ├── inference.py
      │   │   ├── main.py
      │   │   ├── requirements.txt
      │   │   ├── schemas.py
      │   │   └── utils.py
      │   ├── data
      │   │   └── run_processing.py
      │   ├── features
      │   │   └── engineer.py
      │   └── models
      │       └── train_model.py
      └── streamlit_app
          ├── Dockerfile
          ├── README.md
          ├── app.py
          └── requirements.txt
      ```

      

6.  We can see that mlflow use compose, streamlit and the main project(fast api under root) use dockerfile. Let's start working on it. 

   ```
   #compose.yaml
   services:
     fastapi:
       image: docker.io/xxxxxx/fastapi:dev
       build:
         context: "./"
         dockerfile: "Dockerfile"
       ports:
         - "8000:8000"
    
     streamlit:
       image: docker.io/xxxxxx/streamlit:dev
       build:
         context: "streamlit_app/"
         dockerfile: "Dockerfile"
       ports:
         - "8501:8501"
       environment:
         API_URL: "http://fastapi:8000"
         
   # Dockerfile
   FROM python:3.11-slim
   
   WORKDIR /app
   
   COPY src/api/ .
   
   RUN pip install -r requirements.txt
   
   COPY models/trained/*.pkl models/trained/
   
   EXPOSE 8000
   
   CMD [ "uvicorn",  "main:app",  "--host",  "0.0.0.0",  "--port",  "8000" ]
   
   #streamlit_app Dockerfile
   FROM python:3.9-slim
   
   WORKDIR /app
   
   COPY  app.py requirements.txt .
   
   RUN pip install -r requirements.txt
   
   EXPOSE 8501
   
   CMD  [ "streamlit", "run", "app.py", "--server.address=0.0.0.0" ]
   ```

   7. `docker compose build` to prepare all the images
   8.  So, different containers within different network, you will have to connect using local host ports (localhost for container means container itself!!!!!). But, within the same containers, it is perfectly fine to just use the inside port. If it is in the same network, there is dns so you can directly call other container
   9. `docker compose up -d` can modify the running container. It is very beautiful...

