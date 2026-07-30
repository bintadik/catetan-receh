# GitHub Actions: Multi-line `.env` Secrets to Python

This guide explains how to store a complete multi-line `.env` block inside a single GitHub secret, inject it into the runner environment, and read the individual keys using Python.

---

## 1. Store the GitHub Secret (`DOTENV_SECRET`)
In your GitHub repository under **Settings > Secrets and variables > Actions**, create a new repository secret named `DOTENV_SECRET`. 

Paste the exact multi-line `.env` content into the value field:

```env
DATABASE_URL=postgres://user:pass@localhost:5432/db
API_KEY=super-secret-key-123
DEBUG_MODE=False
```

---

## 2. GitHub Actions Workflow Configuration
Use a quoted multi-line delimiter (`'EOF'`) to append the entire secret safely into `$GITHUB_ENV`. This prevents bash from evaluating raw special characters.

```yaml
name: Run Python Script with Env Secrets

on: [push]

jobs:
  run-app:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Load all secrets into environment
        run: |
          cat << 'EOF' >> $GITHUB_ENV
          ${{ secrets.DOTENV_SECRET }}
          EOF

      - name: Execute Python Script
        run: python main.py
```

---

## 3. Read Environment Variables in Python (`main.py`)
Once GitHub Actions appends the lines to `$GITHUB_ENV`, they become standard environment variables. You do not need to parse the block manually; use `os.environ` or `os.getenv()`.

```python
import os

# Fetch the variables injected by the GitHub workflow
db_url = os.getenv("DATABASE_URL")
api_key = os.getenv("API_KEY")
debug = os.getenv("DEBUG_MODE")

# Use the variables in your logic
print(f"Connecting to database: {db_url}")
print(f"API Key successfully loaded: {bool(api_key)}")
print(f"Debug mode is: {debug}")
```
