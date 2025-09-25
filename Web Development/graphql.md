# GraphQL - Query Language for APIs

GraphQL is a query language and runtime for APIs that provides a complete and understandable description of the data in your API, giving clients the power to ask for exactly what they need and nothing more.

---

## What is GraphQL?

GraphQL is a:
- **Query Language**: Allows clients to request exactly the data they need
- **Runtime**: Executes queries against your data using a type system you define
- **API Architecture**: Provides a single endpoint for all your data needs
- **Strongly Typed**: Schema defines the shape of your data and available operations

### Key Benefits
- **Precise Data Fetching**: Request only the fields you need
- **Single Endpoint**: One URL for all operations (queries, mutations, subscriptions)
- **Strong Typing**: Schema serves as documentation and validation
- **Real-time Updates**: Built-in subscription support
- **Version-free**: Evolve your API without breaking changes

---

## Core Concepts

### Schema Definition Language (SDL)
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  published: Boolean!
}

type Query {
  user(id: ID!): User
  users: [User!]!
  post(id: ID!): Post
  posts: [Post!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

type Subscription {
  userCreated: User!
  postPublished: Post!
}

input CreateUserInput {
  name: String!
  email: String!
}

input UpdateUserInput {
  name: String
  email: String
}
```

### Scalar Types
- **String**: Text data
- **Int**: 32-bit integer
- **Float**: Floating point number
- **Boolean**: true/false
- **ID**: Unique identifier (serialized as String)

### Custom Scalars
```graphql
scalar DateTime
scalar JSON
scalar Upload
```

---

## Queries

### Basic Query
```graphql
query GetUsers {
  users {
    id
    name
    email
  }
}
```

### Query with Arguments
```graphql
query GetUser($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
    posts {
      id
      title
      published
    }
  }
}
```

### Query with Fragments
```graphql
fragment UserInfo on User {
  id
  name
  email
  createdAt
}

query GetUsersWithPosts {
  users {
    ...UserInfo
    posts {
      id
      title
    }
  }
}
```

### Query with Aliases
```graphql
query GetUserData($userId: ID!) {
  user: user(id: $userId) {
    name
    email
  }
  recentPosts: posts(first: 5) {
    id
    title
  }
}
```

### Query with Directives
```graphql
query GetUser($userId: ID!, $includePosts: Boolean!) {
  user(id: $userId) {
    id
    name
    email
    posts @include(if: $includePosts) {
      id
      title
    }
  }
}
```

---

## Mutations

### Basic Mutation
```graphql
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
    email
    createdAt
  }
}
```

### Mutation with Error Handling
```graphql
mutation UpdateUser($id: ID!, $input: UpdateUserInput!) {
  updateUser(id: $id, input: $input) {
    user {
      id
      name
      email
    }
    errors {
      field
      message
    }
  }
}
```

### Multiple Mutations
```graphql
mutation CreateUserAndPost($userInput: CreateUserInput!, $postInput: CreatePostInput!) {
  user: createUser(input: $userInput) {
    id
    name
  }
  post: createPost(input: $postInput) {
    id
    title
  }
}
```

---

## Subscriptions

### Real-time Updates
```graphql
subscription UserCreated {
  userCreated {
    id
    name
    email
    createdAt
  }
}
```

### Subscription with Filters
```graphql
subscription PostPublished($authorId: ID!) {
  postPublished(authorId: $authorId) {
    id
    title
    content
    author {
      name
    }
  }
}
```

---

## Server Implementation

### Apollo Server (Node.js)
```javascript
import { ApolloServer, gql } from 'apollo-server';
import { buildSubgraphSchema } from '@apollo/subgraph';

const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
  }

  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
  }

  type Query {
    user(id: ID!): User
    users: [User!]!
    post(id: ID!): Post
    posts: [Post!]!
  }

  type Mutation {
    createUser(input: CreateUserInput!): User!
    createPost(input: CreatePostInput!): Post!
  }

  input CreateUserInput {
    name: String!
    email: String!
  }

  input CreatePostInput {
    title: String!
    content: String!
    authorId: ID!
  }
`;

const resolvers = {
  Query: {
    user: async (_, { id }, { dataSources }) => {
      return dataSources.users.getUser(id);
    },
    users: async (_, __, { dataSources }) => {
      return dataSources.users.getUsers();
    },
    post: async (_, { id }, { dataSources }) => {
      return dataSources.posts.getPost(id);
    },
    posts: async (_, __, { dataSources }) => {
      return dataSources.posts.getPosts();
    }
  },
  Mutation: {
    createUser: async (_, { input }, { dataSources }) => {
      return dataSources.users.createUser(input);
    },
    createPost: async (_, { input }, { dataSources }) => {
      return dataSources.posts.createPost(input);
    }
  },
  User: {
    posts: async (user, _, { dataSources }) => {
      return dataSources.posts.getPostsByAuthor(user.id);
    }
  },
  Post: {
    author: async (post, _, { dataSources }) => {
      return dataSources.users.getUser(post.authorId);
    }
  }
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  dataSources: () => ({
    users: new UserAPI(),
    posts: new PostAPI()
  }),
  context: ({ req }) => {
    const token = req.headers.authorization || '';
    return { token };
  }
});

server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`);
});
```

### Data Sources
```javascript
import { RESTDataSource } from 'apollo-datasource-rest';

class UserAPI extends RESTDataSource {
  constructor() {
    super();
    this.baseURL = 'https://api.example.com/';
  }

  async getUser(id) {
    return this.get(`users/${id}`);
  }

  async getUsers() {
    return this.get('users');
  }

  async createUser(input) {
    return this.post('users', input);
  }
}

class PostAPI extends RESTDataSource {
  constructor() {
    super();
    this.baseURL = 'https://api.example.com/';
  }

  async getPost(id) {
    return this.get(`posts/${id}`);
  }

  async getPosts() {
    return this.get('posts');
  }

  async getPostsByAuthor(authorId) {
    return this.get(`posts?authorId=${authorId}`);
  }

  async createPost(input) {
    return this.post('posts', input);
  }
}
```

### Database Integration (Prisma)
```javascript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

const resolvers = {
  Query: {
    user: async (_, { id }) => {
      return prisma.user.findUnique({
        where: { id },
        include: { posts: true }
      });
    },
    users: async () => {
      return prisma.user.findMany({
        include: { posts: true }
      });
    }
  },
  Mutation: {
    createUser: async (_, { input }) => {
      return prisma.user.create({
        data: input,
        include: { posts: true }
      });
    }
  }
};
```

---

## Client Implementation (React + Apollo)

### Setup
```bash
npm install @apollo/client graphql
```

### Apollo Client Configuration
```javascript
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';

const httpLink = createHttpLink({
  uri: 'http://localhost:4000/graphql',
});

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('token');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : "",
    }
  }
});

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache(),
  defaultOptions: {
    watchQuery: {
      errorPolicy: 'all',
    },
    query: {
      errorPolicy: 'all',
    },
  },
});

export default client;
```

### React Integration
```javascript
import React from 'react';
import { ApolloProvider } from '@apollo/client';
import client from './apollo-client';
import App from './App';

function Root() {
  return (
    <ApolloProvider client={client}>
      <App />
    </ApolloProvider>
  );
}

export default Root;
```

### Query Hook
```javascript
import { useQuery, gql } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
      email
      posts {
        id
        title
      }
    }
  }
`;

function UsersList() {
  const { loading, error, data } = useQuery(GET_USERS);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <h2>Users</h2>
      {data.users.map(user => (
        <div key={user.id}>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
          <p>Posts: {user.posts.length}</p>
        </div>
      ))}
    </div>
  );
}
```

### Mutation Hook
```javascript
import { useMutation, gql } from '@apollo/client';

const CREATE_USER = gql`
  mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
      email
    }
  }
`;

function CreateUserForm() {
  const [createUser, { loading, error }] = useMutation(CREATE_USER);

  const handleSubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    
    try {
      const { data } = await createUser({
        variables: {
          input: {
            name: formData.get('name'),
            email: formData.get('email')
          }
        }
      });
      console.log('User created:', data.createUser);
    } catch (err) {
      console.error('Error creating user:', err);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" placeholder="Name" required />
      <input name="email" type="email" placeholder="Email" required />
      <button type="submit" disabled={loading}>
        {loading ? 'Creating...' : 'Create User'}
      </button>
      {error && <p>Error: {error.message}</p>}
    </form>
  );
}
```

### Subscription Hook
```javascript
import { useSubscription, gql } from '@apollo/client';

const USER_CREATED = gql`
  subscription UserCreated {
    userCreated {
      id
      name
      email
      createdAt
    }
  }
`;

function UserNotifications() {
  const { data, loading } = useSubscription(USER_CREATED);

  if (loading) return <p>Listening for new users...</p>;

  return (
    <div>
      <h3>New User Created!</h3>
      <p>Name: {data?.userCreated.name}</p>
      <p>Email: {data?.userCreated.email}</p>
    </div>
  );
}
```

---

## Advanced Features

### Caching Strategies
```javascript
import { InMemoryCache } from '@apollo/client';

const cache = new InMemoryCache({
  typePolicies: {
    User: {
      fields: {
        posts: {
          merge(existing = [], incoming) {
            return [...existing, ...incoming];
          }
        }
      }
    },
    Query: {
      fields: {
        users: {
          merge(existing = [], incoming) {
            return [...existing, ...incoming];
          }
        }
      }
    }
  }
});
```

### Optimistic Updates
```javascript
const UPDATE_USER = gql`
  mutation UpdateUser($id: ID!, $input: UpdateUserInput!) {
    updateUser(id: $id, input: $input) {
      id
      name
      email
    }
  }
`;

function UserProfile({ user }) {
  const [updateUser] = useMutation(UPDATE_USER);

  const handleUpdate = async (input) => {
    await updateUser({
      variables: { id: user.id, input },
      optimisticResponse: {
        updateUser: {
          __typename: 'User',
          id: user.id,
          ...input
        }
      }
    });
  };

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={() => handleUpdate({ name: 'New Name' })}>
        Update Name
      </button>
    </div>
  );
}
```

### Error Handling
```javascript
import { ApolloError } from '@apollo/client';

function ErrorBoundary({ children }) {
  const [error, setError] = useState(null);

  if (error) {
    return (
      <div>
        <h2>Something went wrong!</h2>
        <p>{error.message}</p>
        <button onClick={() => setError(null)}>Try again</button>
      </div>
    );
  }

  return children;
}

function UsersList() {
  const { loading, error, data } = useQuery(GET_USERS, {
    errorPolicy: 'all',
    onError: (error) => {
      console.error('GraphQL Error:', error);
    }
  });

  if (error) {
    return (
      <div>
        <p>Error: {error.message}</p>
        {error.graphQLErrors.map(({ message }, index) => (
          <p key={index}>GraphQL Error: {message}</p>
        ))}
        {error.networkError && (
          <p>Network Error: {error.networkError.message}</p>
        )}
      </div>
    );
  }

  // ... rest of component
}
```

### Pagination
```graphql
type Query {
  posts(first: Int, after: String): PostConnection!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

```javascript
import { useQuery } from '@apollo/client';

const GET_POSTS = gql`
  query GetPosts($first: Int, $after: String) {
    posts(first: $first, after: $after) {
      edges {
        node {
          id
          title
          content
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

function PostsList() {
  const { data, loading, fetchMore } = useQuery(GET_POSTS, {
    variables: { first: 10 }
  });

  const loadMore = () => {
    fetchMore({
      variables: {
        after: data.posts.pageInfo.endCursor
      }
    });
  };

  if (loading) return <p>Loading...</p>;

  return (
    <div>
      {data.posts.edges.map(({ node }) => (
        <div key={node.id}>
          <h3>{node.title}</h3>
          <p>{node.content}</p>
        </div>
      ))}
      {data.posts.pageInfo.hasNextPage && (
        <button onClick={loadMore}>Load More</button>
      )}
    </div>
  );
}
```

---

## Testing

### Server Testing
```javascript
import { createTestClient } from 'apollo-server-testing';
import { ApolloServer } from 'apollo-server';
import { typeDefs, resolvers } from './schema';

const server = new ApolloServer({ typeDefs, resolvers });
const { query, mutate } = createTestClient(server);

describe('User Queries', () => {
  test('should fetch users', async () => {
    const GET_USERS = gql`
      query GetUsers {
        users {
          id
          name
          email
        }
      }
    `;

    const result = await query({ query: GET_USERS });
    expect(result.data.users).toBeDefined();
    expect(result.data.users.length).toBeGreaterThan(0);
  });

  test('should create user', async () => {
    const CREATE_USER = gql`
      mutation CreateUser($input: CreateUserInput!) {
        createUser(input: $input) {
          id
          name
          email
        }
      }
    `;

    const result = await mutate({
      query: CREATE_USER,
      variables: {
        input: {
          name: 'John Doe',
          email: 'john@example.com'
        }
      }
    });

    expect(result.data.createUser).toBeDefined();
    expect(result.data.createUser.name).toBe('John Doe');
  });
});
```

### Client Testing
```javascript
import { MockedProvider } from '@apollo/client/testing';
import { render, screen } from '@testing-library/react';
import { GET_USERS } from './queries';
import UsersList from './UsersList';

const mocks = [
  {
    request: {
      query: GET_USERS,
    },
    result: {
      data: {
        users: [
          {
            id: '1',
            name: 'John Doe',
            email: 'john@example.com',
            posts: []
          }
        ]
      }
    }
  }
];

test('renders users list', async () => {
  render(
    <MockedProvider mocks={mocks} addTypename={false}>
      <UsersList />
    </MockedProvider>
  );

  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

---

## Performance Optimization

### Query Optimization
```javascript
// Use specific field selection
const GET_USER_BASIC = gql`
  query GetUserBasic($id: ID!) {
    user(id: $id) {
      id
      name
    }
  }
`;

// Use fragments for reusable selections
const USER_FRAGMENT = gql`
  fragment UserInfo on User {
    id
    name
    email
  }
`;

const GET_USERS_WITH_FRAGMENT = gql`
  query GetUsers {
    users {
      ...UserInfo
    }
  }
  ${USER_FRAGMENT}
`;
```

### DataLoader for N+1 Problem
```javascript
import DataLoader from 'dataloader';

const userLoader = new DataLoader(async (ids) => {
  const users = await prisma.user.findMany({
    where: { id: { in: ids } }
  });
  return ids.map(id => users.find(user => user.id === id));
});

const resolvers = {
  Post: {
    author: async (post, _, { userLoader }) => {
      return userLoader.load(post.authorId);
    }
  }
};
```

### Query Complexity Analysis
```javascript
import { createComplexityLimitRule } from 'graphql-query-complexity';

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [
    createComplexityLimitRule(1000, {
      onCost: (cost) => {
        console.log('Query cost:', cost);
      }
    })
  ]
});
```

---

## Security

### Authentication & Authorization
```javascript
import jwt from 'jsonwebtoken';

const resolvers = {
  Query: {
    user: async (_, { id }, { user }) => {
      if (!user) throw new AuthenticationError('Must be logged in');
      return getUser(id);
    }
  },
  Mutation: {
    createPost: async (_, { input }, { user }) => {
      if (!user) throw new AuthenticationError('Must be logged in');
      if (user.role !== 'ADMIN') throw new ForbiddenError('Admin access required');
      return createPost(input);
    }
  }
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    const token = req.headers.authorization?.replace('Bearer ', '');
    if (token) {
      try {
        const user = jwt.verify(token, process.env.JWT_SECRET);
        return { user };
      } catch (error) {
        throw new AuthenticationError('Invalid token');
      }
    }
    return {};
  }
});
```

### Input Validation
```javascript
import { GraphQLScalarType } from 'graphql';
import validator from 'validator';

const EmailType = new GraphQLScalarType({
  name: 'Email',
  serialize: (value) => value,
  parseValue: (value) => {
    if (!validator.isEmail(value)) {
      throw new Error('Invalid email format');
    }
    return value;
  },
  parseLiteral: (ast) => {
    if (ast.kind !== Kind.STRING) {
      throw new Error('Email must be a string');
    }
    if (!validator.isEmail(ast.value)) {
      throw new Error('Invalid email format');
    }
    return ast.value;
  }
});
```

### Rate Limiting
```javascript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});

app.use('/graphql', limiter);
```

---

## Best Practices

### Schema Design
- **Use descriptive names**: `getUserById` instead of `getUser`
- **Consistent naming**: Use camelCase for fields, PascalCase for types
- **Version your schema**: Use schema evolution instead of versioning
- **Document everything**: Add descriptions to types and fields

### Query Design
- **Be specific**: Request only the fields you need
- **Use fragments**: Reuse common field selections
- **Handle errors**: Implement proper error handling
- **Optimize for mobile**: Consider data usage and performance

### Server Implementation
- **Use DataLoaders**: Prevent N+1 query problems
- **Implement caching**: Cache frequently accessed data
- **Monitor performance**: Track query execution times
- **Validate inputs**: Always validate and sanitize inputs

### Client Implementation
- **Use TypeScript**: Get type safety and better DX
- **Implement error boundaries**: Handle GraphQL errors gracefully
- **Cache strategically**: Use Apollo Client's caching features
- **Optimize re-renders**: Use React.memo and useMemo appropriately

---

## Common Patterns

### Relay-Style Pagination
```graphql
type Query {
  posts(first: Int, after: String): PostConnection!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type PostEdge {
  node: Post!
  cursor: String!
}
```

### Union Types
```graphql
union SearchResult = User | Post | Comment

type Query {
  search(query: String!): [SearchResult!]!
}
```

### Interface Types
```graphql
interface Node {
  id: ID!
}

type User implements Node {
  id: ID!
  name: String!
}

type Post implements Node {
  id: ID!
  title: String!
}
```

---

## Tools & Ecosystem

### Development Tools
- **GraphQL Playground**: Interactive GraphQL IDE
- **Apollo Studio**: Schema management and analytics
- **GraphQL Code Generator**: Generate TypeScript types
- **GraphQL Inspector**: Schema validation and diffing

### Popular Libraries
- **Apollo Server**: GraphQL server for Node.js
- **Apollo Client**: GraphQL client for React
- **Prisma**: Database toolkit with GraphQL support
- **Hasura**: Instant GraphQL API over PostgreSQL

### Code Generation
```bash
npm install -g @graphql-codegen/cli
npm install @graphql-codegen/typescript @graphql-codegen/typescript-operations
```

```yaml
# codegen.yml
schema: http://localhost:4000/graphql
documents: 'src/**/*.graphql'
generates:
  src/generated/graphql.ts:
    plugins:
      - typescript
      - typescript-operations
```

---

## Migration from REST

### Gradual Migration
1. **Add GraphQL layer**: Create GraphQL API alongside REST
2. **Migrate clients**: Move clients to GraphQL one by one
3. **Deprecate REST**: Gradually phase out REST endpoints

### Schema-First Approach
```javascript
// Start with schema definition
const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
  }
`;

// Implement resolvers that call existing REST APIs
const resolvers = {
  Query: {
    user: async (_, { id }) => {
      const response = await fetch(`/api/users/${id}`);
      return response.json();
    }
  }
};
```

---

## Resources & Learning

### Official Documentation
- **GraphQL.org**: https://graphql.org/
- **Apollo Docs**: https://www.apollographql.com/docs/
- **Prisma Docs**: https://www.prisma.io/docs/

### Learning Path
1. **Schema Design**: Learn SDL and type system
2. **Queries & Mutations**: Master data fetching and updates
3. **Server Implementation**: Build GraphQL servers
4. **Client Integration**: Connect frontend applications
5. **Advanced Features**: Subscriptions, caching, optimization

### Community Resources
- **GraphQL Weekly**: Newsletter with latest updates
- **GraphQL Conf**: Annual conference
- **GitHub**: Explore open-source GraphQL projects
- **Discord/Slack**: Join GraphQL communities

GraphQL provides a powerful and flexible way to build APIs that can evolve with your application's needs. Start with simple queries and gradually add more complex features as you become comfortable with the ecosystem.
