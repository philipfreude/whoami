# Setup Guide for Forked Repository

## Secrets Configuration

Good news! **No manual secrets setup is required** after forking this repository. 🎉

### What Changed

The workflow now uses GitHub's automatic `GITHUB_TOKEN` which is provided automatically to all GitHub Actions workflows. This token has the necessary permissions to:

1. **Create GitHub Releases** - GoReleaser will create releases with binaries
2. **Push Docker Images to GHCR** - Images will be published to `ghcr.io/philipfreude/whoami`

### Permissions

The workflow has been configured with the following permissions in `.github/workflows/release.yml`:

```yaml
permissions:
  contents: write    # Allows creating releases and tags
  packages: write    # Allows pushing to GitHub Container Registry
```

These permissions are automatically granted to the `GITHUB_TOKEN` when the workflow runs.

### How It Works

When you push a tag (e.g., `v1.0.0`), the release workflow will:

1. ✅ Authenticate to GHCR using `${{ github.actor }}` (your username) and `${{ secrets.GITHUB_TOKEN }}`
2. ✅ Build multi-architecture Docker images (amd64, arm64, armv7)
3. ✅ Push images to `ghcr.io/philipfreude/whoami` with tags: `latest`, `<version>`, and `v<major>.<minor>`
4. ✅ Create a GitHub release with compiled binaries for multiple platforms

### Testing the Release Workflow

To test the workflow:

```bash
# Create and push a tag
git tag v1.0.0
git push origin v1.0.0
```

Then check:
- **GitHub Actions**: https://github.com/philipfreude/whoami/actions
- **Releases**: https://github.com/philipfreude/whoami/releases
- **Packages**: https://github.com/philipfreude?tab=packages

### No Secrets Required! ✨

Previously, the workflow required:
- ❌ `DOCKER_USERNAME` and `DOCKER_PASSWORD` (for DockerHub - removed)
- ❌ `GHCR_TOKEN` (for GHCR - now uses automatic token)
- ❌ `GH_TOKEN_REPO` (for releases - now uses automatic token)

Now everything uses the built-in `GITHUB_TOKEN` with no manual configuration needed!

## Why Did the Original Repo Use Custom Secrets?

### Historical Context

The original Traefik repository used custom secrets for several reasons:

1. **Organization Account**: Traefik is an **organization account**, not a personal account. They needed to:
   - Use a specific user account (`traefiker`) for GHCR authentication
   - Use custom tokens with controlled permissions across the organization
   - Publish to DockerHub (which requires username/password)

2. **Cross-Repository Access**: They may have used a Personal Access Token (PAT) with broader permissions to:
   - Access organization-level secrets
   - Publish to multiple repositories
   - Maintain consistent credentials across projects

3. **Fine-Grained Control**: Custom tokens allow:
   - Rotating credentials without changing workflow files
   - Different permission scopes for different operations
   - Audit trails for specific service accounts

### Security Considerations for Your Fork

#### ✅ **Using `GITHUB_TOKEN` is MORE Secure for Personal Forks**

For your personal fork, using the automatic `GITHUB_TOKEN` is actually **more secure** because:

1. **Automatic Rotation**: The token is generated fresh for each workflow run and expires when the job completes
2. **Scoped Permissions**: Limited to only the permissions defined in the workflow (contents: write, packages: write)
3. **No Credential Storage**: No need to create, store, or rotate Personal Access Tokens
4. **Reduced Attack Surface**: If someone compromises your repository, they can't steal long-lived tokens
5. **Simpler Management**: No secrets to forget about or accidentally expose

#### ⚠️ **When You MIGHT Need Custom Secrets**

You would only need custom secrets if:

1. **Publishing to DockerHub**: You're also publishing to `docker.io` (not just GHCR)
   - Requires: `DOCKER_USERNAME` and `DOCKER_PASSWORD` or Docker access token

2. **Cross-Repository Access**: Your workflow needs to access other private repositories
   - Requires: Personal Access Token with `repo` scope

3. **Organization Requirements**: You're part of an organization with specific credential policies
   - Requires: Following organization security guidelines

4. **Third-Party Services**: Integrating with external services (Slack, AWS, etc.)
   - Requires: Service-specific credentials

5. **Long-Lived Deployments**: Need tokens that persist beyond the workflow run
   - Requires: Personal Access Token or service account token

### Security Best Practices

#### For This Repository (Current Setup) ✅

- ✅ Use automatic `GITHUB_TOKEN` for GHCR and releases
- ✅ Define minimal permissions in workflow (`contents: write`, `packages: write`)
- ✅ Only trigger on tags (not on every push)
- ✅ Use official GitHub Actions from trusted publishers

#### If You Need Custom Secrets 🔐

- 🔐 Use **fine-grained Personal Access Tokens** instead of classic tokens
- 🔐 Set **minimum required permissions** (don't use `repo` if you only need `packages:write`)
- 🔐 Set **token expiration** dates and rotate regularly
- 🔐 Use **organization secrets** for shared credentials
- 🔐 Enable **secret scanning** in repository settings
- 🔐 Never commit secrets to git (use `.gitignore` for local env files)

### Comparison: GITHUB_TOKEN vs Personal Access Token

| Feature | `GITHUB_TOKEN` | Personal Access Token |
|---------|----------------|----------------------|
| **Lifetime** | Single workflow run | Until manually revoked |
| **Permissions** | Defined in workflow | Defined at token creation |
| **Rotation** | Automatic per run | Manual |
| **Storage** | No storage needed | Store in Secrets |
| **Scope** | Repository-only | User/org-wide |
| **Security Risk** | Minimal | Medium to High |
| **Use Case** | Same repository operations | Cross-repo, external services |

### Recommendation

**For your forked personal repository**: Stick with the automatic `GITHUB_TOKEN` approach. It's simpler, more secure, and sufficient for:
- ✅ Publishing Docker images to GHCR
- ✅ Creating GitHub releases
- ✅ Building and testing in CI/CD

Only add custom secrets if you have specific requirements (like publishing to DockerHub).

### Troubleshooting

If you encounter permission issues:

1. **Check workflow permissions** in your repository settings:
   - Go to: Settings → Actions → General → Workflow permissions
   - Ensure "Read and write permissions" is selected
   - Make sure "Allow GitHub Actions to create and approve pull requests" is checked (if needed)

2. **Verify package visibility**:
   - After the first release, check your packages at: https://github.com/philipfreude?tab=packages
   - You may need to make the package public if you want it publicly accessible

3. **Check Actions logs**:
   - Go to: Actions tab → Select the failed workflow → View logs for detailed error messages
