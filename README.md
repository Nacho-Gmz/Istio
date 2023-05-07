# Istio
#### Gómez Aldrete Luis Ignacio
#### 216588253
#
Para este ejemplo, primero se tiene que bajar [Istio](https://github.com/istio/istio/releases) de su repositorio aquí en GitHub y agregar su carpeta *bin* al Path; esta carpeta con tiene *istioctl.exe, que es encargado de los comandos de Istio en la terminal. 

![image](https://user-images.githubusercontent.com/80866790/236693653-9d9ace01-2e3f-481f-958a-e659ee815adb.png)

Además de haber agregado Istio al Path, se requiere tener Docker y Kubernetes instalados. Para verificar que Istio fue correctamente agregado al Path, se puede usar el comando 

```cmd
istioctl version
```

y si el resultado no es el siguiente, quiere decir que Isstioctl no fue correctamente agregado al Path.

![image](https://user-images.githubusercontent.com/80866790/236694049-56259dc6-2187-4b80-8a82-d503a5cdcf6d.png)

Ahora para probar las funcionalidades de Istio, se instalará un perfil demo con el comando 

```cmd
$ istioctl install --set profile=demo -y
```

![image](https://user-images.githubusercontent.com/80866790/236694525-d6c6f230-e3d0-4ecd-b7b7-e735c841b2d8.png)

Ahora se procede a desplegar la aplicación de ejemplo, pero primero se agrega un label al namespace que permite la injección automática de proxies Envoy con el siguiente comando

```cmd
kubectl label namespace default istio-injection=enabled
```

Después de haber configurado esto, ahora si se despliega la aplicación

```cmd
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

![image](https://user-images.githubusercontent.com/80866790/236694804-a725e335-70e7-4c08-a50b-eb4c7dbf522b.png)

Para comprobar el despliegue me metí al dashboard de minikube

![image](https://user-images.githubusercontent.com/80866790/236694842-0e0ade80-ab69-403f-8b4d-2ba272ff5ec0.png)

![image](https://user-images.githubusercontent.com/80866790/236694898-e4caee5f-2060-4c1b-8da3-9b569a9a3720.png)

Y lo comprobe con kubectl en la línea de comandos

![image](https://user-images.githubusercontent.com/80866790/236695147-f4bd59f7-6ff2-426a-8d61-c4025b22bce0.png)

Para abrir la aplicación al tráfico externo, se necesita usar el siguiente comando 

```cmd 
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

Y para determinar las IP y los puertos de Ingress se tienen que usar los siguiente comandos (OJO, los últimos 4 comandos solo funcionan en una terminal Bash)

```bash
minikube tunnel
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

Y después de verificar la dirección de acceso con ```echo "http://$GATEWAY_URL/productpage"```, me pude dirigir al resultado para ver que la apliación tenía acceso externo. 

![image](https://user-images.githubusercontent.com/80866790/236695629-c4502bcd-59c8-4d86-8032-f65594028903.png)

Para tener una representación más visual, se puede hacer uso de los addons de Istio como Kiali, Prometheus, Grafana o Jaeger. Estos son agregados a la aplicación con el siguiente comando

```bash
kubectl apply -f samples/addons
```

Y para comprobar que hayan finalizado de agregase, se puede verificar el estado con el siguiente comando 

```bash
kubectl rollout status deployment/kiali -n istio-system
```

Y se tiene que esperar a que se haya desplegado Kiali correctamente, para después con el comando ```istioctl dashboard kiali``` acceder a la dashboard. Para simular tráfico se puede usar el comando ```for i in $(seq 1 100); do curl -s -o /dev/null "http://$GATEWAY_URL/productpage"; done```.

![image](https://user-images.githubusercontent.com/80866790/236696375-889d4e03-80eb-4b52-91c8-420815caefa1.png)

![image](https://user-images.githubusercontent.com/80866790/236696284-c15ba469-e6dc-4445-94cd-74a91a3fac41.png)

![image](https://user-images.githubusercontent.com/80866790/236696480-7ce9e25a-5274-46e6-8366-bd124aba72ae.png)
