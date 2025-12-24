# Configuration and Storage Domain Technical Documentation

## Overview

The Configuration and Storage Domain is a foundational infrastructure domain within the Gemini CLI Core backend system. It centralizes the management of all configuration-related file paths and project-specific storage directories. This domain ensures consistent, secure, and isolated storage of configuration files, credentials, temporary data, and history across multiple projects and user environments.

The core module in this domain is the `Storage` class, implemented in `config/storage.ts`. It provides a comprehensive API for accessing and managing global and project-scoped directories and files, abstracting away platform-specific path handling and file system operations. This design supports multi-project isolation, extensibility, and environment compatibility.

---

## Responsibilities

- Provide static methods to access global configuration directories and files, including user credentials, OAuth tokens, policy directories, command definitions, and system-wide settings.
- Offer instance methods to manage project-specific directories such as temporary storage, history, extensions, commands, and agents.
- Ensure unique and isolated storage paths per project by generating SHA-256 hash-based directory names derived from project root paths.
- Abstract platform-dependent path resolution and environment variable overrides for system settings.
- Guarantee existence of required directories through file system operations.
- Hide underlying file system and path manipulation details from consumers, exposing only path retrieval interfaces.

---

## Key Concepts and Components

### 1. Global Gemini Directory

- Located under the user's home directory (e.g., `~/.gemini`), or fallback to the OS temporary directory if the home directory is unavailable.
- Contains global configuration files and directories such as:
  - OAuth token storage (`mcp-oauth-tokens.json`)
  - User account information (`google_accounts.json`)
  - Global settings (`settings.json`)
  - User commands directory (`commands/`)
  - Policy files directory (`policies/`)
  - Agents directory (`agents/`)
  - Temporary files directory (`tmp/`)
  - Binary executables directory (`tmp/bin/`)

### 2. System Settings and Policies Paths

- System-wide settings and policies paths are resolved based on the operating system platform:
  - macOS: `/Library/Application Support/GeminiCli/settings.json`
  - Windows: `C:\ProgramData\gemini-cli\settings.json`
  - Linux/Other: `/etc/gemini-cli/settings.json`
- Environment variable `GEMINI_CLI_SYSTEM_SETTINGS_PATH` can override the default system settings path.
- Policies directory is located adjacent to the system settings file.

### 3. Project-Specific Storage

- Each project is identified by its root directory path.
- Project-specific temporary directories and history directories are created under global temp and history directories, respectively.
- To ensure uniqueness and avoid collisions, project directories are named using a SHA-256 hash of the project root path.
- Project directories managed include:
  - Temporary directory (`tmp/<sha256(projectRoot)>`)
  - History directory (`history/<sha256(projectRoot)>`)
  - Extensions directory (`<projectRoot>/.gemini/extensions`)
  - Commands directory (`<projectRoot>/.gemini/commands`)
  - Agents directory (`<projectRoot>/.gemini/agents`)
  - Workspace settings file (`<projectRoot>/.gemini/settings.json`)
  - Temporary checkpoints directory (`tmp/<sha256(projectRoot)>/checkpoints`)
  - Shell history file (`tmp/<sha256(projectRoot)>/shell_history`)
  - Extensions configuration file (`<projectRoot>/.gemini/extensions/gemini-extension.json`)

### 4. Path and Directory Management

- The `Storage` class uses Node.js standard modules (`path`, `os`, `crypto`, `fs`) to handle path concatenation, environment detection, hashing, and file system operations.
- The method `ensureProjectTempDirExists()` creates the project temporary directory recursively if it does not exist.
- All path retrieval methods return absolute paths suitable for file system operations.

---

## Class: Storage

### Constructor

```typescript
constructor(targetDir: string)
```

- Initializes a `Storage` instance bound to a specific project root directory (`targetDir`).
- Instance methods operate relative to this project root.

### Static Methods (Global Scope)

- `getGlobalGeminiDir(): string`  
  Returns the global Gemini directory path under the user's home or OS temp directory.

- `getMcpOAuthTokensPath(): string`  
  Returns the path to the OAuth tokens JSON file in the global Gemini directory.

- `getGlobalSettingsPath(): string`  
  Returns the path to the global settings JSON file.

- `getInstallationIdPath(): string`  
  Returns the path to the installation ID file.

- `getGoogleAccountsPath(): string`  
  Returns the path to the Google accounts JSON file.

- `getUserCommandsDir(): string`  
  Returns the directory path for user-defined commands.

- `getGlobalMemoryFilePath(): string`  
  Returns the path to the global memory markdown file.

- `getUserPoliciesDir(): string`  
  Returns the directory path for user policy files.

- `getUserAgentsDir(): string`  
  Returns the directory path for user agents.

- `getSystemSettingsPath(): string`  
  Returns the system-wide settings file path, considering platform defaults and environment overrides.

- `getSystemPoliciesDir(): string`  
  Returns the system policies directory path adjacent to the system settings file.

- `getGlobalTempDir(): string`  
  Returns the global temporary directory path.

- `getGlobalBinDir(): string`  
  Returns the global binary executables directory path under the global temp directory.

- `getOAuthCredsPath(): string`  
  Returns the path to the OAuth credentials JSON file in the global Gemini directory.

### Instance Methods (Project Scope)

- `getGeminiDir(): string`  
  Returns the `.gemini` directory path inside the project root.

- `getProjectTempDir(): string`  
  Returns the project-specific temporary directory path under the global temp directory, named by SHA-256 hash of the project root.

- `ensureProjectTempDirExists(): void`  
  Ensures the project temporary directory exists by creating it recursively if necessary.

- `getProjectRoot(): string`  
  Returns the project root directory path.

- `getHistoryDir(): string`  
  Returns the project-specific history directory path under the global history directory, named by SHA-256 hash of the project root.

- `getWorkspaceSettingsPath(): string`  
  Returns the path to the workspace settings JSON file inside the project `.gemini` directory.

- `getProjectCommandsDir(): string`  
  Returns the commands directory path inside the project `.gemini` directory.

- `getProjectAgentsDir(): string`  
  Returns the agents directory path inside the project `.gemini` directory.

- `getProjectTempCheckpointsDir(): string`  
  Returns the checkpoints directory path inside the project temporary directory.

- `getExtensionsDir(): string`  
  Returns the extensions directory path inside the project `.gemini` directory.

- `getExtensionsConfigPath(): string`  
  Returns the path to the extensions configuration JSON file inside the extensions directory.

- `getHistoryFilePath(): string`  
  Returns the path to the shell history file inside the project temporary directory.

### Private Methods

- `getFilePathHash(filePath: string): string`  
  Computes a SHA-256 hash of the given file path string, used to generate unique directory names for project isolation.

---

## Interaction and Usage

- The `Storage` class is instantiated with a project root directory to manage project-specific paths.
- Static methods provide access to global configuration and environment paths without requiring instantiation.
- Consumers of this module use the provided path getters to read or write configuration files, manage temporary data, or access policy and credential files.
- The module encapsulates all platform-specific and environment-dependent logic, providing a consistent interface for other system domains such as Policy Management, Telemetry, Authentication, and Code Assistance.
- The unique hashing mechanism ensures that multiple projects can coexist without path conflicts in shared global directories.
- Directory existence checks and creation are handled internally to simplify caller responsibilities.

---

## Architecture Context

- The Configuration and Storage Domain supports other core domains by providing reliable and secure file system paths.
- It is a dependency for the Policy Management Domain (for policy file storage), Telemetry Domain (for exporter and log storage), and Authentication Domain (for credential storage).
- It enables multi-project support by isolating project data and configuration.
- The domain is implemented in TypeScript and leverages Node.js standard libraries for cross-platform compatibility.

---

## Summary

The Configuration and Storage Domain is a critical infrastructure component that abstracts and manages all file system paths related to configuration, credentials, temporary data, and history for the Gemini CLI Core system. Its design ensures multi-project isolation, environment compatibility, and ease of use for other system components. By centralizing path management and encapsulating file system operations, it provides a robust foundation for secure and maintainable configuration handling across the entire backend service platform.