# Development Container Setup

This development container provides a development environment with PostgreSQL and Node.js, along with pre-configured VS Code settings and extensions. It's built on Ubuntu 22.04 and includes essential development tools.

## 🛠 What's Included

### Base Environment
- Ubuntu 22.04 base image
- PostgreSQL ${POSTGRES_VERSION} with contrib packages
- Node.js 20.18.1 (managed via NVM 0.39.7)
- Python 3 with pip
- Basic development tools:
  - git
  - curl
  - wget
  - build-essential
  - vim/nano editors
  - openssh-client
  - SQLite3

### VS Code Extensions
- TypeScript Next
- Prettier
- Auto Rename Tag
- JSON Tools
- YAML Tools
- PostgreSQL

### Development Tools
- Claude Code CLI (installed automatically)
- Task Master AI
- Context7 MCP integration

## 🔌 Port Forwarding
The following ports are automatically forwarded:
- 3000 (Frontend development)
- 3001 (Backend development)
- 5432 (PostgreSQL)
- 8080 (Alternative HTTP)
- 5173 (Vite development server)

## 📋 Prerequisites

- Docker installed on your host machine
- Visual Studio Code with Remote - Containers extension

## 🚀 Getting Started

1. Open the project in Visual Studio Code
2. When prompted, click "Reopen in Container" or run the "Remote-Containers: Reopen in Container" command from the command palette
3. Wait for the container to build and start
4. The container will automatically:
   - Start PostgreSQL service
   - Install Claude Code CLI and MCPs
   - Configure the development environment

## 💾 PostgreSQL Configuration

The container comes with a basic PostgreSQL setup:
- User: `vscode`
- Password: `password`
- Database: `vscode`
- Host: `localhost`
- Connection URL: `postgresql://vscode:password@localhost/vscode`

The PostgreSQL service automatically starts when the container launches.

## 🔧 Customization

### PostgreSQL Version
To change the PostgreSQL version:
1. Open `.devcontainer/Dockerfile`
2. Modify the `POSTGRES_VERSION` argument:
```dockerfile
ARG POSTGRES_VERSION=15
```

### Node.js Version
To change the Node.js version:
1. Open `.devcontainer/Dockerfile`
2. Modify the `NODE_VERSION` environment variable:
```dockerfile
ENV NODE_VERSION=20.18.1
```

### VS Code Settings
The container comes with pre-configured VS Code settings:
- Default terminal: bash (login shell)
- Workspace mount: Cached consistency for better performance
- Remote user: vscode (non-root)

## 📝 Important Notes

- The container runs as a non-root user `vscode` with sudo privileges
- The workspace is mounted at `/workspace` with cached consistency
- PostgreSQL is configured to trust local connections from the `vscode` user
- NVM is pre-configured in the user's `.bashrc`
- Environment variables are pre-configured including `DATABASE_URL` and extended `PATH`
- `.git`, `node_modules`, `.env*`, and log files are excluded from the container build

## 🔍 Verifying Setup

1. Check PostgreSQL status:
```bash
sudo service postgresql status
```

2. Verify Node.js installation:
```bash
node --version
```

3. Test database connection:
```bash
psql -d vscode
```

4. Verify Claude CLI:
```bash
claude --version
```

## 🛟 Troubleshooting

If PostgreSQL fails to start:
```bash
sudo service postgresql restart
```

If Node.js is not found:
```bash
source ~/.bashrc
```

If Claude CLI tools aren't working:
```bash
npm install -g @anthropic-ai/claude-code
npm install -g task-master-ai@latest
claude mcp add taskmaster "$(which task-master-ai)"
claude mcp add --transport sse context7 https://mcp.context7.com/sse
``` 