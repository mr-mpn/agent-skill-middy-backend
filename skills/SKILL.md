---
name: middy-serverless
description: Development guide for building serverless backends using the Middy middleware framework. Details folder structure, routing, authentication, and error handling.
---
# Middy Serverless Development

This skill provides comprehensive instructions on structuring and developing serverless APIs using Node.js and the Middy middleware library.

---

## 1. Directory & File Structure

Middy-based projects must follow a single-entry-point architecture. Restructure or initialize your projects using the following layout:

```
backend/
├── serverless.yml            ← Serverless Framework configuration
├── package.json              ← Node.js dependencies
└── src/
    ├── main.js               ← Main Lambda handler entry point (minimal wrapper)
    ├── handlers/             ← Route handlers
    │   ├── router.js         ← Route mapping file using @middy/http-router
    │   ├── auth.js           ← Feature-specific handler functions (e.g., login, register)
    │   └── posts.js          ← Feature-specific handler functions (e.g., CRUD logic)
    ├── middleware/           ← Custom Middy middlewares (e.g., authenticate.js)
    ├── services/             ← Business logic/Database access layer
    └── utils/                ← Helper utilities (e.g., response formatting, db client)
```

---

## 2. Serverless Mapping (Single Entry Point)

To simplify routing and local development, map all HTTP routes to a single Lambda function (`src/main.handler`) using wildcards.

Example `serverless.yml` configuration:
```yaml
service: my-middy-service

provider:
  name: aws
  runtime: nodejs22.x
  stage: ${opt:stage, 'dev'}

functions:
  api:
    handler: src/main.handler
    timeout: 30
    events:
      - http:
          path: /
          method: any
          cors: true
      - http:
          path: /{proxy+}
          method: any
          cors: true

plugins:
  - serverless-offline
```

---

## 3. Main Handler (`src/main.js`)

The main entry point acts as a minimal wrapper around the router. Keep this file as simple as possible.

```javascript
import middy from '@middy/core';
import { router } from './handlers/router.js';

export const handler = middy(router);
```

---

## 4. Router Configuration (`src/handlers/router.js`)

Utilize `@middy/http-router` to define REST endpoints. Apply middlewares such as body parsing, error handling, and authentication selectively on a per-route basis.

```javascript
import middy from '@middy/core';
import httpRouterHandler from '@middy/http-router';
import jsonBodyParser from '@middy/http-json-body-parser';
import { authenticate } from '../middleware/authenticate.js';
import { errorHandler } from '../utils/response.js';
import { getItemsHandler, createItemHandler } from './items.js';

const routes = [
  {
    method: 'GET',
    path: '/items',
    handler: middy(getItemsHandler)
      .use(errorHandler())
  },
  {
    method: 'POST',
    path: '/items',
    handler: middy(createItemHandler)
      .use(jsonBodyParser())
      .use(authenticate())
      .use(errorHandler())
  }
];

export const router = httpRouterHandler(routes);
export default router;
```

---

## 5. Custom Authentication Middleware (`src/middleware/authenticate.js`)

Middy middlewares are objects with `before`, `after`, and `onError` lifecycle hooks. A typical authentication middleware intercepts incoming requests to parse and validate headers.

```javascript
export class UnauthorizedError extends Error {
  constructor(message) {
    super(message);
    this.name = 'UnauthorizedError';
    this.statusCode = 401;
  }
}

export const authenticate = () => {
  return {
    before: async (request) => {
      const authHeader = request.event.headers?.authorization || request.event.headers?.Authorization;
      
      if (!authHeader || !authHeader.startsWith('Bearer ')) {
        throw new UnauthorizedError('Missing or invalid Authorization header');
      }

      const token = authHeader.split(' ')[1];

      try {
        const decoded = verifyToken(token); // Implement your token validation logic
        
        // Attach decoded user credentials/info to the event object
        request.event.user = decoded;
      } catch (error) {
        throw new UnauthorizedError('Invalid token');
      }
    },
  };
};
```

---

## 6. Response Helpers & Error Middleware (`src/utils/response.js`)

Maintain consistent API response schemas by utilizing helper functions for success and error cases.

```javascript
export const successResponse = (data, statusCode = 200) => {
  return {
    statusCode,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      success: true,
      data,
    }),
  };
};

export const errorResponse = (message, statusCode = 500, details = undefined) => {
  const body = {
    success: false,
    error: message,
  };

  if (process.env.STAGE !== 'prod' && details) {
    body.details = details;
  }

  return {
    statusCode,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(body),
  };
};

// Middy middleware to catch unhandled errors
export const errorHandler = () => {
  return {
    onError: (request) => {
      const { error } = request;
      console.error('Unhandled Error:', error);
      
      const statusCode = error.statusCode || 500;
      const message = error.message || 'Internal Server Error';
      
      request.response = errorResponse(message, statusCode, error.stack);
    },
  };
};
```

---

## 7. Initialization & Dependencies (For Agents)

When an agent initializes a new project using this skill, they must execute the following setup steps:

1. **Initialize Project**: Run `npm init -y` in the `backend/` directory.
2. **ES Modules**: Update `package.json` to include `"type": "module"` (required since the code uses `import/export`).
3. **Install Dependencies**: 
   ```bash
   npm install @middy/core @middy/http-router @middy/http-json-body-parser
   npm install -D serverless serverless-offline
   ```

---

## 8. Implementation Notes for Agents

- **`authenticate.js`**: The `verifyToken()` function is a placeholder. Agents must implement concrete JWT (or other) verification logic.
- **`router.js`**: The `items.js` imported in the example is a placeholder. Agents should create actual feature handlers (e.g., `posts.js`, `users.js`) based on the user's specific requirements.
