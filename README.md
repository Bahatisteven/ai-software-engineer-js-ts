# AI Software Engineer Assignment (JS/TS)

This repository contains a bug fix for an HTTP client OAuth2 token handling issue.

## Running Tests Locally

### Prerequisites
- Node.js 18+ installed
- npm

### Steps
```bash
# Install dependencies
npm install

# Run tests
npm test
```

## Running Tests with Docker

### Prerequisites
- Docker installed

### Steps
```bash
# Build the Docker image
docker build -t ai-software-engineer-test .

# Run tests in Docker
docker run ai-software-engineer-test
```

## Assignment Completion

All assignment requirements have been completed:
- ✅ Dockerfile created for CI-style testing
- ✅ Dependencies pinned to exact versions (no ^ or ~)
- ✅ Bug identified and fixed with minimal changes
- ✅ Comprehensive tests added to reproduce and verify the fix
- ✅ README updated with run instructions
- ✅ Explanation.md created with bug analysis

## Project Structure

```
.
├── src/
│   ├── httpClient.ts    # HTTP client with OAuth2 token management
│   └── tokens.ts         # OAuth2Token class
├── tests/
│   └── httpClient.test.ts # Test suite for HttpClient
├── Dockerfile            # Docker configuration for CI
├── .dockerignore         # Docker build optimization
├── .gitignore            # Git ignore rules
├── package.json          # Dependencies (pinned versions)
├── README.md             # This file
└── Explanation.md        # Bug analysis and fix explanation
```
