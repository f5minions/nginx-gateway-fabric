# 서로 다른 네임스페이스 참조모델 시험

이번 예제는 앞서 진행했던 기본 카페 애플리케이션 [cafe-example](../cafe-example)에서 ReferenceGrant를 통해 백엔드의 앱이 HTTPRoute와 다른 네임스페이스에 있을 경우에 대한 확장 기능 입니다.

## 시험방법

## 1. NGINX Gateway Fabric을 배포

1. [설치가이드](https://docs.nginx.com/nginx-gateway-fabric/installation/)를 참고하여 NGINX Gateway Fabric을 배포 합니다.

1. NGINX Gateway Fabric에 대한 트래픽 시험을 위해 공인 IP를 shell 변수에 저장 합니다:

   ```text
   GW_IP=XXX.YYY.ZZZ.III
   ```

1. NGINX Gateway Fabric 접속을 위한 TCP Port 정보도 동일하게 shell 변수에 저장 합니다:

   ```text
   GW_PORT=<port number>
   ```

## 2. 카페 애플리케이션을 배포
1. 먼저 카페 애플리캐이션을 위한 다른 네임스페이스를 생성 후 카페 애플리케이션을 배포 합니다:

   ```shell
   kubectl apply -f cafe-ns-and-app.yaml
   ```

1. `cafe` 라는 네임스페이스에 배포된 애플리케이션을 확인 합니다.:

   ```shell
   kubectl -n cafe get pods
   ```

   ```text
   NAME                      READY   STATUS    RESTARTS   AGE
   coffee-6f4b79b975-2sb28   1/1     Running   0          12s
   tea-6fb46d899f-fm7zr      1/1     Running   0          12s
   ```

## 3. 카페 애플리케이션을 위한 라우팅 설정

1. Gateway를 생성:

   ```shell
   kubectl apply -f gateway.yaml
   ```

1. HTTPRoute 리소스를 생성:

   ```shell
   kubectl apply -f cafe-routes.yaml
   ```

1. ReferenceGrant 리소스를 생성:

   ```shell
   kubectl apply -f reference-grant.yaml
   ```

   이 ReferenceGrant 리소스에는 `cafe` 네임스페이스에 있는 모든 서비스에 대해서 `default` 네임스페이스에서 참조할 수 있는 권한을 부여하는 설정 입니다.

## 4. 애플리케이션 접속 테스트

`curl` 명령을 이용해서 `coffee` 서비스와 `tea` 서비스에 접속 테스트를 진행 합니다.

 coffee 앱 서비스 접속:

```shell
curl --resolve cafe.example.com:$GW_PORT:$GW_IP http://cafe.example.com:$GW_PORT/coffee
```

```text
Server address: 10.12.0.18:80
Server name: coffee-7586895968-r26zn
```

tea 앱 서비스 접속:

```shell
curl --resolve cafe.example.com:$GW_PORT:$GW_IP http://cafe.example.com:$GW_PORT/tea
```

```text
Server address: 10.12.0.19:80
Server name: tea-7cd44fcb4d-xfw2x
```

## 5. ReferenceGrant의 제거

Step 3에서 생성했던 ReferenceGrant 리소스를 제거하면 `cafe` 네임스페이스의 리소스에 대한 접근을 제거할 수 있으며, 애플리케이션 접속 테스트 시 백엔드 서비스와의 통신 문제로 500 에러가 발생함을 확인할 수 있습니다.

```shell
kubectl delete -f reference-grant.yaml
```

동일하게 카페 애플리케이션을 HTTP를 통해 접속을 시도하면 다음과 같이 internal server error를 확인할 수 있습니다:

```shell
curl --resolve cafe.example.com:$GW_PORT:$GW_IP http://cafe.example.com:$GW_PORT/tea
```

```text
<html>
<head><title>500 Internal Server Error</title></head>
<body>
<center><h1>500 Internal Server Error</h1></center>
<hr><center>nginx/1.25.1</center>
</body>
</html>
```

그리고 이 상태에서 HTTPRoute 설정의 컨디션을 확인하면 `coffee` 서비스와 `tea` 서비스에 대해 참조가 허용되지 않았음을 확인할 수 있습니다:

```shell
kubectl describe httproute coffee
```

```text
Condtions:
      Message:               Backend ref to Service cafe/coffee not permitted by any ReferenceGrant
      Observed Generation:   1
      Reason:                RefNotPermitted
      Status:                False
      Type:                  ResolvedRefs
      Controller Name:       gateway.nginx.org/nginx-gateway-controller
```

```shell
kubectl describe httproute tea
```

```text
Condtions:
      Message:               Backend ref to Service cafe/tea not permitted by any ReferenceGrant
      Observed Generation:   1
      Reason:                RefNotPermitted
      Status:                False
      Type:                  ResolvedRefs
      Controller Name:       gateway.nginx.org/nginx-gateway-controller
```
