

# PROYECTO FINAL - PAOLO SCOTTO DI MASE

## Software necesario

- [docker](https://docs.docker.com/engine/install/)
- [minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [helm](https://helm.sh/docs/intro/install/)
- [kubeseal](https://github.com/bitnami-labs/sealed-secrets#kubeseal)

En este proyecto he desarrollado una aplicación y he configurado 2 repositorios en GitHub para alojar diferentes aspectos del proyecto. El primer repositorio, llamado ***kcfp-app-argocd-src***, contiene el código fuente de la aplicación. Aquí se encuentran todos los archivos y componentes relacionados con la lógica de la aplicación.
El segundo repositorio, llamado ***kcfp-argocd-app***, almacena los manifiestos de Kubernetes, incluyendo tanto la definición de la aplicación como los sealed secrets cifrados. Utilizo Helm para empaquetar estos manifiestos. Este repositorio contiene los archivos YAML que describen cómo se deben desplegar y configurar los recursos de Kubernetes para mi aplicación, así como los secretos cifrados que se utilizan en ella. La configuración de "sealed secrets" nos permite mantener la seguridad de los secretos sensibles, ya que se almacenan cifrados y solo se pueden descifrar dentro del clúster. Esto garantiza que los secretos no estén expuestos en texto plano en el repositorio. En cuanto a Continuous Integration (CI) he utilizado GitHub Actions para realizar acciones automatizadas cada vez que se realizan cambios en el repositorio de código con flujos de trabajo (lint, test y release) para realizar tareas como verificar la calidad del código, ejecutar pruebas y lanzar nuevas versiones de la aplicación. He aprovechado ***semantic release*** para generar automáticamente versiones basadas en los cambios en el repositorio y generar notas de lanzamiento. En cuanto a Continuous Deployment (CD), he utilizado ArgoCD con la metodología GitOps para gestionar y desplegar mi aplicación de manera automatizada. GitOps es una metodología para gestionar y operar aplicaciones en entornos de Kubernetes utilizando Git como fuente única de verdad. En el contexto de GitOps, todos los recursos de la aplicación, como configuraciones, definiciones de despliegue y políticas, se almacenan y versionan en un repositorio Git. La idea principal es que cualquier cambio en el estado deseado de la aplicación se refleje automáticamente en el clúster de Kubernetes a través de un proceso de reconciliación continua. En GitOps, el ciclo de vida de una aplicación se gestiona a través de la utilización de herramientas como ArgoCD, FluxCD o Jenkins. Estas herramientas se encargan de comparar el estado actual del clúster con la definición almacenada en el repositorio Git y, si hay diferencias, realizan las acciones necesarias para alcanzar el estado deseado. Esto puede incluir desplegar nuevas versiones de la aplicación, realizar actualizaciones, revertir cambios y gestionar secretos. Al elegir ArgoCD utilizo tambien el repositorio ***gitops-con-argocd*** que contiene archivos YAML con configuraciones específicas para desplegar el chart predefinido de ArgoCD en GKE. Este repositorio no se carga en Git, sino que se utiliza localmente. Desplegar ArgoCD y la aplicación en GKE de GCP proporciona beneficios como una gestión simplificada de Kubernetes, alta disponibilidad y escalabilidad, integración con servicios de Google Cloud, seguridad y cumplimiento normativo, así como una estrecha integración entre ArgoCD y el entorno de GKE. Estas ventajas combinadas ayudan a mejorar la eficiencia, la confiabilidad y la seguridad de la aplicación.
La aplicación se basa principalmente en el material abordado durante la asignatura de SRE. He integrado algunas de las aplicaciones que desarrollé durante las prácticas del bootcamp, agregando también nuevas funcionalidades y elementos. Es una aplicación web FastAPI que consta de 7 endpoints y se conecta a una base de datos de MongoDB.
Observamos la ejecución de los 7 tests, uno para cada endpoint, con el informe de cobertura de código:

    (venv) 20:22 @/Users/paoloscotto/desktop/kcfp-app-argocd-src/src ~ (git)-[main] $ pytest --cov
    =============================================================== test session starts ================================================================
    platform darwin -- Python 3.9.6, pytest-7.1.1, pluggy-1.0.0 -- /Users/paoloscotto/Desktop/kcfp-app-argocd-src/venv/bin/python3
    cachedir: .pytest_cache
    rootdir: /Users/paoloscotto/Desktop/kcfp-app-argocd-src/src, configfile: pytest.ini, testpaths: tests/
    plugins: asyncio-0.18.3, anyio-3.7.0, cov-3.0.0
    asyncio: mode=auto
    collected 7 items
    
    tests/app_test.py::TestFastAPIApp::read_health_test PASSED                                                                                   [ 14%]
    tests/app_test.py::TestFastAPIApp::read_main_test PASSED                                                                                     [ 28%]
    tests/app_test.py::TestFastAPIApp::analyze_text_file_test PASSED                                                                             [ 42%]
    tests/app_test.py::TestFastAPIApp::create_student_test PASSED                                                                                [ 57%]
    tests/app_test.py::TestFastAPIApp::joke_endpoint_test PASSED                                                                                 [ 71%]
    tests/app_test.py::TestFastAPIApp::update_student_test PASSED                                                                                [ 85%]
    tests/app_test.py::TestFastAPIApp::get_all_students_test PASSED                                                                              [100%]
    
    ---------- coverage: platform darwin, python 3.9.6-final-0 -----------
    Name                      Stmts   Miss Branch BrPart     Cover   Missing
    ------------------------------------------------------------------------
    application/__init__.py       0      0      0      0   100.00%
    application/app.py          147     18     30      7    85.88%   51, 56, 154-157, 163-198, 240, 247, 272, 276, 314, 333
    config/__init__.py            0      0      0      0   100.00%
    config/test_config.py         9      0      0      0   100.00%
    tests/__init__.py             0      0      0      0   100.00%
    tests/app_test.py            95      0      6      0   100.00%
    ------------------------------------------------------------------------
    TOTAL                       251     18     36      7    91.29%
    
    Required test coverage of 80.0% reached. Total coverage: 91.29%
    
    ================================================================ 7 passed in 0.99s =================================================================


<img width="1792" alt="Screenshot 2023-06-20 at 20 25 15" src="https://github.com/maciuozz/gitops-con-argocd/assets/118285718/f54c7a8d-8fc7-41bb-9d00-fa8fb2f2e50a">

La fase de Lint se activa en dos eventos, pull request (PR) y push. La fase de Test se activa cuando se completa exitosamente Lint y se utiliza para ejecutar pruebas unitarias en el código fuente. La fase release-build se activa después de que se completa una ejecución exitosa de Test. Automatiza el proceso de liberación y despliegue de software, generando automáticamente una nueva versión y publicando una imagen de Docker actualizada en dos registros diferentes: Docker Hub y GHCR. ***He usado repositorios privados en mi cuenta personal de GitHub en vez de usar un repositorios dentro de la organización keepcodingclouddevops7***. El fichero release.yaml usa 3 repository secret: GHCR_PAT, DOCKERHUB_TOKEN y DOCKERHUB_USERNAME. Para definir GHCR_PAT generamos un token en Settings --> Developer settings con estos permisos:

<img width="1789" alt="Screenshot 2023-06-20 at 23 54 50" src="https://github.com/maciuozz/gitops-con-argocd/assets/118285718/12d2b5a8-0179-46fd-8175-a00e3a1fc545">

Dentro del repositorio ***kcfp-app-argocd-src*** generamos un secret, GHCR_PAT, en Settings --> Secrets and variables --> Actions, con el valor del PAT creado anteriormente. Para definir DOCKERHUB_TOKEN generamos un token en Docker Hub y usamos su valor para crear el secret. Para definir DOCKERHUB_USERNAME simplemente creamos un secret con el valor del nombre de usuario de Docker Hub. A continuación se muestran los 3 secrets del repositorio ***kcfp-app-argocd-src***:

<img width="1224" alt="233875331-b1faa951-b2cb-40e8-a741-0ac7ccf365ff" src="https://github.com/maciuozz/gitops-con-argocd/assets/118285718/66ebabf9-1007-48f8-b3a3-0f32f45b35c2">

1. Para replicar el funcionamiento general es necesario descargar los 3 repositorios; despues creamos los repositorios ***kcfp-app-argocd-src*** y ***kcfp-argocd-app*** en Git y los
   clonamos. Abrimos 3 pestañas en la terminal y nos ubicamos en cada uno de los repositorios: ***kcfp-app-argocd-src, kcfp-argocd-app y gitops-con-
   argocd***, respectivamente. Copiamos el contenido desde los repositorios descargados hacia los repositorios clonados y hacemos commit de los cambios (el repositorio ***gitops-con-
   argocd*** solo lo descargamos para tenerlo en local). 
2. En Git, en el repositorio ***kcfp-argocd-app***, tenemos que configurar una ***Deploy key***. La deploy key sirve para dar acceso a ArgoCD al repositorio que contiene los
   manifiestos de la apliacion y los seales secrets. Los sealed secrets son secretos encriptados que se utilizan para proteger información sensible, como contraseñas o claves de API.
   Al utilizar una deploy key, se puede otorgar acceso al repositorio que contiene los sealed secrets a herramientas como Sealed Secrets Controller, permitiendo que esta herramienta
   automatizada desencripte y despliegue los secretos de forma segura.Para ello será necesario realizar los siguientes pasos:
   - Crear la deploy key abriendo una terminal y ejecutar el comando:
   
     ```sh
     ssh-keygen -t ed25519 -f $HOME/.ssh/argocd_app_kc
     ```
     > Pulsar el botón Enter en todos las preguntas requeridas

   - En el repositorio ***kcfp-argocd-app*** de Git vamos a Settings --> Deploy keys --> Add deploy key
   - Recuperar la clave pública de la deploy key creada anteriormente, ejecutando el siguiente comando:

        cat ~/.ssh/argocd_app_kc.pub
   - Copiar el contenido del comando anterior y utilizarlo para rellenar la sección Key en el proceso de añadir una nueva deploy key.

3. Recuperar la deploy key privada generada anteriormente:

   ```sh
   cat ~/.ssh/argocd_app_kc
   ```
4. Desde la pestaña ubicada en el repositorio ***gitops-con-argocd*** utilizamos el valor obtenido en el paso anterior para configurar el fichero argocd/values-secret.yaml tal y como    se muestra a continuación para completar la sección sshPrivateKey:

   ```yaml
    configs:
      credentialTemplates:
        test-app-creds:
          url: git@github.com:<github_username>/kcfp-argocd-app
          sshPrivateKey:
            <private_deploy_key>
    ```
   
5. Crear un cluster en GKE con la opción ***regional*** para que haya alta disponibilidad, tolerancia a fallos y mejor rendimiento. 
   Marcar ***Enable cluster autoscaler***. Hacer click en el nombre del cluster y luego en "CONNECT". Se abrirá una ventana donde copiamos
   el comando y lo ejecutamos en nuestra consola para conectarnos al cluster. Ejecutamos este comando para obtener permisos 
   de administrador del cluster:

       kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user $(gcloud config get-value account)
   
   Si echamos un vistazo al namespace ***kube-system*** vemos la siguiente salida: 
   
       16:53 @/Users/paoloscotto/desktop/gitops-con-argocd ~ $ kubectl get po -n kube-system
       NAME                                                             READY   STATUS    RESTARTS   AGE
       event-exporter-gke-755c4b4d97-d88sj                              2/2     Running   0          134m
       fluentbit-gke-dmhb8                                              2/2     Running   0          133m
       gke-metrics-agent-dt5z2                                          2/2     Running   0          133m
       konnectivity-agent-74fc8ddbc-bm6f8                               1/1     Running   0          132m
       kube-dns-5b5dfcd97b-dvtqn                                        4/4     Running   0          134m
       kube-dns-autoscaler-5f56f8997c-wxjh9                             1/1     Running   0          134m
       kube-proxy-gke-cluster-kcfp-ci-cd-a-default-pool-57bd9d51-52kw   1/1     Running   0          133m
       metrics-server-v0.5.2-67864775dc-dcmvk                           2/2     Running   0          132m
       pdcsi-node-f2psl                                                 2/2     Running   0          133m

   El Metric Server desempeña un papel fundamental en el funcionamiento del Horizontal Pod Autoscaler (HPA) y el stack de Prometheus.
   El HPA utiliza el Metric Server para obtener información sobre las métricas de los pods, como el uso de CPU y memoria. Basándose en estas métricas, el HPA ajusta automáticamente
   el número de réplicas de un conjunto de pods para escalar horizontalmente la aplicación y satisfacer la demanda actual.
   Por otro lado, el stack de Prometheus, que es una popular solución de monitoreo y alerta, también se integra con el Metric Server para obtener métricas del clúster de Kubernetes.
   El Metric Server actúa como una fuente de métricas confiable para Prometheus, que puede utilizar esas métricas para generar gráficos, alertas y realizar análisis en tiempo real.


         

6. Para instalar Nginx Ingress Controller en GCP-GKE ejecutar:

       kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.6.4/deploy/static/provider/cloud/deploy.yaml

7. Añadimos el repositorio de helm prometheus-community para poder desplegar el chart kube-prometheus-stack:

       helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
       helm repo update

8. Desde la pestaña ubicada en el repositorio ***kcfp-argocd-app*** desplegar el chart kube-prometheus-stack del repositorio de helm añadido en el paso anterior con los valores
   configurados en el archivo kube-prometheus-stack/values.yaml en el namespace fast-api:

       helm -n fast-api upgrade --install prometheus prometheus-community/kube-prometheus-stack -f kube-prometheus-stack/values.yaml --create-namespace --wait --version 34.1.1

9. Añadimos el repositorio helm de argocd:

       helm repo add argo https://argoproj.github.io/argo-helm
       helm repo update
   
10. Desde la pestaña ubicada en el repositorio ***gitops-con-argocd*** desplegar el helm chart de argocd utilizando el fichero `argocd/values.yaml` y el fichero `argocd/values-secret.yaml`:

    ```sh
    helm -n argocd upgrade --install argocd argo/argo-cd \
      -f argocd/values.yaml \
      -f argocd/values-secret.yaml \
      --create-namespace \
      --wait --version 5.34.1
    ``` 

11. Realizar un port-forward al servicio de argocd al puerto 8080 local:

       kubectl port-forward service/argocd-server -n argocd 8080:443

12. Obtener la contraseña de acceso a argocd mediante el siguiente comando:

        kubectl -n argocd get secret argocd-initial-admin-secret \
        -o jsonpath="{.data.password}" | base64 -d

13. Acceder a la url http://localhost:8080 y utilizar como credenciales el nombre de usuario admin y la contraseña obtenida en el paso anterior.

### Escenario 3: Aplicación con repositorio para código y otro para GitOps con Secrets con actualización automática

#### Configuración de sealed secrets y utilización en repositorio aplicación ArgoCD

1. Se desplegará la aplicación [sealed secrets](https://github.com/bitnami-labs/sealed-secrets), para ello se desplegará como una aplicación de ArgoCD añadiendola en la lista de `argocd-apps`, para ello utilizar el nuevo fichero `argocd-apps/values_v4.yaml` donde el cambio más relevante es el siguiente:

    ```yaml
   - name: sealed-secrets
     namespace: argocd
     finalizers:
     - resources-finalizer.argocd.argoproj.io
     project: default
     source:
       chart: sealed-secrets
       repoURL: https://bitnami-labs.github.io/sealed-secrets
       targetRevision: 2.9.0
       helm:
         releaseName: sealed-secrets
         parameters:
         - name: "secretName"
           value: "sealed-secrets-key"
     destination:
       server: https://kubernetes.default.svc
       namespace: secrets
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
       - CreateNamespace=true
     additionalAnnotations:
       argocd.argoproj.io/sync-wave: "0"
    ```

2. Pero antes es necesario realizar una serie de pasos para preconfigurar la clave de encriptación:

    1. Declarar las variables necesarias:

        ```sh
        export PRIVATEKEY="mytls.key"
        export PUBLICKEY="mytls.crt"
        export NAMESPACE="secrets"
        export SECRETNAME="sealed-secrets-key"
        ```

    2. Generar un nuevo par de claves RSA:

        ```sh
        openssl req -x509 -days 365 -nodes -newkey rsa:4096 -keyout "$PRIVATEKEY" -out "$PUBLICKEY" -subj "/CN=sealed-secret/O=sealed-secret"
        ```

    3. Crear el namespace donde se va a desplegar sealed-secrets:

        ```sh
        kubectl create namespace $NAMESPACE
        ```

    4. Crear un `Secret` de tipo tls, utilizando el par de claves RSA creado anteriormente:

        ```sh
        kubectl -n "$NAMESPACE" create secret tls \
          "$SECRETNAME" --cert="$PUBLICKEY" \
          --key="$PRIVATEKEY"
        kubectl -n "$NAMESPACE" label secret \
          "$SECRETNAME" \
          sealedsecrets.bitnami.com/sealed-secrets-key=active
        ```

3. Desplegar `argocd-apps` con la nueva versión utilizando el fichero `argocd-apps/values_v4.yaml`:

    ```sh
    helm -n argocd upgrade --install argocd-apps argo/argocd-apps \
      -f argocd-apps/values-sealed-secrets.yaml \
      --create-namespace --wait --version 1.1.0
    ```

4. Comprobar en ArgoCD, accediendo a la URL https://localhost:8080, como se dispone de una nueva aplicación llamada `sealed-secrets`, tal y  
   como se muestra en la siguiente captura:

    ![sealed secrets deployed](./img/sealed_secrets_deployed.png)

5. Crear un nuevo Secret utilizando kubeseal para acceder al repositorio creado en DockerHub, será necesario recuperar el token creado en el laboratorio anterior en la sección [Creación de cuenta en DockerHub y obtención de token](../1-pipelines-github-workflows-jenkins/README.md#creación-de-cuenta-en-dockerhub-y-obtención-de-token)

    ```sh
    kubectl create secret docker-registry registry-credential \
    --docker-server="https://index.docker.io/v1/" \
    --docker-username="<dockerhub_username>" \
    --docker-password="<dockerhub_token>" \
    --dry-run=client \
    -n fast-api \
    -o yaml > simple_secret.yaml

    cat simple_secret.yaml | kubeseal \
        --controller-namespace secrets \
        --controller-name sealed-secrets \
        --format yaml \
        -n fast-api \
        > sealed-secret.yaml

    rm simple_secret.yaml
    ```

6. Mover el fichero creado `sealed-secret.yaml` al repositorio de la aplicación ArgoCD.

    ```sh
    mv sealed-secret.yaml ~/desktop/kcfp-argocd-app/helm/templates
    ```

7. Subir el fichero `sealed-secret.yaml` añadiendo al repositorio `test-argocd-app`, abriendo una terminal en la carpet `~/test-argocd-app` y ejecutando los siguientes comandos:

    ```sh
    git add helm/templates/sealed-secret.yaml
    git commit -m "fix: add sealed secret"
    git push
    ```

8. Acceder a la aplicación en ArgoCD a través de la URL https://localhost:8080/applications/argocd/test-argocd-app?view=tree&resource= y comprobar como se ha creado el secreto utilizando sealed-secrets, tal y como se puede ver en la siguiente captura, donde se señala este mediante un rectángulo rojo.

    ![ArgoCD App Creation Sealed Secret](./img/argocd_app_creation_sealed_secret.png)

9. Acceder al repositorio a través de la web de DockerHub y convertirlo en privado, para ello simplemente basta con hacer click sobre el repositorio en la pestaña **Options**, y una vez dentro en la sección **Visibility Settings** hacer click sobre el botón **Make private**, tal y como se expone en la siguiente captura.

    ![ArgoCD App Make repo private](./img/argocd_app_make_repo_private.png)

    Se pedirá confirmación solicitando que se introduzca el nombre del repositorio, escribir el nombre del repositorio y pulsar sobre el botón **Make private**, tal y como se puede ver en la siguiente imagen.

    ![ArgoCD App Make repo private confirmation](./img/argocd_app_make_repo_private_confirmation.png)

10. Modificar el fichero `~/test-argocd-app/helm/templates/deployment.yaml` de forma que quede de la siguiente forma:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ include "fast-api-webapp.fullname" . }}
      labels:
        {{- include "fast-api-webapp.labels" . | nindent 4 }}
    spec:
      {{- if not .Values.autoscaling.enabled }}
      replicas: {{ .Values.replicaCount }}
      {{- end }}
      selector:
        matchLabels:
          {{- include "fast-api-webapp.selectorLabels" . | nindent 6 }}
      template:
        metadata:
          {{- with .Values.podAnnotations }}
          annotations:
            {{- toYaml . | nindent 8 }}
          {{- end }}
          labels:
            {{- include "fast-api-webapp.selectorLabels" . | nindent 8 }}
        spec:
          {{- with .Values.imagePullSecrets }}
          imagePullSecrets:
            {{- toYaml . | nindent 8 }}
          {{- end }}
          serviceAccountName: {{ include "fast-api-webapp.serviceAccountName" . }}
          securityContext:
            {{- toYaml .Values.podSecurityContext | nindent 8 }}
          containers:
            - name: {{ .Chart.Name }}
              securityContext:
                {{- toYaml .Values.securityContext | nindent 12 }}
              image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
              imagePullPolicy: {{ .Values.image.pullPolicy }}
              ports:
                - name: http
                  containerPort: {{ .Values.service.port }}
                  protocol: TCP
              livenessProbe:
                httpGet:
                  path: /health
                  port: http
              readinessProbe:
                httpGet:
                  path: /health
                  port: http
              resources:
                {{- toYaml .Values.resources | nindent 12 }}
          {{- with .Values.nodeSelector }}
          nodeSelector:
            {{- toYaml . | nindent 8 }}
          {{- end }}
          {{- with .Values.affinity }}
          affinity:
            {{- toYaml . | nindent 8 }}
          {{- end }}
          {{- with .Values.tolerations }}
          tolerations:
            {{- toYaml . | nindent 8 }}
          {{- end }}
    ```

11. Modificar el fichero `~/test-argocd-app/helm/values.yaml` en la sección `imagePullSecrets` para que quede de la siguiente forma:

    ```yaml
    imagePullSecrets:
      - name: registry-credential
    ```

12. Crear un nuevo secret usando sealed-secrets para el acceso al registry con la URL del API necesaria por argocd-image-updater:

    ```sh
    kubectl create secret docker-registry reg-cred-argocd-image-updater \
    --docker-server="https://registry-1.docker.io/" \
    --docker-username="<dockerhub_username>" \
    --docker-password="<dockerhub_token>" \
    --dry-run=client \
    -n fast-api \
    -o yaml > reg_cred.yaml

    cat reg_cred.yaml | kubeseal \
        --controller-namespace secrets \
        --controller-name sealed-secrets \
        --format yaml \
        -n fast-api \
        > sealed-secret-reg-cred.yaml
    
    rm reg_cred.yaml
    ```

13. Mover el fichero creado `sealed-secret-reg-cred.yaml` al repositorio `test-argocd-app`:

    ```sh
    mv sealed-secret-reg-cred.yaml ~/desktop/kcfp-argocd-app/helm/templates
    ```

14. Añadir el fichero `~/test-argocd-app/helm/templates/rbac-argocd-image-updater.yaml` con el siguiente contenido:

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: argocd-image-updater-secrets
      labels:
        {{- include "fast-api-webapp.labels" . | nindent 4 }}
    rules:
    - apiGroups: ["*"]
      resources: ["secrets"]
      verbs: ["*"]
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: argocd-image-updater-secrets
      labels:
        {{- include "fast-api-webapp.labels" . | nindent 4 }}
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: argocd-image-updater-secrets
    subjects:
    - kind: ServiceAccount
      name: argocd-image-updater
      namespace: argocd
    ```

    > Es necesario para que argocd-image-updater pueda acceder al secret creado para acceder al registry privado de la imagen docker

15. Subir el contenido modificado en la carpeta `~/test-argocd-app` ejecutando los siguientes comandos sobre una terminal en dicha carpeta:

    ```sh
    git add .
    git commit -m "fix: added imagePullSecrets configuration"
    git push
    ```

16. Realizar cambios en el código de la aplicación, para así generar una nueva versión y comprobar que se puede acceder al código del repositorio Docker utilizando el secreto creado anteriormente. Para ello es necesario modificar el fichero `~/test-app-argocd-src/src/application/app.py` de forma que quede tal y como se muestra a continuación:

    ```python
    """
    Module for define API endpoints
    """

    import uvicorn
    from fastapi import FastAPI


    app = FastAPI(
        port=8081
    )

    root_endpoint_message = {"message": "Hello world will be updated automatically in private repository"}
    health_message = {"health": "ok"}

    """
    Root endpoint definition
    """
    @app.get("/")
    async def root():
        """Result of calling the root endpoint
        Returns
        -------
        'Hello world' message
        """
        return root_endpoint_message

    @app.get("/health")
    async def health():
        """Result of calling /health endpoint
        Returns
        -------
        JSON message with health status
        """
        return health_message

    if __name__ == "__main__":
        uvicorn.run(app, host="0.0.0.0", port=8081)
    ```

17. Para hacer pre-commit:

         pip3 install pre-commit
         pip3 install pre-commit
         pre-commit install
         python3 -m venv venv
         source venv/bin/activate
         pip3 install -r src/requirements.txt
         pre-commit run -a

17. Subir los cambios del repositorio, abriendo una terminal sobre el directorio `~/test-app-argocd-src` y ejecutando los siguientes comandos:

    ```sh
    git add .
    git commit -m "fix: changed root entrypoint message"
    git push
    ```

18. Será necesario indicarle a argocd-image-updater las credenciales para acceder al registry de la imagen, para ello es necesario añadir la anotación `argocd-image-updater.argoproj.io/main.pull-secret: reg-cred-argocd-image-updater` en la aplicación de ArgoCD, para ello se desplegará la nueva versión de argocd-apps a través del fichero `argocd-apps/values_v5.yaml`:

    ```sh
    helm -n argocd upgrade --install argocd-apps argo/argocd-apps \
      -f argocd-apps/values.yaml \
      --create-namespace --wait --version 1.1.0
    ```

14. Usar la Ip externa del servicio 'ingress-nginx-controller' de tipo 'LoadBalancer' para conectarse a la aplicación:

        kubectl get svc -n ingress-nginx

20. Realizar una petición al endpoint `/` usando la Ip obtenida en el paso anterior y comprobar la respuesta recibida:

    ```sh
    curl -X 'GET' \
    'http://34.27.234.146/' \
    -H 'accept: application/json'
    ```

    La respuesta obtenida debería ser la siguiente:

    ```sh
    {"message":"Hello world will be updated automatically in private repository"}
    ```

21. Comprobar como se ha cambiado la imagen utilizada para el Deployment `my-app-fast-api-webapp` desplegado en el namespace `fast-api` por la nueva imagen con la nueva versión:

    ```sh
    kubectl -n fast-api get deployment my-app-fast-api-webapp -o=jsonpath='{$.spec.template.spec.containers[:1].image}' | cut -d ':' -f2
    ```

22. Para ver la salida de las métricas de Prometheus expuestas por la aplicación ejecutamos:

        kubectl -n monitoring port-forward service/my-release-simple-server 8000:8000

23. Abriendo http://localhost:8000/ vemos:

        # HELP main_requests_total Total number of requests to main endpoint
        # TYPE main_requests_total counter
        main_requests_total 5.0
        # HELP students_create_total Total number of requests to the endpoint for create a student
        # TYPE students_create_total counter
        students_create_total 4.0
        # HELP joke_requests_total Total number of requests to joke endpoint
        # TYPE joke_requests_total counter
        joke_requests_total 4.0
        # HELP healthcheck_requests_total Total number of requests to healthcheck
        # TYPE healthcheck_requests_total counter
        healthcheck_requests_total 574.0

24. Abrir una nueva pestaña en la terminal y realizar un port-forward del puerto http-web del servicio de Grafana 
    al puerto 3000 de la máquina:

        kubectl -n monitoring port-forward svc/prometheus-grafana 3000:http-web

25. Acceder a la dirección http://localhost:3000. Las credenciales de Grafana por defecto son admin para el usuario y prom-operator para la  
    contraseña. Hacemos un import del fichero custom_dashboard.json, seleccionamos el namespace monitoring y uno de los pods de la aplicación FastAPI. Se pueden observar 7 paneles:

    - 4 paneles dedicados a los endpoints, los cuales registran el número de llamadas recibidas por cada uno de ellos.
    - 1 panel que cuenta el número de veces que la aplicación ha sido iniciada.
    - 1 panel que muestra el número total de llamadas realizadas.
    - 1 panel que muestra el uso actual de CPU en comparación con la cantidad máxima solicitada.

Cambiar el json y poner foto...

#### Configuración de notificaciones en ArgoCD y commit status

1. En el laboratorio anterior se creó un aplicación en GitHub en la sección [Creación de app en GitHub para utilizar con Jenkins](../1-pipelines-github-workflows-jenkins/README.md#creación-de-app-en-github-para-utilizar-con-jenkins). Acceder a la información de la aplicación instalada desde la sección **Developer settings** dentro de la sección **Settings**, tal y como se muestra en la siguiente imagen.

    ![Access developer settings](./img/access_developer_settings.png)

    Seleccionar la aplicación Jenkins, y hacer click en la sección **Edit**, tal y como se señala en la imagen anterior un rectángulo rojo.

2. Se accederá a la información detallada de la aplicación, tal y como se puede ver en la siguiente captura.
    ![Access Jenkins app installed](./img/access_jenkins_app_installed.png)

    Hacer click en la opción **Install App**, señalada en la imagen anterior mediante un rectángulo rojo.

3. En la cuenta donde se ha instalado hacer click sobre el botón de la rueda dentada, tal y como se expone en la siguiente imagen.

    ![Access Jenkins app installed detailed](./img/access_jenkins_app_installed_detailed.png)

4. En la URL del navegador obtener la última parte de la URL, ya que estos son el installationID de la aplicación.

5. Hacer click en la sección **App Settings**, y se accederá a los detalles de la aplicación y ajustes de la misma, tal y como se puede ver en la siguiente captura.

    ![Access Jenkins app installed settings](./img/access_jenkins_app_installed_settings.png)

    Anotar el nombre de **App ID**, señalado en la imagen anterior mediante un rectángulo rojo.

6. Hacer click en la sección **Private keys** sobre el botón **Generate a private key**, tal y como se muestra en la siguiente captura, donde se ha señalado el botón mediante un rectángulo rojo.

    ![Generate app private key](./img/generate_app_private_key.png)

7. Modificar el fichero `argocd/values-notifications.yaml` utilizando el `installationID` y `App ID` de la aplicación obtenidos en los pasos anteriores:

    ```yaml
    notifications:
      argocdUrl: "https://argocd.example.com"
      notifiers:
        service.github: |
          appID: <appID>
          installationID: <appInstallionID>
          privateKey: $github-privateKey
    ```

8. Modificar el fichero `argocd/values-secret.yaml` añadiendo el contenido de la private key descargada en los pasos anteriores en la siguiente sección:

    ```yaml
    notifications:
    secret:
      items:
        github-privateKey: |
          <app_private_key>
    ```

9. Desplegar argocd con los nuevos valores configurados:

    ```sh
    helm -n argocd upgrade --install argocd argo/argo-cd \
      -f argocd/values.yaml \
      -f argocd/values-secret.yaml \
      -f argocd/values-notifications.yaml \
      --create-namespace --wait --version 5.34.4
    ```

10. Es necesario añadir una serie de anotaciones a la aplicación de ArgoCD para que se subscriba a los cambios y notifique de ellos, para ello es necesario desplegar de nuevo la aplicación `test-argocd-app` utilizando los valores configurados en `argocd-apps/values_v6.yaml` donde los cambios más relevantes están en la definición de la aplicación `test-argocd-app`:

    ```yaml
    # test-app
    - name: test-argocd-app
      namespace: argocd
      finalizers:
      - resources-finalizer.argocd.argoproj.io
      project: default
      source:
        helm:
          releaseName: my-app
          valueFiles:
            - values.yaml
        path: helm
        repoURL: git@github.com:xoanmm/test-argocd-app.git
        targetRevision: main
      destination:
        server: https://kubernetes.default.svc
        namespace: fast-api
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
        - CreateNamespace=true
      ignoreDifferences:
      - group: apps
        kind: Deployment
        jsonPointers:
        - /spec/replicas
      additionalAnnotations:
        argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd/argocd-repo-creds-test-argocd-app
        argocd-image-updater.argoproj.io/main.update-strategy: semver
        argocd-image-updater.argoproj.io/main.helm.image-name: image.repository
        argocd-image-updater.argoproj.io/main.helm.image-tag: image.tag
        argocd-image-updater.argoproj.io/main.force-update: 'true'
        argocd-image-updater.argoproj.io/git-branch: main
        argocd-image-updater.argoproj.io/image-list: >-
          main=xoanmallon/test-argocd-app
        argocd-image-updater.argoproj.io/main.pull-secret: >-
          pullsecret:fast-api/reg-cred-argocd-image-updater
            notifications.argoproj.io/subscribe.on-sync-succeeded.github: ''
        notifications.argoproj.io/subscribe.on-deployed.github: ''
        notifications.argoproj.io/subscribe.on-sync-failed.github: ''
        notifications.argoproj.io/subscribe.app-sync-status-unknown.github: ''
    ```

11. Desplegar de nuevo la aplicación `argocd-apps` utilizando los valores del fichero `argocd-apps/values_v6.yaml`:

    ```sh
    helm -n argocd upgrade --install argocd-apps argo/argocd-apps \
      -f argocd-apps/values_v6.yaml \
      --create-namespace --wait --version 1.1.0
    ```

12. Acceder a la URL https://localhost:8080/applications/argocd/test-argocd-app?view=tree&conditions=false&resource= y hacer click sobre el botón `Sync`, tal y como se señala en la siguiente imagen mediante un rectangulo rojo.

    ![ArgoCD App Sync](./img/argocd_app_sync.png)

    En las opciones que se mostrarán dejar todo por defecto y hacer click sobre el botón **Synchronize**, señalado en la siguiente captura mediante un rectángulo rojo.

    ![ArgoCD App Synchronize](./img/argocd_app_synchronize.png)

13. Acceder al repositorio en GitHub para la aplicación de ArgoCD de nombre `test-argocd-app` y comprobar como se ha mandado una notificación cambiando el commit status del último commit.

    ![ArgoCD App Synchronize Notification](./img/argocd_app_synchronize_notification.png)

    Si se hace click sobre la misma se puede observar con más detalle la información.
    ![ArgoCD App Synchronize Notification Detailed](./img/argocd_app_synchronize_notification_detailed.png)

    Haciendo click sobre el botón **Details** señalado mediante un rectángulo rojo en la imagen anterior se accederá a la URL de la aplicación, tal y como se muestra en la siguiente captura.

    ![ArgoCD App Synchronize Notification Detailed View App](./img/argocd_app_synchronize_notification_detailed_view_app.png)


