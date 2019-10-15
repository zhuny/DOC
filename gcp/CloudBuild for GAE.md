# Intro
* github와 [GAE](https://console.cloud.google.com/appengine)간의 간단한 CI/CD 설정 방법

# Step 1
1. github에서 master 브랜치로 push가 되면 trigger를 생성한다. 일반적으로는 PR에 의해 master 브랜치로 merge될 경우이다.
1. 기본적인 Build 설정

# Trigger
github에서 master 브렌치에 푸시를 하면 Cloud build에서 자동으로 실행할 수 있도록 설정하는 작업이다.
1. [Cloud Build](https://console.cloud.google.com/cloud-build/triggers)에서 먼저 github와 연동하여 새로운 repo를 연결한다.
    * APP 권한에서 어떤 repo와 연동할 지 선택할 수 있으며 필요한 repo만 읽을 수 있도록 그때그때 권한을 추가한다.
1. 새로운 trigger를 추가하여 어떤 이벤트를 처리해 줄 수 있도록 할 지 설정한다.
    * 대표적으론 Pull Request와 Push to Master가 있고, 상황에 맞춰 여러가지를 더 구성한다.
1. `cloudbuild.yaml` 파일에서 빌드할 수 있도록 명시해 줘야 한다.

# Build 1
빌드를 하기 위해서 필요한 단계들을 명시해 준다. 원래 `gcloud app deploy`를 통해서 빌드한 뒤 코드를 실행시켰을 것이다.
1. `cloudbuild.yaml`의 파일 구성
```
steps:
  - name: gcr.io/cloud-builders/gcloud
    args: ['app', 'deploy']
```
앞으로 step들이 많이 추가되는데, 이 step은 무조건 가장 마지막에 와야 한다.

# Step 2
1. 암호화를 위한 키를 발급 받아서 세팅 파일 암호화

# KMS
1. [Key Management System](https://console.cloud.google.com/security/kms)에서 `Key Ring`과 `Key`를 생성한다. `Key Ring`은 `Key`를 모아둔 그룹이다.
1. `gcloud kms encrypt`를 이용하여 세팅 파일 등등을 암호화 하여 레포에 포함시킨다.
1. 빌드할 때 암호화한 파일을 복호화하여 사용한다.

# Build 2
1. `cloudbuild.yaml` 파일에 복호화를 위한 step을 추가한다.
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

# Step 3
여기까지 하면 최소한의 필요한 부분은 끝났다. 지금 step은 필요가 없는 경우 건너뛰어도 된다.
1. trigger에 의해 github에서 코드를 가져올 경우, submodule 코드는 따로 가져오고 있지 않기 때문에 수동으로 추가해 줘야 한다.
1. 레포가 public인 경우 아무 권한없이 clone할 수 있지만, private인 경우, 별도의 권한이 필요하다.

# Clone
먼저 레포가 public인 경우.
1. 필요한 submodule들을 정리해서 clone받는다. 
1. `git submodule` 커멘드가 있지만, 현 디렉토리는 소스코드는 있지만 git의 정보를 가지고 있지 않아서 사용할 수 없다.

# Build 3-1
```
steps:
  - name: gcr.io/cloud-builders/git
    args:
      - clone
      - git@github.com:zhuny/commons.git
      - commons
    volumes:
      - name: ssh
        path: /root/.ssh
```

# Clone private repo
만약 레포가 private인 경우
1. ssh를 통해 별도의 인증 과정이 있어야지 clone을 받을 수 있다.
1. 관련 내용은 [해당 링크](https://cloud.google.com/cloud-build/docs/access-private-github-repos)에 자세히 나와있으니 참고.

# Build 3-2
가이드에 따라 [해당 step 부분](https://cloud.google.com/cloud-build/docs/access-private-github-repos#prepare_the_build)이 추가된다.

# Done
이러면 이제 master에 머지를 할 때마다 자동으로 GAE로 코드가 갱신된다.

# Note
여러가지 알아둬야 할 사항
1. KMS의 키는 주기적으로 갱신이 되기 때문에 키가 만료되면 암호화된 파일들은 사용할 수 없게 된다. 그래서 그 전에 새로운 키로 갱신해 주어야 한다.
1. GAE로 디플로이 하는 과정에서 `gcloud app deploy`로 인해 새로운 build를 만들어 버린다. 이로 인해 이중으로 build가 생기며 총 build시간이 늘어나며, 불필요한 과금이 발생시킬 수 있다. 하루에 120분까지는 무료로 사용할 수 있다.
