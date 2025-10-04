# PureAgile AWS Architecture Documentation

## Overview
PureAgile is a comprehensive agile project management platform with real-time collaboration, AI-powered features, and extensive integrations. Built with GraphQL API, MongoDB database, and modern web technologies for optimal performance and scalability.

## System Architecture Components

### 1. Frontend Layer (Next.js 14.1.0 with GraphQL)
- **Service**: AWS ECS Fargate or AWS App Runner
- **Load Balancer**: Application Load Balancer (ALB)
- **CDN**: CloudFront for static assets
- **Domain**: Route 53 for DNS management
- **SSL**: AWS Certificate Manager
- **GraphQL Client**: Apollo Client for state management and caching

### 2. Backend Layer (NestJS with GraphQL)
- **Service**: AWS ECS Fargate with Auto Scaling
- **API**: GraphQL endpoint with Apollo Server
- **Load Balancer**: Application Load Balancer (ALB)
- **Container Registry**: Amazon ECR
- **Service Discovery**: AWS Cloud Map
- **Real-time**: GraphQL Subscriptions over WebSocket

### 3. Database Layer
- **Primary Database**: Amazon DocumentDB (MongoDB-compatible)
- **Database Clusters**: Multi-AZ with read replicas
- **Cache Layer**: Amazon ElastiCache Redis (Cluster Mode)
- **Search Engine**: Amazon OpenSearch Service
- **Vector Storage**: MongoDB Atlas Search (for AI embeddings)

### 4. GraphQL Architecture
- **Schema Design**: Code-first approach with decorators
- **Query Optimization**: DataLoader pattern for N+1 problem
- **Caching**: Apollo Server caching + Redis
- **Subscriptions**: Real-time updates via WebSocket
- **Federation**: Micro-services with Apollo Gateway

### 5. Authentication & Security
- **Identity Management**: Amazon Cognito with GraphQL integration
- **API Gateway**: AWS API Gateway with GraphQL support
- **Authorization**: RBAC with GraphQL field-level security
- **Secrets Management**: AWS Secrets Manager
- **Security Groups**: VPC Security Groups for network isolation

### 6. File Storage & Media
- **Object Storage**: Amazon S3 with CloudFront
- **Image Processing**: AWS Lambda with Sharp
- **File Uploads**: S3 Direct Upload with GraphQL mutations
- **CDN Integration**: Optimized asset delivery

### 7. AI & Machine Learning Services
- **AI Providers Integration**: 
  - Amazon Bedrock for Claude integration
  - OpenAI API via Lambda functions
  - SageMaker for custom model hosting
- **Vector Embeddings**: MongoDB Atlas Vector Search
- **AI Processing**: Asynchronous GraphQL mutations

### 8. Message Processing & Real-time Features
- **Message Queue**: Amazon SQS
- **Background Jobs**: AWS Batch or ECS Tasks
- **Event Processing**: Amazon EventBridge
- **Real-time Communication**: GraphQL Subscriptions + Redis PubSub
- **WebSocket Management**: AWS API Gateway WebSocket API

### 9. External Integrations
- **Webhook Processing**: API Gateway + Lambda with GraphQL mutations
- **Third-party APIs**: 
  - Slack SDK integration via GraphQL resolvers
  - Jira API synchronization
  - GitHub integration
  - Microsoft Teams webhooks
  - Email service via Amazon SES

### 10. Monitoring & Observability
- **GraphQL Monitoring**: Apollo Studio integration
- **Application Monitoring**: AWS X-Ray
- **Metrics**: CloudWatch Metrics and Dashboards
- **Query Performance**: GraphQL query analysis and optimization
- **Error Tracking**: AWS X-Ray Error Analytics with GraphQL context

### 11. CI/CD Pipeline
- **Source Control**: GitHub integration
- **Build Pipeline**: AWS CodePipeline + CodeBuild
- **GraphQL Schema Registry**: Apollo Studio schema management
- **Container Registry**: Amazon ECR
- **Deployment**: AWS CodeDeploy with Blue/Green deployments

## Detailed Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                    AWS CLOUD                                        │
│                                                                                     │
│  ┌─────────────────┐    ┌──────────────────────────────────────────────────────┐   │
│  │   Route 53      │    │                  CloudFront CDN                      │   │
│  │   DNS Service   │────│         (Static Assets & GraphQL Caching)           │   │
│  └─────────────────┘    └──────────────────────┬───────────────────────────────┘   │
│                                                │                                   │
│  ┌─────────────────────────────────────────────┼───────────────────────────────┐   │
│  │                        VPC (10.0.0.0/16)    │                               │   │
│  │                                              │                               │   │
│  │  ┌───────────────────────────────────────────▼─────────────────────────────┐ │   │
│  │  │                    Public Subnets                                       │ │   │
│  │  │                                                                         │ │   │
│  │  │  ┌─────────────────┐              ┌─────────────────┐                   │ │   │
│  │  │  │  ALB (Frontend) │              │   ALB (GraphQL  │                   │ │   │
│  │  │  │  Port 443/80    │              │   Gateway)      │                   │ │   │
│  │  │  └─────────┬───────┘              └─────────┬───────┘                   │ │   │
│  │  │            │                                │                           │ │   │
│  │  └────────────┼────────────────────────────────┼───────────────────────────┘ │   │
│  │               │                                │                             │   │
│  │  ┌────────────┼────────────────────────────────┼───────────────────────────┐ │   │
│  │  │            │         Private Subnets        │                           │ │   │
│  │  │            │                                │                           │ │   │
│  │  │  ┌─────────▼───────┐              ┌─────────▼───────┐                   │ │   │
│  │  │  │   ECS Fargate   │              │   ECS Fargate   │                   │ │   │
│  │  │  │   (Frontend)    │              │   (GraphQL API) │                   │ │   │
│  │  │  │ Next.js + Apollo│              │ NestJS + Apollo │                   │ │   │
│  │  │  │   Port 3000     │◄─────────────┤   Server 4000   │                   │ │   │
│  │  │  └─────────────────┘   GraphQL    └─────────┬───────┘                   │ │   │
│  │  │                         Queries             │                           │ │   │
│  │  │  ┌─────────────────────────────────────────┼───────────────────────────┐ │   │
│  │  │  │              Database Tier              │                           │ │   │
│  │  │  │                                         │                           │ │   │
│  │  │  │  ┌──────────────────┐      ┌────────────▼──────┐                    │ │   │
│  │  │  │  │   DocumentDB     │      │  ElastiCache      │                    │ │   │
│  │  │  │  │ (MongoDB-compat) │      │  Redis Cluster    │                    │ │   │
│  │  │  │  │   Port 27017     │      │   Port 6379       │                    │ │   │
│  │  │  │  └──────────────────┘      └───────────────────┘                    │ │   │
│  │  │  │                                                                     │ │   │
│  │  │  │  ┌──────────────────┐      ┌───────────────────┐                    │ │   │
│  │  │  │  │  OpenSearch      │      │  S3 Bucket        │                    │ │   │
│  │  │  │  │  Service         │      │  (File Storage)   │                    │ │   │
│  │  │  │  │  Port 443        │      │   + Vector Store  │                    │ │   │
│  │  │  │  └──────────────────┘      └───────────────────┘                    │ │   │
│  │  │  └─────────────────────────────────────────────────────────────────────┘ │   │
│  │  └─────────────────────────────────────────────────────────────────────────┘ │   │
│  │                                                                               │   │
│  │  ┌─────────────────────────────────────────────────────────────────────────┐ │   │
│  │  │                     GraphQL & Serverless Services                      │ │   │
│  │  │                                                                         │ │   │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │ │   │
│  │  │  │   Lambda    │  │   API GW    │  │ EventBridge │  │   Bedrock   │   │ │   │
│  │  │  │(GraphQL Res)│  │(WebSocket)  │  │  (Events)   │  │    (AI)     │   │ │   │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │ │   │
│  │  │                                                                         │ │   │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │ │   │
│  │  │  │   Cognito   │  │ Secrets Mgr │  │     SNS     │  │ Apollo      │   │ │   │
│  │  │  │   (Auth)    │  │   (Config)  │  │(Notifications)│ │ Studio      │   │ │   │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │ │   │
│  │  └─────────────────────────────────────────────────────────────────────────┘ │   │
│  └───────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                          External Integrations                             │   │
│  │                                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │   │
│  │  │    Slack    │  │    Jira     │  │   GitHub    │  │  MS Teams   │       │   │
│  │  │     API     │  │     API     │  │     API     │  │     Bot     │       │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘       │   │
│  │                                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │   │
│  │  │   OpenAI    │  │  Anthropic  │  │   Linear    │  │   Notion    │       │   │
│  │  │     API     │  │   Claude    │  │     API     │  │     API     │       │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘       │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

## GraphQL Schema Design

### Core Schema Structure
```graphql
# User Management
type User {
  id: ID!
  email: String!
  name: String!
  avatar: String
  role: UserRole!
  teams: [Team!]!
  projects: [Project!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

# Project Management
type Project {
  id: ID!
  name: String!
  description: String
  key: String!
  owner: User!
  teams: [Team!]!
  sprints: [Sprint!]!
  epics: [Epic!]!
  stories: [Story!]!
  status: ProjectStatus!
  settings: ProjectSettings!
  integrations: [Integration!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

# Agile Entities
type Epic {
  id: ID!
  title: String!
  description: String
  project: Project!
  stories: [Story!]!
  status: EpicStatus!
  priority: Priority!
  assignee: User
  reporter: User!
  points: Int
  tags: [String!]!
  attachments: [Attachment!]!
  comments: [Comment!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Story {
  id: ID!
  title: String!
  description: String
  type: StoryType!
  epic: Epic
  sprint: Sprint
  project: Project!
  status: StoryStatus!
  priority: Priority!
  assignee: User
  reporter: User!
  storyPoints: Int
  tasks: [Task!]!
  acceptanceCriteria: [AcceptanceCriteria!]!
  comments: [Comment!]!
  attachments: [Attachment!]!
  timeTracking: TimeTracking
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Sprint {
  id: ID!
  name: String!
  goal: String
  project: Project!
  status: SprintStatus!
  startDate: DateTime!
  endDate: DateTime!
  stories: [Story!]!
  velocity: Int
  burndownData: [BurndownPoint!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

# Real-time Subscriptions
type Subscription {
  storyUpdated(projectId: ID!): Story!
  sprintUpdated(projectId: ID!): Sprint!
  commentAdded(storyId: ID!): Comment!
  userPresence(projectId: ID!): UserPresence!
  notificationReceived(userId: ID!): Notification!
}

# Mutations
type Mutation {
  # Project Operations
  createProject(input: CreateProjectInput!): Project!
  updateProject(id: ID!, input: UpdateProjectInput!): Project!
  deleteProject(id: ID!): Boolean!
  
  # Story Operations
  createStory(input: CreateStoryInput!): Story!
  updateStory(id: ID!, input: UpdateStoryInput!): Story!
  moveStory(id: ID!, status: StoryStatus!, sprintId: ID): Story!
  
  # Sprint Operations
  createSprint(input: CreateSprintInput!): Sprint!
  startSprint(id: ID!): Sprint!
  completeSprint(id: ID!): Sprint!
  
  # Bulk Operations
  bulkUpdateStories(ids: [ID!]!, input: BulkUpdateStoriesInput!): [Story!]!
  bulkMoveStories(moves: [MoveStoryInput!]!): [Story!]!
}

# Queries with Advanced Filtering
type Query {
  # Project Queries
  projects(
    filter: ProjectFilter
    sort: ProjectSort
    pagination: PaginationInput
  ): ProjectConnection!
  
  project(id: ID!): Project
  
  # Story Queries with Complex Filtering
  stories(
    filter: StoryFilter
    sort: StorySort
    pagination: PaginationInput
  ): StoryConnection!
  
  # Advanced Search
  searchStories(
    query: String!
    projectId: ID
    filters: SearchFilter
  ): [Story!]!
  
  # Analytics Queries
  projectAnalytics(
    projectId: ID!
    timeRange: TimeRange!
  ): ProjectAnalytics!
  
  sprintBurndown(sprintId: ID!): [BurndownPoint!]!
  velocityChart(projectId: ID!, sprintCount: Int = 6): VelocityChart!
}

# Input Types
input CreateStoryInput {
  title: String!
  description: String
  type: StoryType!
  projectId: ID!
  epicId: ID
  sprintId: ID
  priority: Priority!
  assigneeId: ID
  storyPoints: Int
  acceptanceCriteria: [String!]
  tags: [String!]
}

input StoryFilter {
  projectId: ID
  sprintId: ID
  epicId: ID
  assigneeId: ID
  reporterId: ID
  status: [StoryStatus!]
  priority: [Priority!]
  type: [StoryType!]
  tags: [String!]
  dateRange: DateRangeInput
}

# Enums
enum StoryStatus {
  BACKLOG
  READY
  IN_PROGRESS
  IN_REVIEW
  TESTING
  DONE
}

enum Priority {
  CRITICAL
  HIGH
  MEDIUM
  LOW
}

enum StoryType {
  USER_STORY
  BUG
  TASK
  SPIKE
  TECHNICAL_DEBT
}
```

## MongoDB Schema Design

### Document Structure
```javascript
// Projects Collection
{
  _id: ObjectId,
  name: "PureAgile Platform",
  key: "PURE",
  description: "Next-generation agile management",
  ownerId: ObjectId,
  teamIds: [ObjectId],
  settings: {
    storyPointScale: [1, 2, 3, 5, 8, 13, 21],
    workflowStates: [...],
    integrations: {...}
  },
  createdAt: ISODate,
  updatedAt: ISODate,
  // Indexes: name_text, key_1, ownerId_1, createdAt_-1
}

// Stories Collection (Main agile entity)
{
  _id: ObjectId,
  title: "Implement GraphQL subscriptions",
  description: "Add real-time updates for story changes",
  type: "USER_STORY",
  projectId: ObjectId,
  epicId: ObjectId,
  sprintId: ObjectId,
  status: "IN_PROGRESS",
  priority: "HIGH",
  assigneeId: ObjectId,
  reporterId: ObjectId,
  storyPoints: 8,
  tags: ["backend", "real-time", "graphql"],
  acceptanceCriteria: [
    "Real-time updates work across all clients",
    "Performance impact is minimal",
    "Error handling is robust"
  ],
  tasks: [
    {
      id: ObjectId,
      title: "Set up subscription resolver",
      completed: false,
      assigneeId: ObjectId
    }
  ],
  timeTracking: {
    originalEstimate: 28800, // seconds
    remainingEstimate: 14400,
    timeSpent: 14400,
    workLogs: [...]
  },
  attachments: [ObjectId],
  comments: [
    {
      id: ObjectId,
      authorId: ObjectId,
      content: "Started working on this",
      createdAt: ISODate,
      updatedAt: ISODate
    }
  ],
  metadata: {
    aiSuggestions: [...],
    similarStories: [ObjectId],
    riskScore: 0.3
  },
  createdAt: ISODate,
  updatedAt: ISODate,
  // Indexes: projectId_1_status_1, assigneeId_1, sprintId_1, 
  // title_text_description_text, tags_1, createdAt_-1
}

// Sprints Collection
{
  _id: ObjectId,
  name: "Sprint 24.10",
  goal: "Complete real-time features",
  projectId: ObjectId,
  status: "ACTIVE",
  startDate: ISODate,
  endDate: ISODate,
  storyIds: [ObjectId],
  capacity: 80, // story points
  committed: 65,
  completed: 32,
  burndownData: [
    {
      date: ISODate,
      remainingPoints: 65,
      completedPoints: 0
    }
  ],
  retrospective: {
    what_went_well: [...],
    what_to_improve: [...],
    action_items: [...]
  },
  createdAt: ISODate,
  updatedAt: ISODate
  // Indexes: projectId_1_status_1, startDate_1, endDate_1
}

// Users Collection
{
  _id: ObjectId,
  email: "dev@pureagile.com",
  name: "John Developer",
  avatar: "https://...",
  role: "DEVELOPER",
  teamIds: [ObjectId],
  preferences: {
    timezone: "UTC",
    notifications: {...},
    dashboardLayout: {...}
  },
  integrations: {
    slack: { userId: "U123456", token: "..." },
    jira: { accountId: "...", token: "..." }
  },
  createdAt: ISODate,
  updatedAt: ISODate
  // Indexes: email_1 (unique), teamIds_1
}
```

## Service Configurations

### 1. ECS Task Definitions

#### Frontend (Next.js with Apollo Client)
```yaml
Frontend ECS Task:
  CPU: 512
  Memory: 1024MB
  Port: 3000
  Health Check: /api/health
  Environment Variables:
    - NEXT_PUBLIC_GRAPHQL_ENDPOINT
    - NEXT_PUBLIC_WS_ENDPOINT
    - NEXT_PUBLIC_AUTH_DOMAIN
    - NEXT_PUBLIC_APOLLO_STUDIO_KEY
```

#### Backend (NestJS with GraphQL)
```yaml
Backend ECS Task:
  CPU: 1024
  Memory: 2048MB
  Port: 4000
  Health Check: /graphql (introspection query)
  Environment Variables:
    - MONGODB_URI (from Secrets Manager)
    - REDIS_URL (from Secrets Manager)
    - JWT_SECRET (from Secrets Manager)
    - APOLLO_KEY (from Secrets Manager)
    - AWS_REGION
    - BUCKET_NAME
```

### 2. DocumentDB Configuration
```yaml
DocumentDB Cluster:
  Engine: MongoDB 4.0 compatible
  Instance Class: db.r6g.large
  Number of Instances: 3 (1 primary, 2 replicas)
  Storage: Encrypted at rest
  Backup Retention: 7 days
  VPC Security Groups: Database access only
  
Connection Settings:
  TLS: Required
  Authentication: Username/Password in Secrets Manager
  Connection Pooling: Enabled (min: 5, max: 100)
```

### 3. ElastiCache Configuration
```yaml
Redis Cluster:
  Engine: Redis 7.0
  Node Type: cache.r6g.large
  Number of Nodes: 3
  Cluster Mode: enabled
  Backup Retention: 5 days
  Use Cases:
    - GraphQL query caching
    - Session storage
    - PubSub for subscriptions
    - DataLoader caching
```

### 4. OpenSearch Configuration
```yaml
OpenSearch Service:
  Engine Version: 2.3
  Instance Type: t3.medium.search
  Number of Instances: 3
  Storage: 50GB per node
  Use Cases:
    - Full-text search across stories/epics
    - Advanced analytics and reporting
    - Search suggestions and autocomplete
```

## GraphQL Performance Optimizations

### 1. DataLoader Implementation
```typescript
// Story DataLoader to solve N+1 problem
@Injectable()
export class StoryDataLoader {
  private readonly storyLoader: DataLoader<string, Story>;
  
  constructor(private storyService: StoryService) {
    this.storyLoader = new DataLoader<string, Story>(
      async (storyIds: string[]) => {
        const stories = await this.storyService.findByIds(storyIds);
        return storyIds.map(id => 
          stories.find(story => story.id === id) || null
        );
      },
      {
        cache: true,
        maxBatchSize: 100,
        cacheKeyFn: (key) => key
      }
    );
  }
  
  async load(storyId: string): Promise<Story> {
    return this.storyLoader.load(storyId);
  }
}

// Usage in GraphQL Resolver
@ResolverField('stories', () => [Story])
async stories(@Parent() epic: Epic): Promise<Story[]> {
  return Promise.all(
    epic.storyIds.map(id => this.storyLoader.load(id))
  );
}
```

### 2. Query Complexity Analysis
```typescript
// GraphQL Query Complexity
export const complexityPlugin = {
  plugin: depthLimit(10), // Max query depth
  options: {
    maximumComplexity: 1000,
    maximumDepth: 10,
    scalarCost: 1,
    objectCost: 2,
    listFactor: 10,
    introspectionCost: 1000,
  }
};
```

### 3. Caching Strategy
```typescript
// Redis-based GraphQL caching
@Injectable()
export class GraphQLCacheService {
  constructor(@Inject('REDIS_CLIENT') private redis: Redis) {}
  
  async get<T>(key: string): Promise<T | null> {
    const cached = await this.redis.get(key);
    return cached ? JSON.parse(cached) : null;
  }
  
  async set<T>(key: string, value: T, ttl: number = 300): Promise<void> {
    await this.redis.setex(key, ttl, JSON.stringify(value));
  }
  
  generateKey(operation: string, variables: any): string {
    const hash = createHash('sha256')
      .update(JSON.stringify({ operation, variables }))
      .digest('hex');
    return `gql:${hash}`;
  }
}
```

## Real-time Features with GraphQL Subscriptions

### 1. Subscription Implementation
```typescript
// Real-time story updates
@Subscription(() => Story, {
  filter: (payload, variables, context) => {
    return payload.storyUpdated.projectId === variables.projectId;
  }
})
storyUpdated(
  @Args('projectId') projectId: string,
  @Context() context: GqlExecutionContext
) {
  return this.pubSub.asyncIterator(`story.updated.${projectId}`);
}

// Real-time presence tracking
@Subscription(() => UserPresence)
userPresence(
  @Args('projectId') projectId: string
) {
  return this.pubSub.asyncIterator(`presence.${projectId}`);
}
```

### 2. WebSocket Configuration
```typescript
// GraphQL WebSocket setup
export const graphqlConfig = {
  autoSchemaFile: true,
  installSubscriptionHandlers: true,
  subscriptions: {
    'graphql-ws': {
      onConnect: (context) => {
        const { connectionParams, request } = context;
        return validateToken(connectionParams?.authorization);
      },
      onDisconnect: (context) => {
        // Cleanup user presence
        this.presenceService.userDisconnected(context.user.id);
      }
    }
  }
};
```

## MongoDB Aggregation Pipelines

### 1. Advanced Analytics Queries
```javascript
// Sprint velocity calculation
db.sprints.aggregate([
  {
    $match: {
      projectId: ObjectId("..."),
      status: "COMPLETED",
      endDate: { $gte: new Date(Date.now() - 180*24*60*60*1000) }
    }
  },
  {
    $lookup: {
      from: "stories",
      localField: "storyIds",
      foreignField: "_id",
      as: "stories"
    }
  },
  {
    $addFields: {
      completedPoints: {
        $sum: {
          $map: {
            input: "$stories",
            as: "story",
            in: {
              $cond: [
                { $eq: ["$$story.status", "DONE"] },
                "$$story.storyPoints",
                0
              ]
            }
          }
        }
      }
    }
  },
  {
    $group: {
      _id: null,
      avgVelocity: { $avg: "$completedPoints" },
      sprints: { $push: "$$ROOT" }
    }
  }
]);

// Story burndown data
db.stories.aggregate([
  {
    $match: {
      sprintId: ObjectId("..."),
      updatedAt: { $gte: startDate, $lte: endDate }
    }
  },
  {
    $group: {
      _id: {
        $dateToString: {
          format: "%Y-%m-%d",
          date: "$updatedAt"
        }
      },
      completed: {
        $sum: {
          $cond: [{ $eq: ["$status", "DONE"] }, "$storyPoints", 0]
        }
      },
      total: { $sum: "$storyPoints" }
    }
  },
  { $sort: { "_id": 1 } }
]);
```

### 2. Search and Filtering
```javascript
// Full-text search with MongoDB
db.stories.createIndex({
  title: "text",
  description: "text",
  tags: "text"
});

// Advanced search query
db.stories.aggregate([
  {
    $match: {
      $and: [
        { projectId: ObjectId("...") },
        {
          $or: [
            { $text: { $search: "authentication login" } },
            { tags: { $in: ["auth", "security"] } }
          ]
        }
      ]
    }
  },
  {
    $addFields: {
      score: { $meta: "textScore" }
    }
  },
  { $sort: { score: { $meta: "textScore" } } }
]);
```

## Security & Authorization

### 1. GraphQL Field-Level Security
```typescript
// Role-based field access
@ObjectType()
export class Story {
  @Field()
  id: string;
  
  @Field()
  title: string;
  
  @Field(() => String, { nullable: true })
  @Authorized(['ADMIN', 'PROJECT_MANAGER']) // Only admins can see
  internalNotes?: string;
  
  @Field(() => [Comment])
  @Authorized() // Any authenticated user
  comments: Comment[];
}

// Custom authorization decorator
@Directive('@auth(requires: ADMIN)')
@Field(() => [User])
async sensitiveUserData(): Promise<User[]> {
  return this.userService.getAllUsersWithSensitiveData();
}
```

### 2. Rate Limiting & Query Analysis
```typescript
// GraphQL rate limiting
export const rateLimitConfig = {
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 1000, // limit each IP to 1000 requests per windowMs
  message: 'Too many requests from this IP',
  standardHeaders: true,
  legacyHeaders: false,
};

// Query depth and complexity limiting
export const securityConfig = {
  depthLimit: 10,
  complexityLimit: 1000,
  costAnalysis: {
    maximumCost: 1000,
    defaultCost: 1,
    scalarCost: 1,
    objectCost: 2,
    listFactor: 10,
  }
};
```

## Integration Architecture

### 1. Slack Integration
```typescript
// Slack GraphQL resolver
@Resolver()
export class SlackIntegrationResolver {
  
  @Mutation(() => Boolean)
  async syncSlackChannels(
    @Args('projectId') projectId: string,
    @CurrentUser() user: User
  ): Promise<boolean> {
    const integration = await this.integrationService.getSlackIntegration(projectId);
    const channels = await this.slackService.getChannels(integration.token);
    
    // Create stories from Slack messages
    for (const channel of channels) {
      const messages = await this.slackService.getMessages(channel.id);
      await this.processSlackMessages(messages, projectId);
    }
    
    return true;
  }
  
  @Subscription(() => SlackMessage)
  slackMessageReceived(@Args('channelId') channelId: string) {
    return this.pubSub.asyncIterator(`slack.message.${channelId}`);
  }
}
```

### 2. Jira Synchronization
```typescript
// Bi-directional Jira sync
@Injectable()
export class JiraSyncService {
  
  async syncStoryToJira(story: Story): Promise<void> {
    const jiraIssue = await this.jiraClient.createIssue({
      fields: {
        project: { key: story.project.jiraKey },
        summary: story.title,
        description: story.description,
        issuetype: { name: this.mapStoryTypeToJira(story.type) },
        priority: { name: this.mapPriorityToJira(story.priority) }
      }
    });
    
    await this.storyService.updateStory(story.id, {
      integrations: {
        jira: { issueId: jiraIssue.id, key: jiraIssue.key }
      }
    });
  }
  
  @EventPattern('jira.issue.updated')
  async handleJiraWebhook(data: JiraWebhookData): Promise<void> {
    const story = await this.storyService.findByJiraKey(data.issue.key);
    if (story) {
      await this.storyService.updateFromJira(story.id, data.issue);
      
      // Notify via GraphQL subscription
      this.pubSub.publish(`story.updated.${story.projectId}`, {
        storyUpdated: story
      });
    }
  }
}
```

## Monitoring & Observability

### 1. GraphQL Metrics
```typescript
// Custom GraphQL metrics
export const metricsPlugin = {
  requestDidStart() {
    return {
      didResolveOperation(context) {
        const operationType = context.request.operationName;
        this.incrementCounter('graphql_operations_total', {
          operation: operationType,
          type: context.operation.operation
        });
      },
      
      didEncounterErrors(context) {
        context.errors.forEach(error => {
          this.incrementCounter('graphql_errors_total', {
            type: error.constructor.name,
            operation: context.request.operationName
          });
        });
      },
      
      willSendResponse(context) {
        const duration = Date.now() - context.request.http.timestamp;
        this.recordHistogram('graphql_request_duration', duration, {
          operation: context.request.operationName
        });
      }
    };
  }
};
```

### 2. Performance Monitoring
```yaml
CloudWatch Custom Metrics:
  - GraphQL operation latency
  - MongoDB connection pool usage
  - Redis cache hit ratio
  - WebSocket connection count
  - Story creation/completion rate
  - Sprint velocity trends
  - User engagement metrics
  
Apollo Studio Metrics:
  - Query performance
  - Schema usage analytics
  - Error tracking
  - Field-level performance
  - Operation complexity analysis
```

## Cost Optimization for MongoDB Workload

### Production Environment
```yaml
DocumentDB Cluster:
  Primary: db.r6g.xlarge (4 vCPU, 32GB RAM)
  Replicas: 2x db.r6g.large (2 vCPU, 16GB RAM)
  Storage: 200GB encrypted
  
ECS Services:
  GraphQL API: 3x tasks (1 vCPU, 2GB RAM)
  Frontend: 2x tasks (0.5 vCPU, 1GB RAM)
  
ElastiCache:
  Redis: 3x cache.r6g.large nodes
  
OpenSearch:
  3x t3.medium.search nodes (50GB each)
  
```

### Staging Environment
```yaml
DocumentDB Cluster:
  Single instance: db.t4g.medium (2 vCPU, 4GB RAM)
  
ECS Services:
  GraphQL API: 1x task (0.5 vCPU, 1GB RAM)
  Frontend: 1x task (0.25 vCPU, 0.5GB RAM)
  
ElastiCache:
  Redis: 1x cache.t4g.micro node
  
OpenSearch:
  1x t3.small.search node (20GB)
```

This architecture provides a comprehensive, production-ready foundation for PureAgile with MongoDB and GraphQL, offering excellent scalability, real-time capabilities, and robust integration options while maintaining cost-effectiveness and operational simplicity.
