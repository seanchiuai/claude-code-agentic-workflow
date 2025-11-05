# Internal API Reference

## Core APIs

### [Module/Service Name]

**Purpose:** [What this API provides]

**Key Functions:**

#### `functionName(param1, param2)`
- **Purpose:** [What it does]
- **Parameters:**
  - `param1` (type) - Description
  - `param2` (type) - Description
- **Returns:** [Return type and description]
- **Example:**
  ```javascript
  // Usage example
  ```
- **Notes:** [Important considerations, edge cases]

## Data Contracts

### [Entity Name]
```typescript
interface EntityName {
  field1: type;  // Description
  field2: type;  // Description
}
```

**Validation Rules:**
- [Field validation requirements]
- [Business rules]

## Event System

### Events Emitted
- `event-name` - When [trigger condition], payload: `{...}`

### Events Consumed
- `event-name` - Handler in [location], behavior: [description]

## External Integrations

### [Service Name]
- **Endpoint:** [URL/path]
- **Auth:** [Auth method]
- **Rate Limits:** [Limits]
- **Error Handling:** [How errors are handled]

## Configuration

### Environment Variables
- `VAR_NAME` - [Purpose, default value]

### Feature Flags
- `feature-name` - [What it controls, default state]

---

*Keep API docs in sync with implementation. Document contracts, not implementation details.*
