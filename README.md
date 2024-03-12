[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/nginxinc/nginx-gateway-fabric/badge)](https://api.securityscorecards.dev/projects/github.com/nginxinc/nginx-gateway-fabric)
[![FOSSA Status](https://app.fossa.com/api/projects/custom%2B5618%2Fgithub.com%2Fnginxinc%2Fnginx-gateway-fabric.svg?type=shield)](https://app.fossa.com/projects/custom%2B5618%2Fgithub.com%2Fnginxinc%2Fnginx-gateway-fabric?ref=badge_shield)

# NGINX Gateway Fabric이란?

NGINX Gateway Fabric은 [NGINX](https://nginx.org/)를 [K8s Gateway API](https://gateway-api.sigs.k8s.io/)의 데이터플랜으로 구현하기 위해 제공되는 오픈소스 프로젝트 입니다. 이 프로젝트의 목표는 핵심 Gateway API(`Gateway`, `GatewayClass`, `HTTPRoute`, `TCPRoute`, `TLSRoute` 및 `UDPRoute`)를 구현하여 K8s 기반의 애플리케이션에 대해 HTTP 또는 TCP/UDP 로드 밸런서, 역방향 프록시 또는 API 게이트웨이를 구성하는 것입니다. NGINX 게이트웨이 패브릭은 Gateway API의 모든 기능을 지원합니다.

지원하는 Gateway API 리소스 및 기능들에 대해서는 아래 링크를 참고하실 수 있습니다.

참고문서:[Gateway API Compatibility](https://docs.nginx.com/nginx-gateway-fabric/overview/gateway-api-compatibility/).

그리고 이 오픈소스 프로젝트에 대한 전체적인 컨셉 및 디자인 아키텍처는 아래의 링크를 참고하세요.
디자인 아키텍처 (Design principles](/docs/developer/design-principles.md) 그리고 컨셉 [Architecture](https://docs.nginx.com/nginx-gateway-fabric/overview/gateway-architecture/).


## 시작하기

1. [kind 클러스터에서 빠르게 시작하기](https://docs.nginx.com/nginx-gateway-fabric/installation/running-on-kind/).
2. NGINX Gateway Fabric의 [설치](https://docs.nginx.com/nginx-gateway-fabric/installation/) .
3. 배포 이미지는 [GitHub Container Registry](https://github.com/nginxinc/nginx-gateway-fabric/pkgs/container/nginx-gateway-fabric)를 통해 사전 빌드된 이미지 또는 소스에서 NGINX Gateway Fabric 이미지를 직접 [빌드](https://docs.nginx.com/nginx-gateway-fabric/installation/building-the-images/)할 수 있습니다.
4. 다양한[예제코드](examples)를 통해 직접 설정해보기.
5. [사용자 가이드](https://docs.nginx.com/nginx-gateway-fabric/how-to/)를 읽어보기.


그리고 [NGINX Documentation](https://docs.nginx.com/nginx-gateway-fabric/) 웹사이트에서 상세한 NGINX Gateway Fabric에 관련된 사용자 가이드 문서를 직접 참조할 수 있습니다.

## NGINX Gateway Fabric 릴리스

우리는 GitHub의 [릴리스 페이지](https://github.com/nginxinc/nginx-gateway-fabric/releases)에 직접 NGINX Gateway Fabric에 퍼블리싱을 합니다.

현재 가장 최신 버전은 [1.1.0](https://github.com/nginxinc/nginx-gateway-fabric/releases/tag/v1.1.0) 입니다.

Edge 버전은 아직 릴리스에 게시되지 않은 새로운 기능을 실험적으로 사용해보는데 유용 합니다. 사용하려면 메인 브렌치의 [최신커밋](https://github.com/nginxinc/nginx-gateway-fabric/commits/main)에서 *edge* 버전을 선택하면 됩니다.

아래 표에는 이미지, 매니페스트, 문서 및 예제와 관련된 옵션이 요약되어 있으며 링크가 제공 됩니다:

| 버전        | 설명                              | 설치                                                            | 문서 및 예제                                                                                                                                                 |
|----------------|------------------------------------------|-----------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Latest release | 프로덕션 용도                       | [패키지](https://github.com/nginxinc/nginx-gateway-fabric/tree/v1.1.0/deploy) | [문서](https://docs.nginx.com/nginx-gateway-fabric)/[예제](https://github.com/nginxinc/nginx-gateway-fabric/tree/v1.1.0/examples)                           |
| Edge           | 가장 최신의 업데이트 기능 및 실험적 용도 | [패키지](https://github.com/nginxinc/nginx-gateway-fabric/tree/main/deploy)   | [문서](https://github.com/nginxinc/nginx-gateway-fabric/tree/main/site/content)/[예제](https://github.com/nginxinc/nginx-gateway-fabric/tree/main/examples) |

### 버저닝

NGF는 시맨틱 버저닝 기반으로 릴리스를 관리 합니다. 자세한 내용은 https://semver.org 를 참고하세요

> 메이저 버전 0 `(0.Y.Z)` 는 개발용으로 예약되어 있으므로 언제든지 변경될 수 있기 때문에 공개 API가 안정적이지 않을 수 있습니다.

### 출시 및 개발계획

다음 릴리스에 포함될 기능은 해당 [milestone](https://github.com/nginxinc/nginx-gateway-fabric/milestones)을 참고할 수 있으며, 이슈 생성 및 릴리스 할당에 대한 정보는 [Issue Lifecycle](ISSUE_LIFECYCLE.md) 문서를 참고할 수 있습니다.


## 기술사양

NGINX Gateway Fabric이 지원하는 소프트웨어 버전에 대한 정보는 아래 표를 참고하세요.

| NGINX Gateway Fabric | Gateway API | Kubernetes | NGINX OSS | NGINX Plus |
|----------------------|-------------|------------|-----------|------------|
| Edge                 | 1.0.0       | 1.23+      | 1.25.3    | R31        |
| 1.1.0                | 1.0.0       | 1.23+      | 1.25.3    | n/a        |
| 1.0.0                | 0.8.1       | 1.23+      | 1.25.2    | n/a        |
| 0.6.0                | 0.8.0       | 1.23+      | 1.25.2    | n/a        |
| 0.5.0                | 0.7.1       | 1.21+      | 1.25.x *  | n/a        |
| 0.4.0                | 0.7.1       | 1.21+      | 1.25.x *  | n/a        |
| 0.3.0                | 0.6.2       | 1.21+      | 1.23.x *  | n/a        |
| 0.2.0                | 0.5.1       | 1.21+      | 1.21.x *  | n/a        |
| 0.1.0                | 0.5.0       | 1.19+      | 1.21.3    | n/a        |

\*설치패키지의 NGINX 컨테이너 이미지에서 부 버전(1.25)을 사용하고 패치 버전은 표기가 되어 있짐 않습니다. 이 부분은 가능한 최신 패치 버전이 사용됨을 의미 합니다.

## SBOM (소프트웨어 BOM)

바이너리 및 Docker 이미지에 대한 SBOM을 생성 합니다.

### 바이너리

바이너리용 SBOM은 릴리스 페이지에서 사용할 수 있으며,[syft](https://github.com/anchore/syft)를 사용하여 SPDX 형식으로 제공 됩니다.

### 도커이미지

도커이미지의 SBOM은 다음에서 사용할 수 있습니다. [GitHub Container](https://github.com/nginxinc/nginx-gateway-fabric/pkgs/container/nginx-gateway-fabric)
repository. SBOM은 [syft](https://github.com/anchore/syft)를 사용하여 생성되고 이미지 패키지 형태로 저정됩니다.

예를 들어, `linux/amd64`에 대한 SBOM을 검색하고 [grype](https://github.com/anchore/grype)으로 분석을 하려면 아래 예시와 같이 명령을 실행할 수 있습니다:

```shell
docker buildx imagetools inspect ghcr.io/nginxinc/nginx-gateway-fabric:edge --format '{{ json (index .SBOM "linux/amd64").SPDX }}' | grype
```

## 문제해결

문제 해결을 위해서는 [Troubleshooting](https://docs.nginx.com/nginx-gateway-fabric/how-to/monitoring/troubleshooting/) 문서를 참고하시기 바랍니다.

## 연락하기


여러분의 피드백을 항상 듣고 싶습니다! NGINX Gateway 컨트롤러에 문제가 발생하는 경우 깃허브의 [버그오픈][bug]로 업데이트 주실 수 있습니다.

제안 사항이나 개선 요청이 있는 경우 깃허브 토론[아이디어 열기][idea]를 통해서 보내주세요. 

Kubernetes@nginx.com을 통해 직접 문의하거나 `#nginx-gateway-fabric` Slack 채널의 [NGINX Community Slack]을 통해 문의할 수 있습니다.

[bug]:https://github.com/nginxinc/nginx-gateway-fabric/issues/new?assignees=&labels=&projects=&template=bug_report.md&title=

[idea]:https://github.com/nginxinc/nginx-gateway-fabric/discussions/categories/ideas

[slack]: https://nginxcommunity.slack.com/channels/nginx-gateway-fabric

## 커뮤니티 미팅

2024년 가능한 자주 커뮤니티 미팅/밋업을 통해 NGINX 커뮤니티를 활성화할 수 있도록 하겠습니다.


## 지원

NGINX Gateway Fabric은 커뮤니티 프로젝트이기 때문에 현재 F5 지원 조직을 통한 기술지원은 받을 수 없습니다. 커뮤니티 채널(slack) 및 버그 채널(github/bug)를 통해 문의하시기 바랍니다.
