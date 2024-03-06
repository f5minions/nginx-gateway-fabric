# 예제

본 예제에서 우리는 NGINX Gateway Fabric 및 간단한 웹 애플리케이션을 배포하고 NGINX Gateway Fabric을 HTTPRoute 리소스를 통해서 트래픽을 해당 애플리케이션으로 라우팅하는 것을 확인할 수 있습니다.

## 설정 절차 및 시험 방법

## 1. 첫번째, NGINX Gateway Fabric의 배포

1. NGINX Gateway Fabric의 [설치 가이드](https://docs.nginx.com/nginx-gateway-fabric/installation/)에 따라 NGINX Gateway Fabric을 설치 합니다.

1. Shell에서 NGINX Gateway Fabric의 접속 IP(공인IP)를 변수에 저장 후 테스트에서 활용하려고 합니다.

   ```text
   GW_IP=172.30.1.24
   ```

1. 동일하게 NGINX Gateway Fabric의 접속 포트로 변수에 설정 후 테스트에서 사용하려고 합니다.

   ```text
   GW_PORT=30922
   ```

## 2. 간단한 테스트 애플리케이션인 Cafe 앱을 배포 합니다.

1. 아래와 같이 이미 정의된 coffee 앱과 tea 앱을 배포하고 서비스를 생성 합니다:

   ```shell
   kubectl apply -f cafe.yaml
   ```

1. `default` 네임스페이스에 해당 테스트 애플리케이션이 정상적으로 배포가 되었는가를 확인 합니다.

   ```shell
   kubectl -n default get pods
   ```
   (결과예시)
   ```text
   NAME                      READY   STATUS    RESTARTS   AGE
   coffee-6f4b79b975-2sb28   1/1     Running   0          12s
   tea-6fb46d899f-fm7zr      1/1     Running   0          12s
   ```

## 3. 해당 애플리케이션 접속을 위한 Gateway 및 라우팅 설정을 합니다.

1. Gateway 설정:

   ```shell
   kubectl apply -f gateway.yaml
   ```

1. HTTPRoute 리소스를 이용한 라우팅 설정:

   ```shell
   kubectl apply -f cafe-routes.yaml
   ```

## 4. 아래 방법으로 테스트 애플리케이션의 접속을 확인할 수 있습니다.

본 예제에서는 해당 애플리케이션의 접속을 위해 `curl` 도구를 이용하여 `coffee` 앱 그리고 `tea` 앱으로 요청을 전달 합니다.

coffee 앱의 접속:

```shell
curl --resolve cafe.example.com:$GW_PORT:$GW_IP http://cafe.example.com:$GW_PORT/coffee
```
(결과예시)
```text
Server address: 10.12.0.18:80
Server name: coffee-7586895968-r26zn
```

tea 앱의 접속:

```shell
curl --resolve cafe.example.com:$GW_PORT:$GW_IP http://cafe.example.com:$GW_PORT/tea
```
(결과예시)
```text
Server address: 10.12.0.19:80
Server name: tea-7cd44fcb4d-xfw2x
```

## 5. 이제 "hostname"을 다르게 변경하여 동일한 앱으로 접속을 시도 합니다.

Traffic is allowed to `cafe.example.com` because the Gateway listener's hostname allows `*.example.com`. You can
change an HTTPRoute's hostname to something that matches this wildcard and still pass traffic.

현재 설정된 NGINX Gateway Listener는 `*.example.com`라는 Hostname에 대해서 접속을 허용하는 설정이 되어 있기 때문에 `cafe.example.com` 이라는 호스트는 접속을 정상적으로 할 수 있었습니다.

이제 아래 명령을 통해 tea 앱에 대한 Hostname을 `coffeebean.example.com` 변경하여 접속을 시도하고 결과를 확인 합니다.


```shell
kubectl -n default edit httproute tea
```

Hostname을 변경 후 위에서 수행했던 `curl` 명령으로 `tea` 앱 서비스에 접속을 하면 정상적인 접속이 되지 않습니다. Hostname 설정을 coffeebean.example.com으로 변경 후 접속을 시도하면 정상적인 tea 앱으로 접속을 할 수 있습니다.

마찬가지로 Gateway의 Listener 설정의 Hostname을 다른 이름으로 변경하면 HTTPRoute의 트래픽이 정상적으로 전달되지 않습니다.


예를 들어, 아래 명령을 통해 배포된 Gateway의 설정을 수정하여 Hostname 설정을 `coffeebean.example.com`으로 변경 합니다.

```shell
kubectl -n default edit gateway gateway
```

설정을 변경 후 이전에 사용했던 `curl` 명령을 그대로 사용하여 접속을 시도하면 해당 접속은 허용되지 않을 것이고 `404 Not Found` 결과를 확인할 수 있습니다. 이는 Gateway 레벨에서 허용할 Hostname을 확인 후 HTTPRoute를 통해 트래픽을 애플리케이션으로 라우팅을 하기 때문 입니다. 
