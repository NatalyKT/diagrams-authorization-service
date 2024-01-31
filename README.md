### Тестовое задание на проектирование.
Требуется описать сервис авторизации с разными типами входных данных.
Данный сервис реализует поиск пользователей, проверку авторизационных данных, вызов методов создания и получения данных пользователя. 

[Полный текст задания](https://docs.google.com/document/d/1AB2b8yiZ3sNgbkdMpDuEq2hr2H4xuQIRabNuO6oOlOY/) 

#### Выполнение:

1. Диаграмма таблиц баз данных

Для создания и описания структуры базы данных, включая таблицы и их взаимосвязи, используется ER-диаграмма (зд. - нотация Мартина): 

```mermaid
erDiagram
  USER {
    int id
    string firstName
    string lastName
  }
  AUTHDATA {
    int id
    string login
    string password
    int code
    string authType
  }
  AUTHORIZATIONMETHOD {
    int id
    string methodName
  }
  OAUTHAUTHORIZATION {
    int id
    string provider
    string accessToken
  }
  USER ||--o{ AUTHDATA : "Has"
  AUTHDATA ||--o| AUTHORIZATIONMETHOD : "Uses"
  AUTHDATA ||--o| OAUTHAUTHORIZATION : "Uses"
```

Сущность USER связана с AUTHDATA отношением "Has", что указывает, что пользователь может иметь несколько записей в AUTHDATA.
AUTHDATA связана с AUTHORIZATIONMETHOD и OAUTHAUTHORIZATION отношениями "Uses", указывающими на использование этих методов авторизации.



2. UML-диаграмма классов
```mermaid
classDiagram
  class User {
    +id: number
    +firstName?: string
    +lastName?: string
  }

  class AuthData {
    +id: number
    +userId: number
    +authType: string
    +credential: string
  }

  class Session {
    +userId: number
    +sessionId: string
  }

  class Code {
    +id: number
    +login: string
    +code: string
  }

  class UserAggregate {
    +user: User
    +authDataList: AuthData[]
    +sessionList: Session[]
  }

  class AuthController {
    +entryPoint(login: string, authType: string): UserAggregate
    +login(login: string, code: string, authType: string): UserAggregate
  }

  class SessionService {
    +Auth(user: UserAggregate): void
  }

  class UserService {
    +GetById(id: number): UserAggregate | null
    +Create(id: number): UserAggregate
    +CreateUserWithAuthData(id: number, authType: string, credential: string): UserAggregate
    +GetUserAggregate(user: User): UserAggregate
  }

  class CodeService {
    +Send(login: string, authType: string): AuthData
  }

  class UserFactory {
    +CreateUser(id: number): UserAggregate
  }

  AuthController --> UserService
  AuthController --> SessionService
  AuthController --> CodeService
  UserService --> UserFactory
  CodeService --> AuthData
  UserService --> UserAggregate
  AuthController --> UserAggregate

```

В данной диаграммеп к основным связям добавлены дополнительные:
`CodeService --> AuthData`: Связь между `CodeService` и `AuthData`.
`UserService --> UserAggregate`: Связь между `UserService` и `UserAggregate`.
`AuthController --> UserAggregate`: Связь между `AuthController` и `UserAggregate`.


Данные связи не являются обязательными, и добавлены исключительно для примера, т.к. конкретное количество связей будет зависеть от потребностей проекта, а слишком много связей могут излишне усложнять систему. 
К примеру, в данной диаграмме отсутствует связь `User --|> Session`, потому что здесь сессии ассоциированы с пользователями через `UserAggregate`, который уже включает в себя пользователя и его сессии, и эта связь не является необходимой.



3. UML-диаграмма последовательности (стек вызова методов). 
Отражает последовательность действий при входе и аутентификации пользователя, включая отправку кода, проверку и создание пользователя, а также процесс аутентификации через сессии.


```mermaid
sequenceDiagram
  participant Frontend
  participant AuthController
  participant CodeService
  participant SessionService
  participant UserService

  Frontend ->> AuthController: entryPoint(login)
  AuthController ->> CodeService: Send(login)
  CodeService -->> Frontend: Code Sent

  Frontend ->> AuthController: login(login, credential)
  AuthController ->> UserService: GetById(id)
  UserService -->> AuthController: User Found
  AuthController ->> CodeService: Send(login)
  CodeService -->> Frontend: Code Sent
  Frontend ->> AuthController: Code Verification
  AuthController ->> UserService: Create(id)
  UserService -->> AuthController: User Created
  AuthController ->> SessionService: Auth(user)
  SessionService -->> Frontend: Authenticated
```

> В файле README.md использован язык описания диаграмм [Mermaid](https://mermaid.js.org/).
