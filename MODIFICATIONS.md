# Code Modifications

Below are the specific code changes made to rebrand Open WebUI to Singletruth Assistant.

## 1. Core Constants and Environment Variables

### src/lib/constants.ts
```typescript
// Changed from:
export const APP_NAME = 'Open WebUI';

// Changed to:
export const APP_NAME = 'Singletruth Assistant';
```

### backend/open_webui/env.py
```python
# Changed from:
WEBUI_NAME = os.environ.get("WEBUI_NAME", "Open WebUI")
if WEBUI_NAME != "Open WebUI":
    WEBUI_NAME += " (Open WebUI)"

# Changed to:
WEBUI_NAME = os.environ.get("WEBUI_NAME", "Singletruth Assistant")
if WEBUI_NAME != "Singletruth Assistant" and "Singletruth Assistant" not in WEBUI_NAME:
    WEBUI_NAME += " (Singletruth Assistant)"
```

## 2. HTML Metadata and Title

### src/app.html
```html
<!-- Changed from: -->
<meta name="apple-mobile-web-app-title" content="Open WebUI" />
<meta name="description" content="Open WebUI" />
<link rel="search" type="application/opensearchdescription+xml" title="Open WebUI" href="/opensearch.xml" />
<title>Open WebUI</title>

<!-- Changed to: -->
<meta name="apple-mobile-web-app-title" content="Singletruth Assistant" />
<meta name="description" content="Singletruth Assistant" />
<link rel="search" type="application/opensearchdescription+xml" title="Singletruth Assistant" href="/opensearch.xml" />
<title>Singletruth Assistant</title>
```

## 3. Notification Messages

### src/routes/+layout.svelte
```javascript
// Changed from:
new Notification(`${title} | Open WebUI`, {
  body: content,
  icon: `${WEBUI_BASE_URL}/static/favicon.png`
});

// Changed to:
new Notification(`${title} | ${$WEBUI_NAME}`, {
  body: content,
  icon: `${WEBUI_BASE_URL}/static/favicon.png`
});
```

## 4. Version Check Messages

### src/routes/(app)/admin/functions/create/+page.svelte, src/routes/(app)/admin/functions/edit/+page.svelte, src/routes/(app)/workspace/tools/create/+page.svelte, src/routes/(app)/workspace/tools/edit/+page.svelte
```javascript
// Changed from:
'Open WebUI version (v{{OPEN_WEBUI_VERSION}}) is lower than required version (v{{REQUIRED_VERSION}})'

// Changed to:
'Singletruth Assistant version (v{{OPEN_WEBUI_VERSION}}) is lower than required version (v{{REQUIRED_VERSION}})'
```

## 5. Backend Description

### backend/open_webui/main.py
```python
# Changed from:
"description": "Open WebUI is an open, extensible, user-friendly interface for AI that adapts to your workflow."

# Changed to:
"description": "Singletruth Assistant is an open, extensible, user-friendly interface for AI that adapts to your workflow."
```

## 6. Version Output

### backend/open_webui/__init__.py
```python
# Changed from:
typer.echo(f"Open WebUI version: {VERSION}")

# Changed to:
typer.echo(f"Singletruth Assistant version: {VERSION}")
```

## 7. Translation Updates

### src/lib/i18n/locales/en-US/translation.json
```json
// Changed from:
"Open WebUI uses faster-whisper internally.": "",
"Open WebUI uses SpeechT5 and CMU Arctic speaker embeddings.": "",
"Open WebUI version (v{{OPEN_WEBUI_VERSION}}) is lower than required version (v{{REQUIRED_VERSION}})": "",
"Made by Open WebUI Community": "",
"Redirecting you to Open WebUI Community": "",
"Share to Open WebUI Community": "",

// Changed to:
"Singletruth Assistant uses faster-whisper internally.": "",
"Singletruth Assistant uses SpeechT5 and CMU Arctic speaker embeddings.": "",
"Singletruth Assistant version (v{{OPEN_WEBUI_VERSION}}) is lower than required version (v{{REQUIRED_VERSION}})": "",
"Made by Singletruth Assistant Community": "",
"Redirecting you to Singletruth Assistant Community": "",
"Share to Singletruth Assistant Community": "",
```

## 8. OpenSearch XML

### static/opensearch.xml
```xml
<!-- Changed from: -->
<ShortName>Open WebUI</ShortName>
<Description>Search Open WebUI</Description>

<!-- Changed to: -->
<ShortName>Singletruth Assistant</ShortName>
<Description>Search Singletruth Assistant</Description>
```

## 9. Logo & Favicon Files

All logo and favicon files were replaced with new ones featuring a modern teal/dark green and light blue waveform-like pattern. The files were processed using macOS' built-in `sips` command to generate various sizes and formats needed by the application.