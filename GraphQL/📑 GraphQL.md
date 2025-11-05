
![](assets/Vrhv9WdPddcVDZap7iWDtP_kEtvZh0boOeX9CxBcMpg=.png)


## Схема

Схема определяет операции запроса, разрешенные клиентам, а также то, как связаны разные типы. 

Схема описывает требования к данным, но не выполняет работу по получению этих данных. 


***

## Распознаватель/Resolver

Распознаватель — это функция, которая возвращает данные для определенного поля.

Функции "Распознаватели" возвращают данные в типе и форме, заданных схемой.

Распознаватели могут быть асинхронными и могут извлекать или обновлять данные из REST API, базы данных или любого другого сервиса.

Распознаватели являются ключевыми инструментами для реализации GraphQL.

Каждое поле должно иметь соответствующую функцию распознавателя.

Распознаватель должен следовать правилам схемы — иметь то же имя, что и поле, которое было определено в схеме, и возвращать тип данных, заданный схемой.

Пример функции Распознавателя:

```java
@QueryMapping(name = "sldrProduct")
public ProductView product(@Argument final String id) {

  return this.operationsFactory
      .wrapped(id)
      .perform()
      .asObjectOf(Response.class);    
}
```


***

## Запрос

Документ запроса представляет собой строку. 

Когда мы отправляем запрос в API GraphQL, эта строка разбирается на абстрактное синтаксическое дерево и проверяется до запуска операции. 

Абстрактное синтаксическое дерево, или АСД — иерархический объект, представляющий наш запрос. 

АСД — объект, который содержит вложенные поля, представляющие детали GraphQL-запроса.

Лексирование или лексический анализ - анализ ключевых слов, аргументов, скобок и двоеточий в набор отдельных токенов.

Запрос намного проще динамически модифицировать и проверять в виде АСД.

Документ запроса GraphQL содержит хотя бы одно определение, но оно также может содержать и список определений. 

Определения могут быть только одного из двух типов: OperationDefinition или FragmentDefinition.

Ниже приведен пример документа, который содержит три определения — две операции и один фрагмент:

```graphql
# Operation Query
query jazzCatStatus {
    Lift(id: "jazz-cat") {
        name
        night
        elevationGain
        trailAccess {
            name
            difficulty
        }
    }
}

# Operation Mutation
mutation closeLift($lift: ID!) {
    setLiftStatus(id: $lift, status: CLOSED ) {
        ...liftStatus
    }
}

# Fragment
fragment liftStatus on Lift {
    name
    status
}
```

1\. Поле Lift— это SelectionSet для запроса jazzCatStatus
2\. Поле setLiftStatus представляет собой выборку для мутации closeLift.
3\. Запрос jazzCatStatus содержит 3 вложенные выборки. Первая выборка SelectionSet включает полеLift. Внутрь вложена выборка SelectionSet, которая содержит поля name, night, elevationGain и trailAccess. Ниже поля trailAccess вложена еще одна выборка SelectionSet, которая включает поля name и difficultyдля каждой трассы.


***

## OperationDefinition

OperationDefinition может содержать только один из трех типов операций: mutation, query или subscription.

Каждое определение операции содержит OperationType и SelectionSet.

Фигурные скобки, которые указываются после каждой операции, содержат выборку SelectionSet. 

SelectionSet - это фактические поля, которые мы запрашиваем вместе с их аргументами. 

GraphQL может пройтись по этому АСД и проверить его относительно языка GraphQL и текущей схемы.

Если синтаксис языка запроса правильный, а схема содержит поля и типы, которые мы запрашиваем, выполняется операция.

Если нет, вместо этого возвращается соответствующая ошибка.

Объект АСД легче модифицировать, чем строку.


***

## Восклицательный знак (!)

Везде, где указан восклицательный знак, требуется значение и нельзя вернуть null.

Если восклицательный знак указывается после закрывающей квадратной скобки, это означает, что само поле не может быть нулевым.

Если восклицательный знак указывается перед закрывающей квадратной скобкой, значения, содержащиеся в списке, не могут быть нулевыми.

![](assets/VBBDZDDm5Nhq1VdH7AAPzaSR8QJz-8KVIFyB9x5fS44=.png)


***

## Объединения

Тип объединения — это тип, который можно использовать для возврата одного из нескольких разных типов.

```graphql
query schedule {
    agenda {
        ...on Workout {
            name
            reps
        }
        ...on StudyGroup {
            name
            subject
            students
        }
    }
}
```

Вместо примера выше можно использовать объединение нескольких типов:

```graphql
union AgendaItem = StudyGroup | Workout

type StudyGroup {
    name: String!
    subject: String
    students: [User!]!
}

type Workout {
    name: String!
    reps: Int!
}

type Query {
    agenda: [AgendaItem!]!
}
```

Можно объединить столько типов, сколько нужно в рамках единого объединения.

Пример: union = StudyGroup | Workout | Class | Meal | Meeting | FreeTime


***

## Интерфейсы

Интерфейсы представляют собой абстрактные типы, которые могут быть реализованы как типы объекта.

Интерфейс определяет все поля, которые должны быть включены в любой объект, который его реализует.

```graphql
query schedule {
    agenda {
        name
        start
        end
        ...on Workout {
            reps
        }
    }
}
```

Решение рализованное посредством интерфейсов:

```graphql
scalar DataTime

interface AgendaItem {
    name: String!
    start: DateTime!
    end: DateTime!
}

type StudyGroup implements AgendaItem {
    name: String!
    start: DateTime!
    end: DateTime!
    participants: [User!]!
    topic: String!
}

type Workout implements AgendaItem {
    name: String!
    start: DateTime!
    end: DateTime!
    reps: Int!
}

type Query {
    agenda: [AgendaItem!]!
}
```

В общем случае, если объекты содержат совершенно разные поля, рекомендуется применять объединения.

Если тип объекта должен содержать определенные поля для взаимодействия с другим типом объекта, вам нужно будет использовать интерфейс, а не объединение.


***

## Типы ввода

Порой аргументы для нескольких запросов и мутаций становятся довольно длинными. Существует лучший способ организовать эти аргументы с использованием типов ввода.

Тип ввода аналогичен типу объекта GraphQL, за исключением того, что применяется только для входных аргументов.

```graphql
input PostPhotoInput {
    name: String!
    description: String
    category: PhotoCategory=PORTRAIT
}

type Mutation {
    postPhoto(input: PostPhotoInput!): Photo!
}
```

Тип PostPhotoInput похож на тип объекта, но он был создан только для входных аргументов.

Непосредственный запрос будет выглядеть так:

```graphql
mutation newPhoto($input: PostPhotoInput!) {
    postPhoto(input: $input) {
        id
        url
        created
    }
}
```

Типы ввода — это ключ к организации и написанию четкой схемы GraphQL.

Можно добавлять типы ввода как аргументы в любом поле для улучшения пагинации данных и фильтрации данных в приложениях.

Еще один пример:

```graphql
input PhotoFilter {
    category: PhotoCategory
    createdBetween: DateRange
    taggedUsers: [ID!]
    searchText: String
}

input DateRange {
    start: DateTime!
    end: DateTime!
}

input DataPage {
    first: Int = 25
    start: Int = 0
}

input DataSort {
    sort: SortDirection = DESCENDING
    sortBy: SortablePhotoField = created
}

type User {
    postedPhotos(filter:PhotoFilter paging:DataPage, sorting:DataSort): [Photo!]!
    inPhotos(filter:PhotoFilter paging:DataPage, sorting:DataSort): [Photo!]!
}

type Photo {
    taggedUsers(sorting:DataSort): [User!]!
}

type Query {
    allUsers(paging:DataPage sorting:DataSort): [User!]!
    allPhotos(filter:PhotoFilter paging:DataPage, sorting:DataSort): [Photo!]!
}
```

Типы ввода помогают упорядочивать схемы и использовать аргументы.

Они также улучшают документацию схемы, которую автоматически генерирует GraphiQL или GraphQL Playground. Это сделает API более доступным и более простым в освоении и оценке.


***

## Возвращаемые типы

Иногда нам нужно возвращать метаинформацию о запросах и мутациях в дополнение к фактическим данным полезной нагрузки. Например, когда пользователь авторизовался и прошел аутентификацию, нам нужно вернуть токен в дополнение к полезной нагрузке.


Пример:

Чтобы авторизоваться с помощью GitHub OAuth, мы должны получить код OAuth от GitHub. Предположим, что у нас есть допустимый код GitHub, который мы можем отправить мутации githubAuth для авторизации пользователя:

```graphql
type AuthPayload {
    user: User!
    token: String!
}

type Mutation {
    githubAuth(code: String!): AuthPayload!
}
```

Пользователи аутентифицируются путем отправки допустимого кода GitHub в мутацию githubAuth. 

В случае успеха вернется тип пользовательского объекта, содержащий как информацию об успешно авторизованном пользователе, так и токен, который может применяться для авторизации дополнительных запросов и мутаций.

Можно задействовать пользовательские типы возвращаемых данных в любом поле, для которого нужны более простые данные полезной нагрузки.

Возможно, необходимо знать, сколько времени требуется запросу для доставки ответа или сколько результатов было найдено в конкретном ответе в дополнение к данным полезной нагрузки запроса.


***

## Подписки

Типы Subscription не отличаются от любого другого типа объекта на языке определения схемы GraphQL. 

Здесь определяются доступные подписки как поля в пользовательском типе объектов.

Необходимо убедиться, что подписки реализуют шаблон проектирования Pub/Sub вместе с каким-либо видом транспорта реального времени. 

```graphql
type Subscription {
    newPhoto: Photo!
    newUser: User!
}

schema {
    query: Query
    mutation: Mutation
    subscription: Subscription
}
```

Здесь создается пользовательский объект Subscription, который содержит два поля: newPhoto и newUser.

Когда будет опубликовано новое фото, эта новая фотография будет передана всем клиентам, оформившим подписку newPhoto. Когда будет создан новый пользователь, его данные передадутся каждому клиенту, подписанному на оповещения о новых пользователях.

Подобно запросам или мутациям, подписки могут использовать аргументы.

```graphql
type Subscription {
    newPhoto(category: PhotoCategory): Photo!
    newUser: User!
}
```

Когда пользователи оформляют подписку newPhoto, у них появляется возможность фильтровать фотографии, которые были добавлены к этой подписке. 

Подписка — отличное решение, когда важно обрабатывать данные в реальном времени.


***

## Документация схемы

При написании схемы GraphQL можно добавить необязательные описания для каждого поля, которые предоставят дополнительную информацию о типах и полях схемы.

Предоставление описаний может облегчить вашей команде, вам и другим пользователям API понимание вашей системы типов.

```graphql
"""
A user who has been authorized by GitHub at least once
"""
type User {
    """
    The user's unique GitHub login
    """
    githubLogin: ID!
    """
    The user's first and last name
    """
    name: String
    """
    A url for the user's GitHub profile photo
    """
    avatar: String
    """
    All of the photos posted by this user
    """
    postedPhotos: [Photo!]!
    """
    All of the photos in which this user appears
    """
    inPhotos: [Photo!]!
}
```

Добавив три кавычки выше и ниже вашего комментария к каждому типу или полю,пользователю предоставляется словарь для API.

Помимо типов и полей, также можно документировать аргументы:

```graphql
type Mutation {
    """
    Authorizes a GitHub User
    """
    githubAuth(
        "The unique code from GitHub that is sent to authorize the user"
        code: String!
    ): AuthPayload!
}
```

Комментарии аргументов предоставляют имя аргумента и информацию о том, не является ли поле необязательным.

Если используются типы ввода, можно документировать их, как и любой другой тип:

```graphql
"""
The inputs sent with the postPhoto Mutation
"""
input PostPhotoInput {
    "The name of the new photo"
    name: String!
    "(optional) A brief description of the photo"
    description: String
    "(optional) The category that defines the photo"
    category: PhotoCategory=PORTRAIT
}

postPhoto(
"input: The name, description, and category for a new photo"
input: PostPhotoInput!
): Photo!
```

Все эти комментарии затем перечисляются в документации схемы GraphQL Playground или GraphiQL, как показано на рисунке ниже.

![](assets/xXmYkdy-jWeOTYJhZHQmi2BFi53BAT9lGvPPVND_qBM=.png)

