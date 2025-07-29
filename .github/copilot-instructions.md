# MVP Library - Copilot Instructions

## Architecture Overview

MVP Library is a Node.js/Express app that creates a collaborative documentation site powered by Google Docs. It's a fork of the NYT Library, serving Google Drive documents as web pages with authentication, caching, and customization capabilities.

**Core Components:**

- **`server/index.js`**: Main Express app with middleware chain and route setup
- **`server/list.js`**: Google Drive traversal and file metadata management (tree structure cached in memory)
- **`server/docs.js`**: Google Docs/Sheets fetching, HTML processing, and content caching
- **`server/auth.js`**: Google API authentication using service accounts or OAuth
- **`server/userAuth.js`**: User authentication (Google OAuth or Slack OAuth) with domain authorization
- **`server/cache.js`**: Document caching with edit-detection and purge capabilities

## Key Patterns & Conventions

### MVP Customization Approach

**MVP Library uses direct modifications to the base library files rather than the custom override system:**

- Core functionality is modified directly in `server/` and `layouts/` directories
- This approach provides more direct control and easier maintenance for MVP's use case
- Key modifications include Google Doc template support in `layouts/partials/footer.ejs`

### Google Doc Template Feature

- **Template Support**: The "Create New Page" button supports using predefined Google Doc templates
- **Environment Variable**: Set `GOOGLE_DOC_TEMPLATE_ID` to enable template-based document creation
- **Fallback**: When not set, creates blank documents (original behavior)
- **Implementation**: Modified `layouts/partials/footer.ejs` to check for template ID and adjust creation URL

### Environment-Based Behavior

- **Development mode** (`NODE_ENV=development`): Bypasses user authorization, allows any user
- **Production mode**: Enforces `APPROVED_DOMAINS` email/domain restrictions
- **Trust proxy** (`TRUST_PROXY=true`): Enables HTTPS redirects behind proxies

### Google Drive Integration

- Supports both Team Drives (`DRIVE_TYPE=team`) and shared folders (`DRIVE_TYPE=folder`)
- Tree structure refreshed every 60s (configurable via `LIST_UPDATE_DELAY`)
- Document paths generated from folder hierarchy: `/folder-name/document-name`

## Development Workflow

### Setup Commands

```bash
npm install --no-optional
npm run build        # Compile Sass to CSS
npm run watch        # Development server with hot reload
npm test            # Run functional and unit tests
npm run test:cover  # Generate coverage report
```

### Essential Environment Variables

```bash
NODE_ENV=development                    # Enables local development mode
GOOGLE_CLIENT_ID=...                   # OAuth client ID
GOOGLE_CLIENT_SECRET=...               # OAuth client secret
APPROVED_DOMAINS="domain1.com,domain2.com"  # Authorized email domains (supports regex)
DRIVE_TYPE=folder                      # "team" or "folder"
DRIVE_ID=ABC123                       # Google Drive/folder ID
SESSION_SECRET=supersecret            # Session encryption key
```

### Authentication Files

- **Service account**: Place credentials in `server/.auth.json` for local development
- **Production**: Set `GOOGLE_APPLICATION_CREDENTIALS=parse_json` and `GOOGLE_APPLICATION_JSON` with service account JSON

## Critical Integrations

### Caching Strategy

- **Memory cache** by default (`server/cache/store.js`)
- **Document cache**: Keyed by Google Doc ID, invalidated on modification time changes
- **Edit mode**: `?edit=1` bypasses cache for 1 hour, `?purge=1` force-refreshes cache
- **Custom cache**: Override with `custom/cache/store.js` (must export `get()`/`set()` methods)

### Document Processing

- **HTML export**: Google Docs exported as HTML, processed by `server/formatter.js`
- **Spreadsheets**: Converted to HTML tables via `xlsx` library
- **Clean-up**: Empty tables removed, content sanitized through Cheerio

### URL Structure

- **Documents**: `/folder-path/document-name` (slugified from Google Drive hierarchy)
- **Categories**: `/category-name` (folder listing pages)
- **Special routes**: `/search`, `/reading-history`, `/view-on-site/:docId`

## Testing & Debugging

### Test Structure

- **Functional tests**: `test/functional/` - Route and authentication testing
- **Unit tests**: `test/unit/` - Individual component testing
- **Mocks**: `test/utils/__mocks__/` - Google API mocking for tests
- **Bootstrap**: `test/utils/bootstrap.js` - Test environment setup

### Common Debug Patterns

- **Cache issues**: Check `?edit=1` or `?purge=1` query parameters
- **Auth problems**: Verify `APPROVED_DOMAINS` and OAuth credentials
- **Drive access**: Ensure service account has access to specified `DRIVE_ID`
- **Template errors**: Check both `layouts/` and `custom/layouts/` for template conflicts

## Deployment (Google App Engine)

Configured via `app.yaml` with environment variables. Key differences from local:

- Uses `GOOGLE_APPLICATION_CREDENTIALS=parse_json` with JSON in environment
- `TRUST_PROXY=true` for HTTPS handling
- Production OAuth URLs and approved domains
