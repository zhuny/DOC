# 개요
* github과 연동해서 자동으로 build한다.

# 기초
1. github trigger setting
1. build 실행하기

# Trigger
깃헙에서 master 브렌치에 푸시를 하면 Cloud build에서 자동으로 실행할 수 있도록 설정하는 작업이다.
1. [Cloud Build](https://console.cloud.google.com/cloud-build/triggers)에서 새로운 trigger를 설정한다.
1. `codebuild.yaml` 파일에서 빌드할 수 있도록 명시해 줘야 한다.

# Build
빌드를 하기 위해서 필요한 단계들을 명시해 준다. 원래 `gcloud app deploy`를 통해서 빌드한 뒤 코드를 실행시켰을 것이다.
1. `codebuild.yaml` 파일에 저 커맨드를 넣어준다.
```
steps:
  - name: gcr.io/cloud-builders/gcloud
    args: ['app', 'deploy']
```
