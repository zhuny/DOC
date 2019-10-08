# 개요
* github와 [GAE](https://console.cloud.google.com/appengine)간의 간단한 CI/CD 설정 방법

# 단계 1
1. github에서 master 브랜치로 push가 되면 trigger를 생성한다. 일반적으로는 PR에 의해 master 브랜치로 merge될 경우이다.
1. 기본적인 Build 설정

# Trigger
깃헙에서 master 브렌치에 푸시를 하면 Cloud build에서 자동으로 실행할 수 있도록 설정하는 작업이다.
1. [Cloud Build](https://console.cloud.google.com/cloud-build/triggers)에서 새로운 trigger를 설정한다.
1. `codebuild.yaml` 파일에서 빌드할 수 있도록 명시해 줘야 한다.

# Build 1
빌드를 하기 위해서 필요한 단계들을 명시해 준다. 원래 `gcloud app deploy`를 통해서 빌드한 뒤 코드를 실행시켰을 것이다.
1. `codebuild.yaml`의 파일 구성
```
steps:
  - name: gcr.io/cloud-builders/gcloud
    args: ['app', 'deploy']
```

# 단계 2
1. 암호화를 위한 키를 발급 받아서 세팅 파일 암호화

# KMS
1. [Key Management System](https://console.cloud.google.com/security/kms)에서 `Key Ring`과 `Key`를 생성한다. `Key Ring`은 `Key`를 모아둔 그룹이다.
1. `gcloud kms encrypt`를 이용하여 세팅 파일 등등을 암호화 하여 레포에 포함시킨다.
1. 빌드할 때 암호화한 파일을 복호화하여 사용한다.

# Build 2
1. `codebuild.yaml` 파일에 steps를 추가한다. `Build 1`보다 선행되어야 한다.
```
steps
  - name: gcr.io/cloud-builders/gcloud
    args:
      - kms
      - decrypt
      - --ciphertext-file=settings.py.enc
      - --plaintext-file=settings.py
      - --location=asia-northeast1
      - --keyring=keyring
      - --key=key
```
