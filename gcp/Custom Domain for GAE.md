# Intro
GAE를 사용하고 있을 때, custom domain을 설정하는 방법

# Register
* [GAE Domain 설정](https://console.cloud.google.com/appengine/settings/domains)에서 먼저 자신이 소유하고 있는 도메인을 등록해야 한다. 이 과정은 도메인당 한번만 하면 된다.
* 페이지의 과정에 따라 subdomain을 등록해서 원하는 도메인이 GAE로 등록될 수 있도록 한다.

# Dispatch Routes
* GAE에서 라우팅을 할 수 있도록 해야 한다. 규칙은 [GAE 서비스 목록](https://console.cloud.google.com/appengine/services)에서 확인할 수 있다.
* `dispatch.yaml`파일에 다음과 같이 url을 설정해서 라우팅한다. 더 자세한 방법은 [요청 라우팅 방법](https://cloud.google.com/appengine/docs/standard/python/how-requests-are-routed)에 설명되어 있다.
```
dispatch:
  - url: my.domain.com/*
    service: my
```
* 설정이 된 이후 `gcloud app deploy dispatch.yaml`를 이용해서 등록한다.
* 프로젝트마다 하나의 `dispatch.yaml`을 사용해서 관리해야 한다.
