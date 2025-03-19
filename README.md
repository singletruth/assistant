# Singletruth Assistant

A customized AI chat interface based on Open WebUI, rebranded for Singletruth.

## Modifications Made

This repository contains a fully rebranded version of Open WebUI with the following changes:

### 1. Name Rebranding

All occurrences of "Open WebUI" have been replaced with "Singletruth Assistant" throughout the codebase:

- **Core Configuration Files**:
  - `src/lib/constants.ts` - Changed APP_NAME constant
  - `backend/open_webui/env.py` - Changed default WEBUI_NAME and condition logic
  - `src/app.html` - Changed HTML metadata and titles
  - `src/routes/+layout.svelte` - Updated notification strings

- **Application Files**:
  - `backend/open_webui/main.py` - Updated application description
  - `backend/open_webui/__init__.py` - Updated version output
  - Modified version check messages in function and tools creation/edit pages

- **Resource Files**:
  - `static/opensearch.xml` - Changed search description
  - Updated English translation file to change app name references
  - Updated community-related strings

- **Other References**:
  - Updated GitHub issue template for version information

### 2. Logo & Visual Branding

All logo and favicon files have been replaced throughout the application:

- `/static/favicon.png`
- `/static/favicon-96x96.png`
- `/static/favicon.svg`
- `/static/favicon.ico`
- `/static/favicon-dark.png`
- `/static/logo.png`
- `/static/splash.png`
- `/static/splash-dark.png`
- `/static/apple-touch-icon.png`
- `/static/web-app-manifest-192x192.png`
- `/static/web-app-manifest-512x512.png`
- `/backend/open_webui/static/` - All logo files in this directory

### 3. Translation Updates

- Updated the main English translation file to reflect the new branding in:
  - Version messages
  - Community references
  - Feature descriptions

## Branding Details

The new branding uses a modern teal/dark green and light blue waveform-like pattern to convey voice and AI assistance capabilities. The color scheme reflects Singletruth's brand identity.

## Original Project

This application is based on [Open WebUI](https://github.com/open-webui/open-webui), an open-source web interface for Ollama and various AI models.