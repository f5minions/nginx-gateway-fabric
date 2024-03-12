# 트래픽 분할 예제

이번 예제에서는 NGINX Gateway Fabric을 배포하고 카페 애플리케이션에 대한 트래픽을 분할하는 예제를 시험 합니다. 카페 애플리케이션은 2개의 버전 `coffee-v1`, `coffee-v2`으로 배포 후 HTTPRoute의 트래픽 분할 기능을 통해 시험을 진행 합니다.


## 시험방법

## 1. NGINX Gateway Fabric을 배포 합니다.

1. [설치가이드](https://docs.nginx.com/nginx-gateway-fabric/installation/)를 참고하셔서 NGINX Gateway Fabric을 배포 합니다.

1. NGINX Gateway Fabric 배포 후 트래픽 시험을 위해 접속할 수 있는 공인 IP를 쉘 변수로 등록 합니다:

   ```text
   GW_IP=XXX.YYY.ZZZ.III
   ```

1. NGINX Gateway Fabric 시험을 위한 TCP 포트로 쉡 변수로 등록 합니다:

   ```text
   GW_PORT=<port number>
   ```

## 2. 카페 애플리케이션을 배포 합니다.

1. 카페 애플리케이션에 대한 deployment 및 service를 배포 합니다:

   ```shell
   kubectl apply -f cafe.yaml
   ```

1. `default` 네임스페이스에 카페 애플리케이션의 정상 배포를 확인 합니다:

   ```shell
   kubectl -n default get pods
   ```

   ```text
   NAME                         READY   STATUS    RESTARTS   AGE
   coffee-v1-7c57c576b-rfjsh    1/1     Running   0          21m
   coffee-v2-698f66dc46-vcb6r   1/1     Running   0          21m
   ```

## 3. 애플리케이션을 위한 라우팅을 설정 합니다

1. Gateway의 생성:

   ```shell
   kubectl apply -f gateway.yaml
   ```

1. HTTPRoute 리소스의 생성:

   ```shell
   kubectl apply -f cafe-route.yaml
   ```

이 HTTPRoute 리소스에는 `/coffee` URL Path에 대해 80% 트래픽은 `coffee-v1` 서비스로 전달하고, 20%의 트래픽은 `coffee-v2`서비스로 전달하는 설정 입니다.

이 시험에서 우리는 80:20 비율을 사용하지만 꼭 100%를 기준으로 비율을 설정할 필요는 없습니다. 예를 들어, 8:2 또는 16:4 또는 32:8 이런 비율을 설정해도 동일한 비율로 계산되어 사용할 수 있습니다.

## 4. 애플리케이션 테스트

다른 예제와 동일하게 애플리케이션에 대한 접속 테스트는 `curl` 명령을 통해 진행되며, `/coffee` URL Path로 접속 시험을 진행 합니다:

```shell
curl --resolve cafe.example.com:$GW_PORT:$GW_IP http://cafe.example.com:$GW_PORT/coffee
```

시험결과 80%의 요청에 대해서는 `coffee-v1` 서비스를 통해 처리되어 응답이 옵니다:

```text
Server address: 10.12.0.18:80
Server name: coffee-v1-7c57c576b-rfjsh
```

그리고 20%의 요청은 `coffee-v2` 서비스를 통해 처리되어 응답이 오는 것을 확인할 수 있습니다:

```text
Server address: 10.12.0.19:80
Server name: coffee-v2-698f66dc46-vcb6r
```

### 5. 트래픽 분할 비율을 조정하여 시험

이번엔 `coffee-v2` 서비스로의 트래픽을 더 많은 비율로 설정하여 시험을 진행 합니다. HTTPRoute 리소스 설정에서 `coffee-v2` 서비스에 대한 비율을 80으로 변경하여 업데이트 합니다. 이 설정을 통해 `coffee-v1` 서비스와 `coffee-v2` 서비스는 동일하게 80이란 비율 설정을 가지게 됩니다.

이 설정으로 동일한 비율을 가진 서비스로 트래픽이 균등 분배되어 처리되는 것을 확인할 수 있습니다.

1. HTTPRoute 리소스를 변경하여 업데이트:

   ```shell
   kubectl apply -f cafe-route-equal-weight.yaml
   ```

2. 위와 동일한 트래픽을 통해 결과를 확인:

   ```shell
   curl --resolve cafe.example.com:$GW_PORT:$GW_IP http://cafe.example.com:$GW_PORT/coffee
   ```

시험결과 예상과 같이 `coffee-v1` 서비스와 `coffee-v2` 서비스에 트래픽이 균등하게 분배되어 처리되는 것을 확인할 수 있습니다. 추가적으로 `coffee-v2` 서비스에 대한 비율을 계속 증가시키면서 트래픽에 대한 응답을 확인하고, 만약 `coffee-v2` 서비스에 문제가 발생할 경우 즉시 `coffee-v1` 으로 트래픽이 전달됨을 함께 확인을 할 수 있습니다.
