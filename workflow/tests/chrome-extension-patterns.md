# Chrome Extension Test Patterns

## Infrastructure
- **Test framework:** Jest
- **Test runner:** `npm test` or `jest`
- **Config file:** `jest.config.js`
- **Mocking library:** sinon or jest.mock for Chrome APIs

## Setup Patterns

### jest.config.js
```js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/tests/setup.js'],
  testMatch: ['**/__tests__/**/*.test.js', '**/*.test.js'],
  moduleNameMapper: {
    '\\.(css|less)$': 'identity-obj-proxy'
  }
};
```

### tests/setup.js - Chrome API Mocks
```js
global.chrome = {
  storage: {
    local: {
      get: jest.fn((keys, callback) => callback({})),
      set: jest.fn((items, callback) => callback && callback()),
    },
    sync: {
      get: jest.fn((keys, callback) => callback({})),
      set: jest.fn((items, callback) => callback && callback()),
    }
  },
  runtime: {
    sendMessage: jest.fn(),
    onMessage: { addListener: jest.fn() }
  },
  tabs: {
    query: jest.fn((queryInfo, callback) => callback([])),
    sendMessage: jest.fn()
  }
};
```

## Code Patterns

### Testing Storage Operations
```js
describe('Storage operations', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should save data to chrome.storage.local', async () => {
    await saveSettings({ theme: 'dark' });

    expect(chrome.storage.local.set).toHaveBeenCalledWith(
      { theme: 'dark' },
      expect.any(Function)
    );
  });
});
```

### Testing Message Passing
```js
describe('Message handling', () => {
  it('should send message to background', () => {
    sendMessageToBackground({ action: 'getData' });

    expect(chrome.runtime.sendMessage).toHaveBeenCalledWith(
      { action: 'getData' },
      expect.any(Function)
    );
  });
});
```

### Testing DOM Manipulation (Sidepanel/Popup)
```js
describe('UI component', () => {
  beforeEach(() => {
    document.body.innerHTML = '<div id="app"></div>';
  });

  it('should render button on initialization', () => {
    initializeUI();

    const button = document.querySelector('#save-btn');
    expect(button).toBeTruthy();
  });
});
```

## Learned Patterns
(TDD skill will append learned patterns here)
