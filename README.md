

# PROYECTO FINAL KEEPCODING 7 

## Software necesario

- [docker](https://docs.docker.com/engine/install/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [helm](https://helm.sh/docs/intro/install/)
- [kubeseal](https://github.com/bitnami-labs/sealed-secrets#kubeseal)

En este proyecto hemos implementado un proceso de despliegue automático utilizando ArgoCD y hemos configurado 2 repositorios en GitHub para separar y distinguir diferentes conceptos. El primer repositorio, llamado ***kcfp-app-argocd-src***, contiene el código fuente de la aplicación. Aquí se encuentran todos los archivos y componentes relacionados con la lógica de la aplicación. El segundo repositorio, llamado ***kcfp-argocd-app***, almacena los manifiestos de Kubernetes, incluyendo tanto la definición de la aplicación como los sealed secrets cifrados. Utilizamos Helm para empaquetar estos manifiestos. Este repositorio contiene los archivos YAML que describen cómo se deben desplegar y configurar los recursos de Kubernetes para la aplicación, así como los secretos cifrados que se utilizan en ella. La configuración de "sealed secrets" nos permite mantener la seguridad de los secretos sensibles, ya que se almacenan cifrados y solo se pueden descifrar dentro del clúster. Esto garantiza que los secretos no estén expuestos en texto plano en el repositorio. En cuanto a Continuous Integration (CI) hemos utilizado GitHub Actions para realizar acciones automatizadas cada vez que se realizan cambios en el repositorio de código con flujos de trabajo (lint, test y release) para realizar tareas como verificar la calidad del código, ejecutar pruebas y lanzar nuevas versiones de la aplicación. Hemos aprovechado ***semantic release*** para generar automáticamente versiones basadas en los cambios en el repositorio y generar notas de lanzamiento. En cuanto a Continuous Deployment (CD), he utilizado ArgoCD con la metodología GitOps para gestionar y desplegar la aplicación de manera automatizada. GitOps es una metodología para gestionar y operar aplicaciones en entornos de Kubernetes utilizando Git como fuente única de verdad. En el contexto de GitOps, todos los recursos de la aplicación, como configuraciones, definiciones de despliegue y políticas, se almacenan y versionan en un repositorio Git. La idea principal es que cualquier cambio en el estado deseado de la aplicación se refleje automáticamente en el clúster de Kubernetes a través de un proceso de reconciliación continua. En GitOps, el ciclo de vida de una aplicación se gestiona a través de la utilización de herramientas como ArgoCD, FluxCD o Jenkins. Estas herramientas se encargan de comparar el estado actual del clúster con la definición almacenada en el repositorio Git y, si hay diferencias, realizan las acciones necesarias para alcanzar el estado deseado. Esto puede incluir desplegar nuevas versiones de la aplicación, realizar actualizaciones, revertir cambios y gestionar secretos. Al elegir ArgoCD utilizo tambien el repositorio ***gitops-con-argocd*** que contiene archivos YAML con configuraciones específicas para desplegar el chart predefinido de ArgoCD en GKE. Este repositorio no se carga en Git, sino que se utiliza localmente. Desplegar ArgoCD y la aplicación en GKE de GCP proporciona beneficios como una gestión simplificada de Kubernetes, alta disponibilidad y escalabilidad, integración con servicios de Google Cloud, seguridad y cumplimiento normativo, así como una estrecha integración entre ArgoCD y el entorno de GKE. Estas ventajas combinadas ayudan a mejorar la eficiencia, la confiabilidad y la seguridad de la aplicación.
La aplicación se basa principalmente en el material abordado durante la asignatura de SRE. Hemos integrado algunas de las aplicaciones usadas durante las prácticas del bootcamp, agregando también nuevas funcionalidades y elementos. Es una aplicación web FastAPI que consta de 7 endpoints y se conecta a una base de datos de MongoDB.
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
   manifiestos de la apliacion y los Sealed Secrets. Los Sealed Secrets son secretos encriptados que se utilizan para proteger información sensible, como contraseñas o claves de API.
   Al utilizar una deploy key, se puede otorgar acceso al repositorio que contiene los sealed secrets a herramientas como Sealed Secrets Controller, permitiendo que esta herramienta
   automatizada desencripte y despliegue los secretos de forma segura. Para ello será necesario realizar los siguientes pasos:
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


         

6. Para exponer la aplicación y permitir que tenga una IP externa, se requiere la configuración de un objeto Ingress y un Ingress Controller que esté vinculado a ese objeto Ingress.
   El objeto Ingress es un recurso de Kubernetes que define reglas de enrutamiento para el tráfico entrante hacia los servicios dentro del clúster. Estas reglas especifican cómo se
   debe redirigir el tráfico a los servicios en función de las rutas y otros criterios.
   El Ingress Controller es el componente responsable de implementar y hacer cumplir las reglas definidas en el objeto Ingress. El Ingress Controller se encarga de gestionar los
   recursos de red subyacentes (como balanceadores de carga o servicios de enrutamiento) y garantizar que el tráfico llegue a los servicios adecuados en función de las reglas de
   enrutamiento definidas en el objeto Ingress. Para instalar Nginx Ingress Controller en GCP-GKE ejecutar:

       kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.6.4/deploy/static/provider/cloud/deploy.yaml

8. Añadimos el repositorio de helm prometheus-community para poder desplegar el chart kube-prometheus-stack:

       helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
       helm repo update

9. Desde la pestaña ubicada en el repositorio ***kcfp-argocd-app*** desplegar el chart kube-prometheus-stack del repositorio de helm añadido en el paso anterior con los valores
   configurados en el archivo kube-prometheus-stack/values.yaml en el namespace fast-api:

       helm -n fast-api upgrade --install prometheus prometheus-community/kube-prometheus-stack -f kube-prometheus-stack/values.yaml --create-namespace --wait --version 34.1.1

10. Añadimos el repositorio helm de argocd:

        helm repo add argo https://argoproj.github.io/argo-helm
        helm repo update
   
11. Desde la pestaña ubicada en el repositorio ***gitops-con-argocd*** desplegar el helm chart de argocd utilizando el fichero `argocd/values.yaml` y el fichero `argocd/values-secret.yaml`:

    ```sh
    helm -n argocd upgrade --install argocd argo/argo-cd \
      -f argocd/values.yaml \
      -f argocd/values-secret.yaml \
      --create-namespace \
      --wait --version 5.34.1
    ``` 

12. Realizar un port-forward al servicio de argocd al puerto 8080 local:

        kubectl port-forward service/argocd-server -n argocd 8080:443

13. Obtener la contraseña de acceso a argocd mediante el siguiente comando:

        kubectl -n argocd get secret argocd-initial-admin-secret \
        -o jsonpath="{.data.password}" | base64 -d

14. Acceder a la url http://localhost:8080 y utilizar como credenciales el nombre de usuario admin y la contraseña obtenida en el paso anterior.

### Configuración de Sealed Secrets con actualización automática

1. Se desplegará la aplicación [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) como una aplicación de ArgoCD añadiendola en la lista de `argocd-apps`.
   El controlador de Sealed Secrets es responsable de descifrar los objetos SealedSecret y generar los Secretos originales en el clúster y tiene la clave privada necesaria para
   descifrar los secretos sellados y cifrados con su clave pública. El Sealed Secrets Controller está vinculado al secreto TLS, que creamos a continuación, que contiene la clave
   pública y privada. Este controlador actúa como una capa adicional de seguridad, ya que solo el controlador tiene acceso a la clave privada necesaria para descifrar los secretos
   sellados. A continuación se muestra el contenido del fichero `gitops-con-argocd/argocd-apps/values-sealed-secrets.yaml`:

    ```yaml
   applications:
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

3. Antes es necesario realizar una serie de pasos para preconfigurar la clave de encriptación. Para que no se modifique el contenido de los repositorios remotos,
   desde la pestaña ubicada en el repositorio ***gitops-con-argocd*** o desde cualquier otra ubicación ejecutamos:

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

    3. Crear el namespace donde se va a desplegar Sealed Secrets:

        ```sh
        kubectl create namespace $NAMESPACE
        ```

    4. Un secreto TLS en Kubernetes se utiliza para almacenar certificados y claves privadas utilizados en conexiones seguras. Se compone de un certificado (clave publica) y una
       clave privada que garantizan la identidad del servidor o cliente. Este secreto se usa para asegurar las comunicaciones entre el controlador de Sealed Secrets y otros
       componentes de Kubernetes. En resumen, el uso de claves públicas y privadas, junto con el secreto TLS, permite un cifrado seguro y confiable de los secretos en Kubernetes.
       Creamos entonces un `Secret` de tipo tls, utilizando el par de claves RSA creado anteriormente:

        ```sh
        kubectl -n "$NAMESPACE" create secret tls \
          "$SECRETNAME" --cert="$PUBLICKEY" \
          --key="$PRIVATEKEY"
        kubectl -n "$NAMESPACE" label secret \
          "$SECRETNAME" \
          sealedsecrets.bitnami.com/sealed-secrets-key=active
        ```

3. Desde la pestaña ubicada en el repositorio ***gitops-con-argocd*** desplegar el controlador de Sealed Secrets como una nueva aplicación de ArgoCD utilizando el fichero
   `argocd-apps/values-sealed-secrets.yaml`:

    ```sh
    helm -n argocd upgrade --install argocd-apps argo/argocd-apps \
      -f argocd-apps/values-sealed-secrets.yaml \
      --create-namespace --wait --version 1.1.0
    ```

4. Comprobar como se dispone de una nueva aplicación llamada `sealed-secrets`, tal y como se muestra en la siguiente captura:

  <img width="1792" alt="Screenshot 2023-06-07 at 15 06 48" src="https://github.com/maciuozz/gitops-con-argocd/assets/118285718/8706ef87-cbc3-4c42-89fd-47559e973a4b">


5. Creamos un nuevo Secret utilizando kubeseal para acceder al repositorio creado en DockerHub. Para cifrar (o sellar) un secreto, kubeseal interactúa con el controlador de
   Sealed Secrets. En el contexto de Sealed Secrets, "sellado" se utiliza para referirse al proceso de cifrar los
   secretos utilizando una clave pública. Cuando se crea un secreto utilizando kubeseal, el controlador de Sealed Secrets toma el secreto sin cifrar y lo cifra
   utilizando una clave pública específica del clúster. Esto garantiza que el secreto esté protegido de manera segura mientras se transfiere o almacena en un repositorio de código
   fuente, como Git. El secreto cifrado resultante se almacena en un archivo YAML, como sealed-secret.yaml. Este archivo YAML contiene el secreto cifrado y también incluye metadatos
   que indican al controlador de Sealed Secrets cómo descifrarlo cuando se despliega en el clúster.
   ***--docker-server="https://index.docker.io/v1/"*** especifica el servidor de Docker llamado "index.docker.io". Este servidor es la URL de la API de índice de Docker. El índice de Docker es responsable de realizar la
   búsqueda y recuperación de imágenes de contenedores en el Docker Hub. Este secret sirve para proporcionar a los pods del clúster de
   Kubernetes las credenciales de acceso al registro de Docker especificado, lo que permite que los contenedores y las aplicaciones desplegadas en el clúster puedan autenticarse y
   acceder a las imágenes almacenadas en ese registro. Será necesario recuperar el token de DockerHub creado anteriormente:

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

7. Mover el fichero creado `sealed-secret.yaml` al repositorio ***kcfp-argocd-app***:

    ```sh
    mv sealed-secret.yaml ~/desktop/kcfp-argocd-app/helm/templates
    ```

8. Crear un nuevo secret que proporciona las credenciales de acceso al registry de Docker necesarias para el componente argocd-image-updater de Argo CD, permitiéndole acceder y
    descargar las imágenes de Docker necesarias durante el proceso de despliegue y actualización de aplicaciones.
    ***--docker-server="https://registry-1.docker.io/"*** especifica el servidor de Docker llamado "registry-1.docker.io". Este servidor es la URL principal del Docker
    Hub, que es un registro de Docker público ampliamente utilizado. Se utiliza para almacenar y distribuir imágenes de contenedores. Será necesario recuperar el
    token de DockerHub creado anteriormente:

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

9. Mover el fichero creado `sealed-secret-reg-cred.yaml` al repositorio ***kcfp-argocd-app***:

    ```sh
    mv sealed-secret-reg-cred.yaml ~/desktop/kcfp-argocd-app/helm/templates
    ```
10. Subir los ficheros `sealed-secret.yaml` y `sealed-secret-reg-cred.yaml` añadiendolos al repositorio ***kcfp-argocd-app***:

    ```sh
    git add .
    git commit -m "fix: added sealed-secret.yaml and sealed-secret-reg-cred.yaml"
    git push
    ```

11. Antes de realizar el despliegue de la aplicación, transformamos nuestro repositorio de DockerHub en privado. En el mundo real, es muy común tener repositorios privados
    de Docker para garantizar la seguridad y la gestión adecuada de las imágenes utilizadas en las aplicaciones. Para ello, dentro del repositorio, vamos a ***Settings --> Make
    private***:

<img width="1792" alt="Screenshot 2023-06-21 at 15 17 17" src="https://github.com/maciuozz/gitops-con-argocd/assets/118285718/61479dad-e654-42d1-96d0-d0eb534800af">

11. Es importante destacar estos fragmentos de codigo que pertenecen al ***deployment.yaml*** y al ***values.yaml*** respectivamente:

    ```yaml
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
    ```
    ```yaml
    imagePullSecrets:
      - name: registry-credential
    ```
    La declaración `imagePullSecrets` se utiliza para asociar un secreto de autenticación al Deployment. Al especificar `imagePullSecrets:`, se está indicando que el pod (contenedor)
    necesita acceder a un registro de Docker privado y utilizar un secreto específico para la autenticación. En este caso, se está especificando `imagePullSecrets:` con el nombre
    `registry-credential`, lo que significa que se está asociando el Sealed Secret del fichero `sealed-secret.yaml` creado anteriormente, que se llama `registry-credential`, al
    Deployment. El sealed secret `registry-credential` contiene la información confidencial necesaria para autenticarse en el registro de Docker y descargar las imágenes del
    contenedor. El campo encryptedData del sealed secret almacena la información cifrada, en este caso, .dockerconfigjson, que representa las credenciales cifradas del registro.
    Cuando el Deployment hace referencia al secreto `registry-credential` a través de imagePullSecrets, Kubernetes desencriptará automáticamente el
    sealed secret y lo utilizará para autenticar la descarga de imágenes del registro de Docker.


13. En el repositorio ***kcfp-argocd-app/helm/templates*** vale la pena comentar tambien el fichero `rbac-argocd-image-updater.yaml`:

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

    Es un archivo de definición de roles y permisos (RBAC), Role-Based Access Control. Se define un ClusterRole llamado `argocd-image-updater-secrets`, que tiene permisos para acceder a recursos de
    tipo secrets en cualquier grupo de API (apiGroups: ["\*"]), incluyendo el Sealed Secret del fichero `sealed-secret-reg-cred.yaml` que se llama `reg-cred-argocd-image-updater` creado anteriormente.
    El ClusterRole permite todas las operaciones (verbs: ["\*"]) en los recursos secrets. Además, se define un ClusterRoleBinding llamado `argocd-image-updater-secrets` que vincula el ClusterRole
    anterior al ServiceAccount llamado `argocd-image-updater` en el namespace argocd. Esto permite que el ServiceAccount tenga los permisos definidos por el ClusterRole para acceder a los recursos secrets.
    En resumen, este código establece los roles y permisos necesarios para que el ServiceAccount `argocd-image-updater` tenga acceso y autorización para interactuar con los secrets en el clúster de Kubernetes.

14. Desde la pestaña ubicada en el repositorio ***kcfp-app-argocd-src*** hacer pre-commit:

         pip3 install pre-commit
         pip3 install pre-commit
         pre-commit install
         python3 -m venv venv
         source venv/bin/activate
         pip3 install -r src/requirements.txt
         pre-commit run -a

15. Finalmente vamos a desplegar el image updater y la aplicación desde la pestaña ubicada en el repositorio ***gitops-con-argocd***. Para ello se desplegará la nueva versión de
    argocd-apps a través del fichero `argocd-apps/values.yaml`:

    ```sh
    helm -n argocd upgrade --install argocd-apps argo/argocd-apps \
      -f argocd-apps/values.yaml \
      --create-namespace --wait --version 1.1.0 
    ```
    
    Tendriamos que obtener algo parecido:
    
    <img width="1790" alt="Screenshot 2023-06-21 at 16 59 09" src="https://github.com/maciuozz/gitops-con-argocd/assets/118285718/5d6d97e4-cdec-4e2a-b150-156bb877eed1">
    
    Si seleccionamos la aplicación, podemos observar lo que se muestra en las capturas a continuación, donde vemos todos los recursos desplegados y notamos la versión 1.0.28:
    
    <img width="1791" alt="Screenshot 2023-06-21 at 17 04 40" src="https://github.com/maciuozz/gitops-con-argocd/assets/118285718/33f886de-e022-46b7-91d6-6eaaa7cbc84e">
    <img width="1792" alt="Screenshot 2023-06-21 at 17 04 09" src="https://github.com/maciuozz/gitops-con-argocd/assets/118285718/193c25b0-ba60-49b5-8d4e-d36fcb0b270f">
    
16. Usar la Ip externa del servicio 'ingress-nginx-controller' de tipo 'LoadBalancer' para conectarse a la aplicación:

        kubectl get svc -n ingress-nginx

    Si nos conectamos a la aplicación desplegada por ArgoCD vemos:
    
    <img width="1791" alt="Screenshot 2023-06-22 at 15 42 17" src="https://github.com/maciuozz/gitops-con-argocd/assets/118285718/4c15522d-9d18-46aa-96ed-b02f96a56d7e">

    La aplicación tiene 7 endpoint:
    - /health (GET):  Se utiliza para configurar las sondas de `livenessProbe` y `readinessProbe`. Estas sondas se utilizan para verificar el estado de un
      contenedor en ejecución y determinar si está vivo y listo para recibir tráfico.
    - / (GET): Es el endpoint principal y devuelve un mensaje relacionado con el punto de entrada principal de la aplicación.
    - /analyze-text-file (POST): Permite seleccionar un archivo de input y analizarlo. Calcula la frecuencia de las palabras en el archivo y devuelve los resultados del análisis.
    - /api/student (POST): Agrega un nuevo estudiante a la base de datos utilizando los datos proporcionados en el cuerpo de la solicitud. Devuelve el estudiante recién creado.
    - /students/{student_id}/{field_name}/{field_value} (PUT): Actualiza un campo específico de un estudiante en la base de datos, identificado por su ID. Devuelve el estudiante
      actualizado.
    - /allstudents (GET): Obtiene una lista de todos los estudiantes almacenados en la base de datos.
    - /joke (GET): Devuelve un chiste aleatorio obtenido de una API externa.
      
17. Realizamos un cambio en el archivo ***kcfp-app-argocd-src/src/application/app.py*** con el objetivo de generar una nueva versión y que ArgoCD la detecte automáticamente. En este
    cambio, modificamos el `root_endpoint_message` como se muestra a continuación:

    ```python
    
    ...

    app = FastAPI()

    root_endpoint_message = {"message": "The new version will be detected and updated automatically in private repository"}

    ...
    ```



18. Subimos los cambios del repositorio desde la pestaña ubicada en el repositorio ***kcfp-app-argocd-src***. Vemos 2 ejemplos de sintaxis para realizar un commit que
    son ambos validos:

    ```sh
    git add .
    git commit -m "fix: changed root endpoint message" --> git commit -m "fix(app): changed root endpoint message"
    git push
    ```


        


19. Esperamos algunos minutos para que se despliegue la nueva version y realizamos una petición al endpoint `/` usando la Ip obtenida en el paso anterior para
     comprobar la respuesta recibida:

    ```sh
    curl -X 'GET' \
    'http://34.27.234.146/' \
    -H 'accept: application/json'
    ```

    La respuesta obtenida debería ser la siguiente:

    ```sh
    {"message":"The new version will be detected and updated automatically in private repository"}
    ```

20. Comprobar como se ha cambiado la imagen utilizada para el Deployment `my-app-fast-api-webapp` desplegado en el namespace `fast-api` por la nueva imagen con la nueva versión:

    ```sh
    kubectl -n fast-api get deployment my-app-fast-api-webapp -o=jsonpath='{$.spec.template.spec.containers[:1].image}' | cut -d ':' -f2
    ```

21. Para ver la salida de las métricas de Prometheus expuestas por la aplicación ejecutamos:

        kubectl -n fast-api port-forward service/my-app-fast-api-webapp 8000:8000

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
        # HELP frequencies_requests_total Total number of requests to frequency analyzer endpoint
        # TYPE frequencies_requests_total counter
        frequencies_requests_total 7.0
        # HELP get_all_students_total Total number of requests to the endpoint to get a list of students
        # TYPE get_all_students_total counter
        get_all_students_total 50.0
        # HELP students_update_total Total number of requests to the endpoint to update a student
        # TYPE students_update_total counter
        students_update_total 8.0
        # HELP server_requests_total Total number of requests to this webserver
        # TYPE server_requests_total counter
        server_requests_total 26990.0
        # HELP app_start_count_total Number of times the application has started
        # TYPE app_start_count_total counter
        app_start_count_total 1.0

24. Abrir una nueva pestaña en la terminal y realizar un port-forward del puerto http-web del servicio de Grafana 
    al puerto 3000 de la máquina:

        kubectl -n fast-api port-forward svc/prometheus-grafana 3000:http-web

25. Acceder a la dirección http://localhost:3000. Las credenciales de Grafana por defecto son admin para el usuario y prom-operator para la  
    contraseña. Hacemos un import del fichero custom_dashboard.json, seleccionamos el namespace monitoring y uno de los pods de la aplicación FastAPI. Se pueden observar 10 paneles:

    - 7 paneles dedicados a los endpoints, los cuales registran el número de llamadas recibidas por cada uno de ellos.
    - 1 panel que cuenta el número de veces que la aplicación ha sido iniciada.
    - 1 panel que muestra el número total de llamadas realizadas.
    - 1 panel que muestra el uso actual de CPU en comparación con la cantidad máxima solicitada.

<img width="1792" alt="Screenshot 2023-06-22 at 15 33 34" src="https://github.com/maciuozz/gitops-con-argocd/assets/118285718/e3f3e982-3840-4888-8218-5ab745738959">





