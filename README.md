# Wazuh MSSP Docs

This repository contains the MkDocs-based documentation site for the Wazuh MSSP implementation guide.

## Local development

1. Create and activate a virtual environment
   ```powershell
   python -m venv venv
   .\venv\Scripts\Activate.ps1
   ```
2. Install dependencies
   ```powershell
   pip install -r requirements.txt
   ```
3. Preview locally
   ```powershell
   mkdocs serve
   ```
4. Build the site
   ```powershell
   mkdocs build
   ```

## Deployment

The generated site output is typically built into the `site/` directory and can be published using any static site hosting platform.
