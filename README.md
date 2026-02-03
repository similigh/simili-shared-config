# Simili-Bot Shared Configuration

Central configuration repository for Simili-Bot integration across Event-Integrator projects in the `similigh` organization.

## Purpose

This repository hosts:
- **Reusable GitHub Actions workflow** for issue triage and duplicate detection
- **Central Simili-Bot configuration** for all Event-Integrator repositories
- **Organization secrets management** for cross-repo access
- **Documentation and setup guides** for new users

## What is Simili-Bot?

Simili-Bot is an intelligent issue management system that uses vector embeddings to detect similar and duplicate issues across your repositories. When a new issue is created, it automatically searches for related issues and comments with links to similar discussions, helping teams avoid duplicate work and improve issue organization.

## Quick Start

### 1. Using This Shared Workflow

Each Event-Integrator repository uses this central workflow for issue triage. To enable Simili-Bot in a repository:

1. Create `.github/workflows/issue-triage.yml`:
   ```yaml
   name: Issue Triage
   on:
     issues:
       types: [opened, edited, closed, reopened, deleted]
   jobs:
     call-simili-workflow:
       uses: similigh/simili-shared-config/.github/workflows/simili.yml@main
       with:
         config_path: .github/simili.yaml
       secrets:
         GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
         QDRANT_URL: ${{ secrets.QDRANT_URL }}
         QDRANT_API_KEY: ${{ secrets.QDRANT_API_KEY }}
   ```

2. Create `.github/simili.yaml` (or use the organization-level config)

3. Enable the workflow in your repository settings

### 2. Organization Secrets

Configure these secrets at the organization level for all repositories:
- `GEMINI_API_KEY` - Google Gemini API key for embeddings
- `QDRANT_URL` - Qdrant vector database URL
- `QDRANT_API_KEY` - Qdrant authentication key

Set visibility to **All repositories** to allow all repos in the organization to access these secrets.

### 3. Repository Configuration

Each repository should have a `.github/simili.yaml` file. Minimal example:
```yaml
qdrant:
  url: "${QDRANT_URL}"
  api_key: "${QDRANT_API_KEY}"
  collection: "event-int-index"

embedding:
  provider: "gemini"
  api_key: "${GEMINI_API_KEY}"

defaults:
  similarity_threshold: 0.70
  max_similar_to_show: 5
  cross_repo_search: true
```

## Configuration Reference

### Qdrant Settings
- `url` - Qdrant instance URL (use environment variable)
- `api_key` - Authentication key (use environment variable)
- `collection` - Vector collection name for storing embeddings

### Embedding Settings
- `provider` - Embedding service (currently supports "gemini")
- `api_key` - API key for embedding provider

### Defaults
- `similarity_threshold` - Minimum similarity score (0-1) to report issues
- `max_similar_to_show` - Maximum number of similar issues to display
- `cross_repo_search` - Enable searching across multiple repositories

### Repository List
Define which repositories Simili-Bot monitors:
```yaml
repositories:
  - org: "similigh"
    repo: "event-integrator-cli"
    enabled: true
    labels: ["cli"]
```

## Indexing Issues

After creating issues, index them to the Qdrant database:

```bash
# Install simili-bot as gh extension (if not already installed)
gh extension install similigh/simili-bot

# Index issues for a repository
gh simili index --repo similigh/event-integrator-cli \
  --config path/to/simili.yaml \
  --workers 5
```

This creates embeddings for all issues and enables similar issue detection.

## Testing Simili-Bot

1. Create a new issue in any monitored repository
2. The issue-triage workflow will trigger automatically
3. Simili-Bot will search for similar issues and comment with results
4. Look for a comment from Simili-Bot with links to similar discussions

Example comment:
```
Found 3 similar issues:
- #15: Add shell autocomplete support (92% similar)
- #8: Implement progress bar for long-running operations (85% similar)
- #3: Support interactive mode for CLI (78% similar)
```

## Troubleshooting

### Workflow Not Triggering
- Verify the workflow file is in `.github/workflows/` directory
- Check repository Actions are enabled in settings
- Verify secrets are set at organization level
- Check workflow file syntax: `gh workflow view issue-triage`

### No Similar Issues Found
- Ensure issues have been indexed: `gh simili list-collections`
- Check Qdrant connectivity: Verify `QDRANT_URL` and `QDRANT_API_KEY` are correct
- Review similarity threshold in config (too high threshold = fewer results)
- Index more issues for better coverage

### Embedding Errors
- Verify `GEMINI_API_KEY` is valid and has API access enabled
- Check API quota hasn't been exceeded
- Ensure key has appropriate permissions in Google Cloud console

## Architecture

```
similigh/simili-shared-config (this repo)
├── .github/
│   ├── workflows/
│   │   └── simili.yml (Reusable workflow)
│   └── simili.yaml (Central config)
└── README.md (This file)

Each Event-Integrator repo
├── .github/
│   ├── workflows/
│   │   └── issue-triage.yml (Calls shared workflow)
│   └── simili.yaml (Repo-specific overrides)
└── [repo contents]
```

## Useful Resources

- [Simili-Bot GitHub](https://github.com/similigh/simili-bot)
- [Qdrant Documentation](https://qdrant.tech/documentation/)
- [Google Gemini API](https://ai.google.dev/)
- [GitHub Actions Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

## Support

For issues with Simili-Bot:
- Check [Simili-Bot Issues](https://github.com/similigh/simili-bot/issues)
- Review the troubleshooting guide above
- Check organization secrets configuration

For Event-Integrator issues:
- See individual repository issue trackers
