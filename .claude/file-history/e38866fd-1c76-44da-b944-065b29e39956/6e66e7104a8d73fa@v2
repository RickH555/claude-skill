# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Directive de session obligatoire - PRIORITE ABSOLUE

**A CHAQUE NOUVELLE SESSION**, Claude Code DOIT IMPERATIVEMENT :

1. **Consulter le fichier `C:\Users\User\.claude\all-capabilities.md`** — c'est le catalogue UNIQUE et COMPLET de tous les skills, agents, plugins et commandes disponibles (400+ capabilities).
2. **Scanner l'INDEX RAPIDE PAR BESOIN** en haut du fichier pour identifier instantanement les capabilities pertinentes au contexte du projet ou de la tache.
3. **Utiliser AUTOMATIQUEMENT les skills et agents adaptes** SANS attendre que l'utilisateur le demande. C'est une obligation, pas une option :
   - Debug → `systematic-debugging`, `debug`, agents `debugger`, `error-detective`, `gsd-debugger`
   - Architecture → `senior-architect`, `c4-architecture`, agents `backend-architect`, `architect-review`
   - API → `api-design`, `api-scaffolding`, agents `graphql-architect`, `api-documenter`
   - Frontend → `frontend-design`, `frontend-patterns`, `senior-frontend`, agent `frontend-developer`
   - Mobile → `frontend-mobile-development`, agents `mobile-developer`, `flutter-expert`
   - Backend → `backend-patterns`, `python-development`, `golang-patterns`, agents `python-pro`, `golang-pro`
   - BDD → `postgres-patterns`, `database-design`, agents `database-architect`, `sql-pro`
   - Securite → `security-review`, `owasp-security`, `security-scanning`, agents `security-auditor`, `threat-modeling-expert`
   - DevOps/CI-CD → `deployment-patterns`, `cicd-automation`, agents `deployment-engineer`, `terraform-specialist`
   - Cloud → `aws-solution-architect`, `cloud-infrastructure`, agents `cloud-architect`, `kubernetes-architect`
   - ML/AI/LLM → `machine-learning-ops`, `llm-application-dev`, agents `ai-engineer`, `ml-engineer`
   - Tests → `test`, `tdd-workflow`, `senior-qa`, agents `tdd-orchestrator`, `test-automator`
   - SEO → `seo-*` plugins et agents
   - Documents → `pdf`, `docx`, `pptx`, `xlsx`
   - Design UI/UX → `ui-ux-pro-max`, `ui-design`, agents `ui-designer`
   - Performance → `application-performance`, agents `performance-engineer`
   - Gestion projet → `/gsd:*`, `senior-pm`, `scrum-master`
   - Trading → `quantitative-trading`, agents `quant-analyst`
   - Blockchain → `blockchain-web3`, agent `blockchain-developer`
   - Game Dev → `game-development`, agents `unity-developer`
   - Recherche → `deep-research`, `fact-checker`, agent `Explore`
   - Parallelisation → `dispatching-parallel-agents`, `agent-teams`
   - Refactoring → `refactor`, `code-refactoring`, agents `legacy-modernizer`
4. **Proposer proactivement** les skills pertinents meme si l'utilisateur ne les mentionne pas.
5. **Pour les langages specifiques**, toujours utiliser le plugin/agent dedie : `python-pro`, `typescript-pro`, `rust-pro`, `java-pro`, `golang-pro`, `cpp-pro`, `ruby-pro`, `php-pro`, `julia-pro`, etc.

> **FICHIER DE REFERENCE UNIQUE** : `C:\Users\User\.claude\all-capabilities.md`
> Contient le catalogue complet : 72 plugins wshobson (112+ agents), 18 plugins officiels, 140 skills standalone, 12 agents GSD, 30 commandes GSD, 5 serveurs MCP = **400+ capabilities**.

## Project Overview

This is a JavaScript/TypeScript project optimized for modern web development. The project uses industry-standard tools and follows best practices for scalable application development.

## Development Commands

### Package Management
- `npm install` or `yarn install` - Install dependencies
- `npm ci` or `yarn install --frozen-lockfile` - Install dependencies for CI/CD
- `npm update` or `yarn upgrade` - Update dependencies

### Build Commands
- `npm run build` - Build the project for production
- `npm run dev` or `npm start` - Start development server
- `npm run preview` - Preview production build locally

### Testing Commands
- `npm test` or `npm run test` - Run all tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Run tests with coverage report
- `npm run test:unit` - Run unit tests only
- `npm run test:integration` - Run integration tests only
- `npm run test:e2e` - Run end-to-end tests

### Code Quality Commands
- `npm run lint` - Run ESLint for code linting
- `npm run lint:fix` - Run ESLint with auto-fix
- `npm run format` - Format code with Prettier
- `npm run format:check` - Check code formatting
- `npm run typecheck` - Run TypeScript type checking

### Development Tools
- `npm run storybook` - Start Storybook (if available)
- `npm run analyze` - Analyze bundle size
- `npm run clean` - Clean build artifacts

## Technology Stack

### Core Technologies
- **JavaScript/TypeScript** - Primary programming languages
- **Node.js** - Runtime environment
- **npm/yarn** - Package management

### Common Frameworks
- **React** - UI library with hooks and functional components
- **Vue.js** - Progressive framework for building user interfaces
- **Angular** - Full-featured framework for web applications
- **Express.js** - Web application framework for Node.js
- **Next.js** - React framework with SSR/SSG capabilities

### Build Tools
- **Vite** - Fast build tool and development server
- **Webpack** - Module bundler
- **Rollup** - Module bundler for libraries
- **esbuild** - Extremely fast JavaScript bundler

### Testing Framework
- **Jest** - JavaScript testing framework
- **Vitest** - Fast unit test framework
- **Testing Library** - Simple and complete testing utilities
- **Cypress** - End-to-end testing framework
- **Playwright** - Cross-browser testing

### Code Quality Tools
- **ESLint** - JavaScript/TypeScript linter
- **Prettier** - Code formatter
- **TypeScript** - Static type checking
- **Husky** - Git hooks

## Project Structure Guidelines

### File Organization
```
src/
├── components/     # Reusable UI components
├── pages/         # Page components or routes
├── hooks/         # Custom React hooks
├── utils/         # Utility functions
├── services/      # API calls and external services
├── types/         # TypeScript type definitions
├── constants/     # Application constants
├── styles/        # Global styles and themes
└── tests/         # Test files
```

### Naming Conventions
- **Files**: Use kebab-case for file names (`user-profile.component.ts`)
- **Components**: Use PascalCase for component names (`UserProfile`)
- **Functions**: Use camelCase for function names (`getUserData`)
- **Constants**: Use UPPER_SNAKE_CASE for constants (`API_BASE_URL`)
- **Types/Interfaces**: Use PascalCase with descriptive names (`UserData`, `ApiResponse`)

## TypeScript Guidelines

### Type Safety
- Enable strict mode in `tsconfig.json`
- Use explicit types for function parameters and return values
- Prefer interfaces over types for object shapes
- Use union types for multiple possible values
- Avoid `any` type - use `unknown` when type is truly unknown

### Best Practices
- Use type guards for runtime type checking
- Leverage utility types (`Partial`, `Pick`, `Omit`, etc.)
- Create custom types for domain-specific data
- Use enums for finite sets of values
- Document complex types with JSDoc comments

## Code Quality Standards

### ESLint Configuration
- Use recommended ESLint rules for JavaScript/TypeScript
- Enable React-specific rules if using React
- Configure import/export rules for consistent module usage
- Set up accessibility rules for inclusive development

### Prettier Configuration
- Use consistent indentation (2 spaces recommended)
- Set maximum line length (80-100 characters)
- Use single quotes for strings
- Add trailing commas for better git diffs

### Testing Standards
- Aim for 80%+ test coverage
- Write unit tests for utilities and business logic
- Use integration tests for component interactions
- Implement e2e tests for critical user flows
- Follow AAA pattern (Arrange, Act, Assert)

## Performance Optimization

### Bundle Optimization
- Use code splitting for large applications
- Implement lazy loading for routes and components
- Optimize images and assets
- Use tree shaking to eliminate dead code
- Analyze bundle size regularly

### Runtime Performance
- Implement proper memoization (React.memo, useMemo, useCallback)
- Use virtualization for large lists
- Optimize re-renders in React applications
- Implement proper error boundaries
- Use web workers for heavy computations

## Security Guidelines

### Dependencies
- Regularly audit dependencies with `npm audit`
- Keep dependencies updated
- Use lock files (`package-lock.json`, `yarn.lock`)
- Avoid dependencies with known vulnerabilities

### Code Security
- Sanitize user inputs
- Use HTTPS for API calls
- Implement proper authentication and authorization
- Store sensitive data securely (environment variables)
- Use Content Security Policy (CSP) headers

## Development Workflow

### Before Starting
1. Check Node.js version compatibility
2. Install dependencies with `npm install`
3. Copy environment variables from `.env.example`
4. Run type checking with `npm run typecheck`

### During Development
1. Use TypeScript for type safety
2. Run linter frequently to catch issues early
3. Write tests for new features
4. Use meaningful commit messages
5. Review code changes before committing

### Before Committing
1. Run full test suite: `npm test`
2. Check linting: `npm run lint`
3. Verify formatting: `npm run format:check`
4. Run type checking: `npm run typecheck`
5. Test production build: `npm run build`