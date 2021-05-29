# velog-jenkins-posting-react-app
velog에 젠킨스 배포 셋업 방법을 보여주기 위해서 만든 레포. velog : https://velog.io/@kimsehwan96/Jenkins-Github%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%A6%AC%EC%95%A1%ED%8A%B8-%EC%95%B1-%EC%9E%90%EB%8F%99-%EB%B0%B0%ED%8F%AC-with-aws-S3

호스팅된 URL :  http://velog-jenkins-deploy-posting.s3-website.ap-northeast-2.amazonaws.com/

![](https://images.velog.io/images/kimsehwan96/post/e72875a1-bf2e-4238-8794-dedfeab21ffb/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.35.42.png)

> 이 글에서는 Jenkins, Github, aws S3를 활용하여 리액트 앱을 배포(호스팅)하는 방법을 소개합니다.
> 호스팅은 S3 버킷 정적 호스팅 기능을 이용하며, 추후 Route53, Cloudfront와 같은 AWS 서비스를 사용하여 도메인 연결까지도 가능합니다.

우선 Jenkins가 EC2나 개인 서버에 설치가 되었다고 가정하고 시작합니다.. (이 글에서는 EC2에 Jenkins를 올려놓았을 때 기준입니다.)

## Jenkins의 Pipeline 이란?

우리는 Jenkins의 Pipeline 기능을 활용하여 배포 프로세스를 구축할 예정입니다.
Pipeline이란 Jenkins Job(빌드/테스트/배포 등을 Job으로 부릅니다.)을 연속적으로 수행 가능하도록 만들 수 있는 기능입니다.

![](https://images.velog.io/images/kimsehwan96/post/d6a11344-27b1-4231-ba70-0c4c8e8876f8/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.37.51.png)

위 사진에 보이는 5개의 리스트가 각각 하나의 `Pipeline` 이라고 생각하면 됩니다.

`Pipeline`은 `스크립트` 형태로 작성하여 사용하게 되는데, 뒤에서 설명하기 앞서 어떤 모양인지 아래 사진에서 확인해봅시다.

![](https://images.velog.io/images/kimsehwan96/post/9d4bcc2f-d226-41a3-8600-2a6fae7d3421/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.40.49.png)

위 사진을 보면 `shell script`와 비슷하지만 다른 스크립트 형태로 구성되어있음을 알 수 있습니다!


## Pipeline 생성

![](https://images.velog.io/images/kimsehwan96/post/0c78afc5-3ae1-478e-a2a4-85b55fde0ff2/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.44.13.png)

새로운 Item 을 클릭합니다.

![](https://images.velog.io/images/kimsehwan96/post/d102a483-89f8-497f-9f73-34cba48c3c5f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.46.09.png)

item name에 각자 원하는 내용을 적습니다! (한글로 작성해도 문제 없습니다!)

그리고 Pipeline을 클릭후 OK 버튼을 눌러주세요!

![](https://images.velog.io/images/kimsehwan96/post/37bbbbfd-1db6-4538-a203-21f6f20960e5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.47.18.png)

위와 같이 세부 내용을 작성하는 칸으로 넘어갔습니다!

설명 역시 원하는 내용을 작성하면 됩니다. 

여기서 우리는 `이 빌드는 매개변수가 있습니다.` 를 선택합니다.

Jenkins Pipeline Job 이 실행 될 때, 스크립트에 변수를 주입 할 수 있는데요. 우리는 이 기능을 통해서 `git branch` 이름을 매개변수로 사용하여 master / main / develop 등 브랜치를 지정하여 빌드 가능하도록 만들어 볼 예정입니다!

![](https://images.velog.io/images/kimsehwan96/post/e7e1f316-0342-482b-826f-d5c036eeaff9/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.35.08.png)

요렇게 매개변수 명, Default Value, 설명을 적어줍니다.

매개변수명은 앞으로 `script`에서 변수로서 사용 가능한데. `{BRANCH}` 와 같이 접근 가능합니다.

![](https://images.velog.io/images/kimsehwan96/post/d35008e7-9f0a-4e5a-9cc8-07365f45b21b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.36.05.png)

빌드 트리거는 빌드를 유발시키는 방법이라고 생각하면 됩니다. 추후 Github Webhook 을 이용하여 PR이 Merge되는 경우 자동으로 빌드되도록 설정할 계획입니다. 지금 단계에서는 패스

![](https://images.velog.io/images/kimsehwan96/post/bce4f2e6-6031-49e3-8d66-c3c0ec6130c0/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.37.02.png)

맨 밑으로 가보면 `Pipeline Script`를 생성할 수 있습니다. 빈 공백이 벌써부터 막막하게 느껴지지만 차근 차근 해봅시다.

![](https://images.velog.io/images/kimsehwan96/post/b8d00bf0-a454-49af-bc9a-ca8fbc39073d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.39.31.png)

Pipeline Script에 다음과 같이 작성해봅시다.

```groovy
pipeline {
    agent any

    stages {
        stage('prepare') {
            steps {
                echo 'prepare'
            }
        }
        stage('build') {
            steps {
                echo 'build'
            }
        }
        stage('deploy') {
            steps {
                echo 'deploy'   
            }
        }
    }
}
```

위 스크립트는 단순히 `echo` 명령어를 각 단계에서 수행하는 스크립트입니다. 이 스크립트를 통해 원리를 이해하고 시작해봅시다.


stages의 경우 `단계`를 분리하는 역할입니다. 보통은 `prepare`, `build`, `deploy`와 같은 세단계로 구분하는게 편한데, 상황에 따라서는 `test` 혹은 `post`와 같은 단계를 추가하기도 합니다. 네이밍은 여러분의 몫입니다!

추후 스크립트를 완성할 때 `prepare`는 `git clone` 및 `yarn install`, 혹은 `.env` 주입과 같은 행위를 할 예정입니다.

`build`는 `CI=false yarn build`와 같은 리액트 앱 프로덕션 빌드 명령어를 수행 할 예정입니다.

`deploy`에서는 `aws cli`를 통하여 S3 버킷에 정적호스팅이 가능하도록 빌드된 파일을 업로드 할 예정입니다.

우선 위 스크립트를 저장해봅시다.

![](https://images.velog.io/images/kimsehwan96/post/5ac85f05-9f9b-44b4-90e3-539de6abe3cc/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.43.56.png)

왼쪽 메뉴의 `Build with Parameters`를 클릭

![](https://images.velog.io/images/kimsehwan96/post/60bcc6fa-6a83-4718-8e57-330bd86d3693/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.44.26.png)

이후 빌드하기를 눌러봅시다.


![](https://images.velog.io/images/kimsehwan96/post/0e7bb790-f09d-463f-aed1-97d00fa6dbb7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.45.37.png)

성공했다면 위와 같이 단계별로 성공여부 및 수행 시간이 표시됩니다.

한번 `#1`,`#2`와 같은 번호를 눌러서 들어가봅시다.

![](https://images.velog.io/images/kimsehwan96/post/a4b5aaec-b40c-42b7-bec2-1f5b6b95fff2/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.46.34.png)

왼쪽의 `Console Output`을 클릭

![](https://images.velog.io/images/kimsehwan96/post/b8de8619-f264-4299-a1a5-7dc8f66af329/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.46.37.png)

위와 같이 우리가 만든 Pipeline의 수행 결과가 나옵니다. 추후 `git`, `yarn`과 같은 명령어를 수행한다면 해당하는 명령어를 수행하며 나오는 표준 출력이 콘솔 출력에 그대로 찍혀 나올 것 입니다.

## Github 세팅

Github에서 git clone을 하기 위해 계정 세팅을 해야 합니다.(github 로그인 ID/Password)

> Github ID/Password 기반의 priavate repo clone 기능은 올해 말 deprecated 되는것으로 알고 있습니다. Jenkins 자체에 github OAuth 관련 설정을 하는것이 더 좋겠지만 이 내용은 추후 포스팅 하겠습니다.

![](https://images.velog.io/images/kimsehwan96/post/8e925ec6-5af6-4649-be9d-cef9b6bb375b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.17.52.png)

젠킨스 메인 메뉴의 Jenkins 관리를 클릭합니다.

![](https://images.velog.io/images/kimsehwan96/post/af373991-a34d-48eb-b6ac-59ff5b94b605/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.15.33.png)

중간에 보이는 `Security` 메뉴의 Manage Credentials를 클릭합니다.

![](https://images.velog.io/images/kimsehwan96/post/9e199c91-adf3-4656-8188-315cd4d1b110/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.17.31.png)

`global` 부분에 마우스를 올리면 Add credentials가 보입니다. 클릭합니다.

![](https://images.velog.io/images/kimsehwan96/post/fb245f6a-10ed-454f-90e1-00b5207e97c9/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.13.51.png)

Username에는 깃허브 Username
Password에는 깃허브 Password

ID는 `GIT_ACCOUNT`로 설정합니다. ID는 파이프라인 스크립트에서 우리가 방금 설정한 `credential`을 구분하는 변수명이라고 할 수 있습니다.

OK 버튼을 클릭합니다.

## Jenkins Tool 설정(git, node)

이제 Jenkins 파이프라인에서 사용하기 위한 각종 명령어 도구를 세팅하는 작업입니다.

![](https://images.velog.io/images/kimsehwan96/post/af373991-a34d-48eb-b6ac-59ff5b94b605/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.15.33.png)

역시 Jenkins 관리 화면에 들어간 이후 플러그인 관리를 클릭합니다.

![](https://images.velog.io/images/kimsehwan96/post/cbecce4f-188e-4df7-bd87-d96713e9b07c/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.24.32.png)

위 사진과 같이 `설치 가능` 탭을 클릭한 이후 node를 검색하여 `NodeJS` 플러그인을 설치합니다. 설치는 왼쪽 아래 `Install without restart`를 클릭합니다.

해당 플러그인 설명에도 나와있습니다만. 이 플러그인을 설치하는 이유는 우리가 파이프라인 스크립트에서 node기반 명령어를 사용하기 위해서입니다.(yarn)

설치가 다 되었다면 이제 `git`과 `node`를 `설정` 해주어야 합니다.


![](https://images.velog.io/images/kimsehwan96/post/af373991-a34d-48eb-b6ac-59ff5b94b605/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.15.33.png)

이번엔 Global Tool Configuartion을 들어갑니다.

![](https://images.velog.io/images/kimsehwan96/post/a8dce171-bd05-42a4-8de2-3b759bcb6885/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.27.13.png)

`Git` 부분을 스크린샷과 같이 수정합니다.

`Name`을 `git`으로, `Install automatically`를 체크해줍니다.

`Name`의 경우 파이프라인 스크립트에서 어떤 이름으로 `git`을 사용할거냐고 물어보는 내용입니다. 당연히 우리가 평소에 하듯 `git`이라는 이름으로 사용하면 좋겠죠.

다음은 `NodeJS` 툴 설정입니다.

![](https://images.velog.io/images/kimsehwan96/post/1988ef00-0c47-4f6a-bd8b-de7315adaca3/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.29.28.png)


저는 Name을 `node14`로 정했습니다. 밑에 Version 선택에서 여러분이 원하는 버전으로 설정 가능합니다. 웬만하면 16버전을 선택했다면 Name을 `node16`으로 하시는게 좋겠습니다.

Global npm packages to install 의 경우 스크립트가 수행될 때 디폴트로 사전에 설치할 npm 패키지를 명시하는 내용입니다.

저는 `yarn`만 설치했습니다. 위 사진에 나온것처럼 버전 명시해서 설치가 가능합니다. 우리가 로컬에서 개발 할 때 `npm install -g`로 설치하는 친구들을 설치하는게 좋겠습니다.

위 사진처럼 세팅하고 `Save`를 눌러줍니다.


## 파이프라인 스크립트 완성

자 이제 모든 사전 작업은 완료되었습니다. 이제 남은건 파이프라인 스크립트를 깃허브 clone, 의존성 설치, build 등의 과정을 알아서 하도록 작성하는 일 뿐입니다!

스크립트를 작성하기 전에 여러분의 깃 레포를 확인해봅시다.

![](https://images.velog.io/images/kimsehwan96/post/5e797ec9-182c-439c-acb6-160c686ea82b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.37.51.png)

전 이렇게 루트 디렉터리에 `myapp`이라는 디렉터리가 있고, 이 디렉터리 내부에 `src` 디렉터리나 `package.json`, `yarn.lock`등의 세팅 파일이 존재합니다.

따라서 `git clone`을 한 이후에 `myapp` 디렉터리에서 `yarn install`, `yarn build`등 의 작업을 해야합니다.

따라서 저는 스크립트를 이렇게 작성하도록 하겠습니다.

```groovy
pipeline {
    agent any
    tools {
        nodejs "node14"
        git "git"
    }
    stages {
        stage('prepare') {
            steps {
                echo 'prepare'
                 git branch: "${BRANCH}", credentialsId: "GIT_ACCOUNT", url: 'https://github.com/kimsehwan96/velog-jenkins-posting-react-app.git'
                 sh  'ls -al'
            }
        }
        stage('build') {
            steps {
                    dir('myapp'){
                        sh 'ls -al'
                        sh "yarn install"
                        sh "CI=false yarn build"
                }
            }
        }
        stage('deploy') {
            steps {
                sh "ls -al"
                echo 'deploy'   
            }
        }
    }
}

```

위 내용을 살펴보면.. tools를 설정하고 (git, node) 

prepare 단계에서 git clone을 진행합니다. 

이후 `build` 단계에서 `myapp` 디렉터리로 진입하고(dir('myapp') 구분이 디렉터리로 진입하는 구문입니다.), 이 디렉터리에서 패키지 설치 & 빌드등의 작업을 수행합니다.

`sh "ls -al"`의 경우 각 단계에서 작업하는 디렉터리가 어떤 구조로 되어있고, 어떤식으로 구성되는지 보여드리고자 작성했습니다.

자 이렇게 저장하고 한번 배포해보겠습니다.

![](https://images.velog.io/images/kimsehwan96/post/fb2ad3ca-4968-4514-9602-ee44ebf8ea9d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.47.16.png)

`prepare` 단계의 로그입니다.

`git clone`을 한다는 내용의 로그가 다수 보입니다. 

`ls -al`을 하였을 때 `git clone`한 디렉터리 내부로 들어왔음을 확인 할 수 있습니다.

![](https://images.velog.io/images/kimsehwan96/post/f699c3cd-a790-4362-90c2-9c8bd63dcac9/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.47.27.png)

`build` 단계의 로그입니다. `dir('myapp')`을 통해 작업 디렉토리를 `myapp`으로 이동했습니다. 

이후 `yarn install` 명령어가 수행되며 패키지들을 설치합니다.

![](https://images.velog.io/images/kimsehwan96/post/379ed6e9-5997-41e0-b02a-a934ea7f3f82/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.47.35.png)

`CI=false yarn build` 명령어가 수행되었습니다. 프로덕션 빌드가 되었네요.

현재 작업중인 디렉터리에 `build` 디렉터리 내부에 이 프로덕션 빌드 된 앱 파일들이 존재하겠죠?

![](https://images.velog.io/images/kimsehwan96/post/7058a13b-0553-4cf1-b6b4-87007ab9d1b1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.47.50.png)

`deploy` 단계입니다. 실제로 우리가 배포하진 않았습니다만. `deploy` 단계에서의 `ls -al` 출력을 보면 `prepare` 단계에서의 디렉터리 구조와 동일함을 알 수 있습니다.

즉 각 `stage`가 넘어갈 때마다 작업하는 디렉터리가 이렇게 변경됨을 알 수 있습니다.

이는 이러한 파이프라인 스크립트를 통해 수행되는  jenkins job이 `jenkins` 내부의 `workspace` 공간에서 이루어지기 때문입니다.

![](https://images.velog.io/images/kimsehwan96/post/ec5470e9-7dfd-4489-8c36-1adc0fbdaf9a/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.55.24.png)

`/var/lib/jenkins/workspace/React App Deploy` 와 같이 

`{jenkins 홈 디렉터리 경로}/workspace/{job이름}` 공간에서 이러한 빌드 작업이 이루어지기 때문이라고 보시면 되겠습니다. 

각 stage는 이렇게 `{jenkins 홈 디렉터리 경로}/workspace/{job이름}` 공간으로 이동하여 각종 명령어들을 수행하게 됨을 주의하세요!

자 이제 deploy 부분만 마무리하면 되겠습니다. 

이 부분이 잘 이해가 되지 않는다면 `S3 정적 웹 호스팅` 에 대해서 검색하신 뒤 보시면 편하겠습니다.

![](https://images.velog.io/images/kimsehwan96/post/3c11eae7-e547-43a4-bb77-4a9ed67f9271/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.58.55.png)

자 이렇게 버킷을 하나 S3에 생성했습니다.

![](https://images.velog.io/images/kimsehwan96/post/6326f4a1-44db-4862-98ba-0b3c73a9c8ab/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.59.01.png)

속성 부분에서 맨 밑으로 내려보시면 정적 웹 사이트 호스팅 부분이 있습니다. 편집을 클릭합니다.

![](https://images.velog.io/images/kimsehwan96/post/c1492104-5255-4253-9b73-a1f9c9b54ead/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.59.14.png)

따로 건드려줄건 없어보입니다. 인덱스 문서를 `index.html`로 작성해주고 저장합니다.

![](https://images.velog.io/images/kimsehwan96/post/15633c55-fe05-4bf6-a1bf-f0fc7ab15f95/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%203.11.56.png)

추가적으로 `권한` 설정에 들어가서 위 사진처럼 퍼블릭한 접근을 허용합시다.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ReadOnly",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::{여러분의 버킷명}/*"
        }
    ]
}
```

이제 `Jenkins`가 구동되는 서버 내에 `AWS Credential`을 설정해주어야 합니다.

여러분이 `Jenkins`를 설치한 서버에 접속합니다. 이후 다음 명령어를 입력하세요

```bash
sudo su
su jenkins
cd
```

위 명령어들을 입력하면 `jenkins` 유저로 로그인하게되고, `jenkins` 홈 디렉터리로 이동하게 됩니다.

`aws cli`가 설치되어있지 않다면 설치하고, 설치되어있다면 설치하시면 됩니다.

```bash
aws configure
```

![](https://images.velog.io/images/kimsehwan96/post/3d5ffbf4-dc16-4d04-914e-9a0c679b0960/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%203.04.37.png)

이렇게 여러분 계정의 `AWS Credential`을 `Jenkins` 유저에게 설정해줍니다. 이 과정을 거치면 이제 `jenkins job`에서 `aws cli` 명령어를 마음껏 수행 가능한 상태가 됩니다.

이제 파이프라인 스크립트를 마무리합시다.

```groovy
pipeline {
    agent any
    tools {
        nodejs "node14"
        git "git"
    }
    stages {
        stage('prepare') {
            steps {
                 echo 'prepare'
                 git branch: "${BRANCH}", credentialsId: "GIT_ACCOUNT", url: 'https://github.com/kimsehwan96/velog-jenkins-posting-react-app.git'
                 sh  'ls -al'
            }
        }
        stage('build') {
            steps {
                    dir('myapp'){
                        sh 'ls -al'
                        sh "yarn install"
                        sh "CI=false yarn build"
                }
            }
        }
        stage('deploy') {
            steps{
                    dir('myapp'){
                        sh 'ls -al'
                        sh "aws s3 sync ./build s3://velog-jenkins-deploy-posting --delete --profile default"
                        echo 'deploy done.'   
                }
            }
        }
    }
}

```

위와 같이 `aws s3 sync` 명령어를 이용해 프로덕션 빌드된 리액트 앱 파일들을 s3 버킷에 올립니다.

`s3://{여러분의 버킷명}` 으로 수정하셔서 사용하시면 되겠습니다.

이렇게 저장하고 한번 배포해보겠습니다.

![](https://images.velog.io/images/kimsehwan96/post/db17826e-a680-499e-9e9a-3bbbe6e049d1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%203.12.47.png)


![](https://images.velog.io/images/kimsehwan96/post/52cff633-7a5b-470a-8eb9-2c3f5aec0a45/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%203.13.07.png)

위 사진은 `deploy` 단계의 로그입니다. `aws s3 sync` 명령어를 통해 프로덕션 빌드된 파일들을 s3에 옮기고 있습니다.

이렇게 모두 옮겨지고나면 우리의 리액트 앱을 누구나 접근 가능합니다.

우리의 S3 버킷 `속성` 부분 맨 아래로 내려가면 이 정적 웹사이트에 접근하기 위한 링크를 확인 가능합니다.

![](https://images.velog.io/images/kimsehwan96/post/a3c16c69-f2e7-49fa-a608-0e6265e35663/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%203.14.35.png)

http://velog-jenkins-deploy-posting.s3-website.ap-northeast-2.amazonaws.com



![](https://images.velog.io/images/kimsehwan96/post/20def1a6-9113-4844-8db1-54beff21a2ad/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-29%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%203.15.30.png)

프로덕션 빌드 및 배포가 매우 잘 되었음을 확인 했습니다.

이제 여러분의 입맛에 맞게 도메인을 연결하고, https를 적용하면 됩니다.

## 마치며

이렇게 Jenkins를 활용하여 리액트 앱을 배포하는 방법을 소개했습니다.

생각보다 많이 복잡하고 어렵지만 이러한 개발 -> 빌드 -> 배포 과정 한 싸이클을 겪어보신 분들이라면 충분히 이해하고 `Jenkins`를 어떻게 활용 할 수 있을지 감이 오셨으리라 생각합니다.

여러분 모두 강력한 `Jenkins`를 통하여 CI/CD 자동화를 구축해보세요. 

다음 포스팅에서는 `깃허브 PR Merge` 이벤트가 발생할 때 마다 `Jenkins`가 자동으로 빌드하도록 설정하는 방법을 소개하겠습니다.

> 피드백은 언제나 환영입니다. :)
