### 도커 컴포즈
등장한 이유 여러개의 이미지를 빌드하기가 귀찮다
그리고 하나하나 run하는것도 귀찮다. 그래서 등장했다.
왜 등장했는지 좀 다르게 보면은 현재 디렉토리에 DockerFile하나랑 DockerFile-mongo 하나 이렇게 2개가 존재할경우 Docker build . 을 한다면<br>
DockerFile밖에 빌드가 되지않는다 그래서 Docker build -f DockerFile-mongo . 이렇게 명시적으로 정해줘야한다. <br>
-f 옵션은 이 옵션은 빌드할 Docker 이미지를 생성할 때 사용할 Dockerfile의 경로 또는 이름을 지정하는 데 사용됩니다.
기본적으로 docker build 명령은 현재 디렉토리에서 Dockerfile을 찾아 빌드합니다. 그러나 -f 옵션을 사용하면 다른 경로에 있는 Dockerfile을 명시적으로 지정할 수 있습니다.
즉 명시적으로 빌드 하겠다라는것이다.


### 결론은 그룹핑하는게 중요
결론은 한번에 실행 한번에 종료를 위해서가 더 등장한 이유다.


```
version: '3.7' # compose파일 형식의 버전 버전마다 구문 및 기능이 다름


services:
  my-backend:
    build: # image나 build나 둘다 이미지 만드는것이고 이미지를 하면 이미지를 실행시키고 빌드를하면 이미지를 만들고 실행
      context : . # 빌드할 파일 경로
      dockerfile : Dockerfile # 빌드할 파일명
    ports : 4000:3000

  my-database:
    build:
      context :  .
      docekrfile : Dockerfile-mongo
    ports: 27017:27017
```
