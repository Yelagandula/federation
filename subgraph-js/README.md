# `Apollo Subgraph`

This package provides utilities for creating GraphQL microservices, which can be combined into a single endpoint through tools like [Apollo Gateway](https://github.com/apollographql/federation/tree/main/gateway-js).

For complete documentation, see the [Apollo Subgraph API reference](https://www.apollographql.com/docs/federation/subgraphs/).

## Usage

```js
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';
import { gql } from 'graphql-tag';
import { buildSubgraphSchema } from '@apollo/subgraph';

const typeDefs = gql`
  type Query {
    me: User
  }

  type User @key(fields: "id") {
    id: ID!
    username: String
  }
`;

const resolvers = {
  Query: {
    me() {
      return { id: "1", username: "@ava" }
    }
  },
  User: {
    __resolveReference(user, { fetchUserById }){
      return fetchUserById(user.id)
    }
  }
};

const server = new ApolloServer({
  schema: buildSubgraphSchema([{ typeDefs, resolvers }])
});

// Note the top-level await!
const { url } = await startStandaloneServer(server);
console.log(`🚀  Server ready at ${url}`);
```

## Usage with Kotlin (Spring Boot)

If you are building a subgraph using the JVM ecosystem, you can use the [`federation-jvm`](https://github.com/apollographql/federation-jvm) library with Spring Boot:

**build.gradle.kts**
```kotlin
dependencies {
    implementation("com.apollographql.federation:federation-graphql-java-support:4.1.0")
    implementation("org.springframework.boot:spring-boot-starter-graphql")
    implementation("org.springframework.boot:spring-boot-starter-web")
}
```

**schema.graphqls**
```graphql
extend schema
  @link(url: "https://specs.apollo.dev/federation/v2.0",
        import: ["@key", "@shareable"])

type Query {
  product(id: ID!): Product
}

type Product @key(fields: "id") {
  id: ID!
  name: String!
  price: Float!
}
```

**ProductResolver.kt**
```kotlin
@Controller
class ProductResolver(private val productRepository: ProductRepository) {

    @QueryMapping
    fun product(@Argument id: String): Product? {
        return productRepository.findById(id)
    }

    @SchemaMapping(typeName = "Product", field = "__resolveReference")
    fun resolveReference(reference: Map<String, Any>): Product? {
        return productRepository.findById(reference["id"] as String)
    }
}
```

For complete documentation, see the [federation-jvm README](https://github.com/apollographql/federation-jvm).
```

---

Then:

1. Scroll down → select **"Create a new branch"**
2. Name it:
```
docs/add-kotlin-spring-boot-example
```
3. Click **"Propose changes"**

**Commit message:**
```
docs: add Kotlin Spring Boot subgraph example to README
