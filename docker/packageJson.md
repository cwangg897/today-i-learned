### 도커파일 캐시
docker build -t hello . 치면서 같은 내용이 있다면 캐시를 해서 시간을 줄이는것을 보았다.
```
FROM node:14

COPY . /myfolder

WORKDIR /myfolder
RUN yarn install 

CMD yarn start:dev
```

### 비효율적을 효율적으로
만약에, `index.js` 파일의 소스코드를 일부 수정했다고 가정해봅시다. 
이 경우 `COPY . /myfolder/` 아래의 명령어들이 모두 새로 build되어 변경된 것이 없는 모든 모듈들 또한 `RUN yarn install` 명령어를 통해 다시 설치되게 됩니다.
`index.js` 파일만 수정했지, `package.json` 내용은 건드리지 않았는데
다시 설치를 모두 한다는게 매우 비효율적입니다.


Docker는 `Dockerfile` 을 보면서 명령어를 한줄씩 실행하다가 
기존 내용에서 변경된 점이 없으면 새로 build 하지 않고 먼저 생성된 캐시 부분을 가져와 사용하고, 
아니라면 그 부분부터 새로 build 하게 됩니다.
따라서 이러한 문제점을 해결하기 위해 지금부터 `Dockerfile` 을 변경해 보겠습니다.
모든 소스 코드를 복사하기 전에 먼저, `package.json` 와 `yarn.lock` 을 복사하도록 적어줍니다. 
`COPY ./package.json /myfolder/`
`COPY ./yarn.lock /myfolder/`

### 효율적으로 변경
즉 변경이 없으면 캐시를 사용하는것
```
# 컴퓨터 만드는 설명서

# 1. 운영체제 설치(node 14버전과 npm과 yarn이 모두 설치되어있는 리눅스)
FROM node:14

# 2. 내 컴퓨터에 있는 폴더나 파일을 도커 컴퓨터 안으로 복사하기
COPY ./package.json /myfolder/
COPY ./yarn.lock /myfolder/
WORKDIR /myfolder/
RUN yarn install

COPY . /myfolder/

# 3. 도커안에서 index.js 실행시키기
CMD yarn start:dev
```
