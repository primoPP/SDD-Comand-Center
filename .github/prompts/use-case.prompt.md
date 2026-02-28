---
name: use-case
description: Template for business logic layer components
version: 5.0.1
project: vscode-copilot-chat
agent: implementation
language: en
---

# Use-Case Prompt Template

## Purpose

Create business logic layer components (UseCase, Service, Handler, Hook, Store) following project-specific patterns.

## Project Context

- **Project**: vscode-copilot-chat
- **Domain**: VS Code Copilot Chat Extension — AI-powered coding assistant providing conversational AI, inline editing, agent mode, language model tools, and MCP integration
- **Language**: TypeScript
- **Framework**: VS Code Extension API (vscode ^1.110.0)

## Technical Context

- **Architecture Style**: Service-oriented with custom DI (`IInstantiationService`), Contribution-based modular system, Layer hierarchy (util → platform → extension)
- **Business Logic Location**: `src/extension/<feature>/` (organized by feature: conversation/, tools/, context/, prompts/, mcp/, testing/, etc.)
- **Component Pattern**: Service class implementing interface, registered via `SyncDescriptor` and `IInstantiationService`; Contributions via `IExtensionContribution` + `asContributionFactory()`
- **Interface Pattern**: Interface defined with `createServiceIdentifier<T>('IMyService')`, implementation class with `@IServiceName` parameter decorators
- **Dependency Injection**: Custom `@IServiceName` parameter decorator injection; `IInstantiationService.createInstance()` for runtime resolution; `SyncDescriptor` for lazy instantiation
- **Transaction Management**: N/A (no database; state managed via VS Code API disposables and event emitters)
- **Validation**: TypeScript strict mode, `readonly` properties, compile-time type checking; runtime assertions where needed
- **Exception Handling**: try/catch with `ILogService` for structured logging; VS Code error notifications via `vscode.window.showErrorMessage()`
- **Logging**: `ILogService` (never `console.log`); structured telemetry via `ITelemetryService`

## Input Parameters

| Parameter | Description | Required |
|-----------|-------------|----------|
| `use-case-name` | Name of the use case/service | Yes |
| `command` | Command from spec to implement | Yes |
| `inputs` | Input parameters | Yes |
| `outputs` | Expected output type | Yes |
| `business-rules` | Business rules to enforce | Yes |
| `side-effects` | Side effects (persistence, events) | No |

## Execution Steps

### Step 1: Apply Project Patterns

Use the embedded project context.

### Step 2: Create Interface (if pattern requires)

```
If project uses Interface + Implementation pattern:
1. Create interface with execute/handle method
2. Define input/output types
3. Place in correct location
```

### Step 3: Create Implementation

```
Create implementation following:
- Location: src/extension/<feature>/ or src/platform/<service>/
- Pattern: Service class implementing interface, registered via SyncDescriptor
- DI: @IServiceName parameter decorator injection via IInstantiationService
```

### Step 4: Implement Business Rules

```
For each business rule from spec:
1. Implement validation/check
2. Throw appropriate exception if violated
3. Log relevant information
```

### Step 5: Handle Side Effects

```
For each side effect:
- Persistence: Call Gateway/Repository
- Events: Publish domain events
- External calls: Use appropriate Gateway
```

## Output Expectations

- Use case/service file(s)
- Interface file (if pattern requires)
- Correct dependency injection setup
- All business rules implemented
- Proper exception handling

## Constraints

- MUST follow project service/contribution pattern exactly
- MUST implement ALL business rules from spec
- MUST use `ILogService` for logging (never `console.log`)
- MUST use arrow functions `=>` over anonymous function expressions
- MUST respect import hierarchy: util → platform → extension (NO reverse imports)
- MUST use `readonly` whenever possible
- MUST NOT use `any` or `unknown` unless absolutely necessary
- MUST properly dispose resources via VS Code Disposable pattern

## Example (Project-Specific)

```typescript
// Service interface
export const IMyFeatureService = createServiceIdentifier<IMyFeatureService>('IMyFeatureService');
export interface IMyFeatureService {
	processRequest(input: MyInput): Promise<MyOutput>;
}

// Service implementation with DI
export class MyFeatureService implements IMyFeatureService {
	constructor(
		@ILogService private readonly logService: ILogService,
		@IContextService private readonly contextService: IContextService,
		@ITelemetryService private readonly telemetryService: ITelemetryService,
	) { }

	async processRequest(input: MyInput): Promise<MyOutput> {
		try {
			const context = await this.contextService.resolve(input);
			// business logic
			this.telemetryService.logEvent('myFeature/processed');
			return result;
		} catch (e) {
			this.logService.error('Failed to process request', e);
			throw e;
		}
	}
}

// Contribution to register the service
export class MyFeatureContribution implements IExtensionContribution {
	static readonly ID = 'myFeature';
	constructor(@IMyFeatureService private readonly myFeature: IMyFeatureService) { }
}
```

## Validation Checklist

Before completing:
- [ ] Component is in correct package/module
- [ ] All business rules from spec are implemented
- [ ] Dependencies are properly injected
- [ ] Side effects are delegated correctly
- [ ] Follows project naming conventions
- [ ] Exception handling matches project pattern
