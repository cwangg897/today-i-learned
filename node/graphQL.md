```js
const typeDefs = `#graphql
    type Query {
        qqq: String
    }
`

const resolvers = {
    Query: {
        qqq: () => {
            return "zxcvㅁㄴㅇㄹㅁㄴㅇㄹㅁㄴㅇㄹzxvzxㄴ"
        }
    }
}

const server = new ApolloServer({
    typeDefs: typeDefs,
    resolvers: resolvers
})
```
다음코드는 server에 typeDefs변수를 통해 스키마를 정의하고 resolvers변수릁 통해 resolver를 정희하여 아폴로서버를 구성하겠다라는 코드이다 핵심인것같다.<br>
graphQL은 query와 mutation만 존재한다.
rest의 경우는 url로 요청하지만 graphQL은 그렇지않다.
```js
const resolvers = {
    Query : {
        qqq : () => {
            return "hello world"
        }
    }
}
```
다음 코드는 qqq로 Query 요청했을때 hello World를 보내주는것이다. 

### typeDefs는 스키마를정의하는것이다(스웨거가 아니다)
typeDefs는 Apollo Server를 사용할 때 필수적인 옵션입니다. typeDefs는 GraphQL 스키마를 정의하는 부분으로, 
서버에게 사용 가능한 쿼리와 뮤테이션의 형태를 알려줍니다. 스키마는 GraphQL API의 계약(Contract)으로 볼 수 있으며, 클라이언트가 어떤 데이터를 요청할 수 있는지, 어떤 형식으로 응답을 받을 수 있는지를 결정합니다.

### cors
apollo서버는 cors가 내장되어있다. 
express는 설치해줘야했다 
key value가 동일한 경우는 생략이가능하다(js문법인듯) - short hand property


### 함수 function팁
앞에 매개변수는 지울수없지만 뒤에는 지울 수 있다.
앞에거를 안쓸려면 언더바를 많이 쓴다

### graphQL특징
graphQL은 꼭 docs를 만들어야 실행이 가능하다

깨달은점 스프링에서도 graphQL이나 다른거 사용할때 파일경로나 그런거 만들어주는데 스프링은 거기에 넣어서 사용해라 그러면 스프링이 알아서 그파일을 읽어서 실행시켜준다.
즉 프레임워크의 느낌이왔다 개발자는 프레임워크에 맞춰서 넣기만 하면 실행은 알아서 시켜준다 즉 사용법에 대해서만 알고 맞춰사용하기만 하면된다

### graphQL타입
return타입을 String, Int이런거 빼면 직접 스키마를 정의해 주어야한다
```js
{ number: 1, writer: "철수", title: "제목입니다~~", contents: "내용이에요!!!" }
애를 하나의 객체로 만들고

    type MyResult {
        number : Int
        writer : String
        title : String
        contents : String
    }

    type Query {
        fetchBoards: [MyResult] // 배열안에 하나의 객체를 의미
    }

```
