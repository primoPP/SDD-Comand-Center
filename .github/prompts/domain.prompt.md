---
name: domain
description: Template for domain entity/model creation
version: 5.0.1
project: vscode-copilot-chat
agent: implementation
language: en
---

# Domain Prompt Template

## Purpose

Create domain entities and value objects following project-specific patterns.

## Project Context

- **Project**: vscode-copilot-chat
- **Domain**: VS Code Copilot Chat Extension — AI-powered coding assistant providing conversational AI, inline editing, agent mode, language model tools, and MCP integration
- **Language**: TypeScript
- **Framework**: VS Code Extension API (vscode ^1.110.0)

## Technical Context

- **Domain Location**: `src/extension/` (organized by feature: conversation/, tools/, context/, prompts/, mcp/)
- **Entity Pattern**: Service-oriented classes with custom DI via `@IServiceName` parameter decorator injection; interfaces defined with `createServiceIdentifier<T>()`
- **Validation**: TypeScript strict mode + runtime assertions; `readonly` properties for immutability
- **Immutability**: TypeScript `readonly` modifier, `ReadonlyArray<T>`, `Object.freeze()` for config objects
- **Framework Dependencies**: `vscode` API namespace, `IInstantiationService`, `SyncDescriptor` for lazy instantiation
- **ID Pattern**: String-based identifiers; `createServiceIdentifier<T>('IMyService')` for service registration

## Input Parameters

| Parameter | Description | Required |
|-----------|-------------|----------|
| `entity-name` | Name of the domain entity | Yes |
| `attributes` | List of entity attributes | Yes |
| `invariants` | Business rules to enforce | Yes |
| `value-objects` | Nested value objects if any | No |

## Execution Steps

### Step 1: Apply Project Patterns

Use the embedded project context (no inspection needed at runtime).

### Step 2: Create Entity Structure

```
Create domain entity following:
- Location: src/extension/<feature>/ or src/platform/<service>/
- Pattern: Interface + Implementation class with @IServiceName DI decorator
- Immutability: readonly properties, ReadonlyArray<T>, Object.freeze()
```

### Step 3: Implement Invariants

```
For each invariant:
1. Implement validation logic in constructor or factory method
2. Throw domain exception if invariant is violated
3. Follow project's exception pattern
```

### Step 4: Create Value Objects

```
For each value object:
1. Create as immutable type
2. Implement validation
3. Place in same domain package/module
```

## Output Expectations

- Service interface file with `createServiceIdentifier<T>()` registration
- Implementation class with `@IServiceName` DI decorators
- TypeScript types/interfaces for data contracts
- Clear separation: `src/util/` → `src/platform/` → `src/extension/` (no reverse imports)

## Constraints

- MUST follow project patterns exactly (service-oriented DI, contribution system)
- MUST respect import hierarchy: util → platform → extension (NO reverse imports)
- MUST implement all invariants from spec
- MUST use `ILogService` for logging (not `console.log`)
- MUST use `IFileSystemService` instead of Node `fs` or `vscode.workspace.fs`
- MUST use `readonly` whenever possible
- MUST NOT use `any` unless absolutely necessary

## Example (Project-Specific)

```typescript
// Service interface definition
export const IMyFeatureService = createServiceIdentifier<IMyFeatureService>('IMyFeatureService');
export interface IMyFeatureService {
	readonly onDidChange: Event<void>;
	getItems(): ReadonlyArray<MyItem>;
	processItem(item: MyItem): Promise<void>;
}

// Implementation with DI
export class MyFeatureService implements IMyFeatureService {
	private readonly _onDidChange = new Emitter<void>();
	readonly onDidChange = this._onDidChange.event;

	constructor(
		@ILogService private readonly logService: ILogService,
		@IInstantiationService private readonly instantiationService: IInstantiationService,
	) { }
}

// Registration via contribution
export class MyFeatureContribution implements IExtensionContribution {
	static readonly ID = 'myFeature';
	constructor(@IMyFeatureService private readonly myFeature: IMyFeatureService) { }
}
```

## Validation Checklist

Before completing:
- [ ] Entity is in correct package/module
- [ ] All invariants are implemented
- [ ] No framework annotations in domain
- [ ] Value objects are immutable
- [ ] Follows project naming conventions
