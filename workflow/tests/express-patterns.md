# Express API Test Patterns

## Infrastructure
- **Test framework:** Jest or Mocha
- **Test runner:** `npm test`
- **HTTP testing:** supertest
- **Database:** Mock or test database

## Setup Patterns

### jest.config.js
```js
module.exports = {
  testEnvironment: 'node',
  testMatch: ['**/__tests__/**/*.test.js', '**/tests/**/*.test.js'],
  setupFilesAfterEnv: ['<rootDir>/tests/setup.js'],
};
```

### tests/setup.js
```js
const mongoose = require('mongoose');

beforeAll(async () => {
  // Connect to test database
  await mongoose.connect(process.env.TEST_MONGO_URI);
});

afterAll(async () => {
  await mongoose.connection.close();
});

afterEach(async () => {
  // Clean up collections
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});
```

## Code Patterns

### Endpoint Testing
```js
const request = require('supertest');
const app = require('../app');

describe('GET /api/users', () => {
  it('should return all users', async () => {
    const res = await request(app)
      .get('/api/users')
      .expect('Content-Type', /json/)
      .expect(200);

    expect(Array.isArray(res.body)).toBe(true);
  });
});

describe('POST /api/users', () => {
  it('should create new user', async () => {
    const userData = { name: 'John', email: 'john@example.com' };

    const res = await request(app)
      .post('/api/users')
      .send(userData)
      .expect(201);

    expect(res.body).toHaveProperty('id');
    expect(res.body.name).toBe('John');
  });

  it('should return 400 for invalid data', async () => {
    await request(app)
      .post('/api/users')
      .send({ name: '' })
      .expect(400);
  });
});
```

### Middleware Testing
```js
describe('Auth middleware', () => {
  it('should reject requests without token', async () => {
    await request(app)
      .get('/api/protected')
      .expect(401);
  });

  it('should allow requests with valid token', async () => {
    const token = generateTestToken();

    await request(app)
      .get('/api/protected')
      .set('Authorization', `Bearer ${token}`)
      .expect(200);
  });
});
```

### Database Mock Pattern
```js
jest.mock('../models/User');
const User = require('../models/User');

describe('User service', () => {
  it('should find user by id', async () => {
    User.findById.mockResolvedValue({ id: 1, name: 'John' });

    const user = await userService.findById(1);
    expect(user.name).toBe('John');
  });
});
```

## Learned Patterns
(TDD skill will append learned patterns here)
