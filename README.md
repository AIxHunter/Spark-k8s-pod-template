## File used:
- Dockerfile: Used to build the Docker image for Spark.
- entrypoint.sh: Used as the entry point for the Docker container.
- app.jar: The Spark application JAR file.
- pod.yaml: Used to submit a Spark job to Kubernetes.

## Description:
- This project is based on Apache Spark and Kubernetes.
- It uses Spark version 3.3.2.
- The Dockerfile and entrypoint.sh are sourced from the Apache Spark Docker repository: https://github.com/apache/spark-docker, use the equivalent Dockerfile and entrypoint.sh if you want to use a different Spark version.

## Steps to deploy:
The used namespace is ```default``` namespace for the below commands.

1. **Build the Docker image**:
```console
docker build -t docker-spark:0.0.0.1 .
```

2. **Push the Docker image to a Docker registry**:
```console
docker push docker-spark:0.0.0.1
```

3. **Create a service account and give it the edit role on the cluster**:
```console
kubectl create serviceaccount sa-spark
kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:sa-spark --namespace=default
```

4. **Get the Kubernetes master URL**:
```console
kubectl cluster-info
```

Replace https with k8s and replace the ```--master``` parameter with the new master url in the below command or in the pod.yaml file.


5. **Submit the Spark job to Kubernetes (2 methods)**:


    **- Run using a pod template**:
    ```console
    kubectl apply -f pod.yaml
    ```
    
    **- Run using spark submit**:
    ```console
    ./bin/spark-submit --master k8s://192.168.49.2:8443 --deploy-mode cluster --name spark-docker --class org.example.Main --conf spark.executor.instances=2 --conf spark.kubernetes.container.image=docker-spark:0.0.0.1 --conf spark.kubernetes.authenticate.driver.serviceAccountName=sa-spark --conf spark.kubernetes.namespace=default --conf spark.kubernetes.file.upload.path=/tmp/ "local:///opt/spark/work-dir/app.jar"
    ```

