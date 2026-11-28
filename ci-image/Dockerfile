# syntax=docker/dockerfile:1
# CI image (DEL-CI-003): every repository's CI runs `just` recipes inside it,
# and `just` on a developer machine runs them inside it too (DEL-TOOL-004).
# Base: the Frappe bench image (Python, Node, yarn, bench, MariaDB client).
# Every version is pinned (SEC-VULN-005); Renovate updates the pins through the
# `# renovate:` comments. The image tag is a hash of this file (`just ci-tag`).
FROM docker.io/frappe/bench:latest@sha256:67bd9140a0b9240a399ebf25033a3faa1b41a0be4468193f449193fb95df1d01

# root (numeric: DL3066)
USER 0
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# renovate: datasource=github-releases depName=casey/just
ARG JUST_VERSION=1.59.0
# renovate: datasource=github-releases depName=gitleaks/gitleaks
ARG GITLEAKS_VERSION=8.30.1
# renovate: datasource=github-releases depName=lycheeverse/lychee extractVersion=^lychee-v(?<version>.*)$
ARG LYCHEE_VERSION=0.24.2
# renovate: datasource=github-releases depName=hadolint/hadolint
ARG HADOLINT_VERSION=2.15.1
# renovate: datasource=github-releases depName=aquasecurity/trivy
ARG TRIVY_VERSION=0.76.2
# renovate: datasource=github-releases depName=anchore/syft
ARG SYFT_VERSION=1.57.2
# renovate: datasource=github-releases depName=sigstore/cosign
ARG COSIGN_VERSION=3.1.6
# renovate: datasource=github-releases depName=orhun/git-cliff
ARG GIT_CLIFF_VERSION=2.14.2
# renovate: datasource=github-releases depName=astral-sh/uv
ARG UV_VERSION=0.13.10
# renovate: datasource=github-releases depName=koalaman/shellcheck
ARG SHELLCHECK_VERSION=0.11.0
# renovate: datasource=docker depName=docker
ARG DOCKER_VERSION=29.10.1
# renovate: datasource=github-releases depName=docker/buildx
ARG BUILDX_VERSION=0.39.1
# renovate: datasource=pypi depName=pre-commit
ARG PRE_COMMIT_VERSION=4.7.0
# renovate: datasource=pypi depName=mypy
ARG MYPY_VERSION=2.4.1
# renovate: datasource=pypi depName=pydantic
ARG PYDANTIC_VERSION=2.14.4
# renovate: datasource=pypi depName=import-linter
ARG IMPORT_LINTER_VERSION=2.17
# renovate: datasource=npm depName=markdownlint-cli2
ARG MARKDOWNLINT_VERSION=0.24.2

# hadolint ignore=DL3008
RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates curl git jq openssh-client xz-utils \
 && rm -rf /var/lib/apt/lists/*

# Binary tools into /usr/local/bin (downloaded in a temporary folder).
# hadolint ignore=DL3003
RUN set -eux; \
    case "$(uname -m)" in \
      x86_64) A=x86_64; G=x64; D=amd64; T=64bit ;; \
      aarch64) A=aarch64; G=arm64; D=arm64; T=ARM64 ;; \
      *) echo "unsupported architecture $(uname -m)"; exit 1 ;; \
    esac; \
    gh_rel=https://github.com; tmp=$(mktemp -d); mkdir "$tmp/l"; cd "$tmp"; \
    curl -fsSL "$gh_rel/casey/just/releases/download/${JUST_VERSION}/just-${JUST_VERSION}-${A}-unknown-linux-musl.tar.gz" | tar -xz just; \
    curl -fsSL "$gh_rel/gitleaks/gitleaks/releases/download/v${GITLEAKS_VERSION}/gitleaks_${GITLEAKS_VERSION}_linux_${G}.tar.gz" | tar -xz gitleaks; \
    curl -fsSL "$gh_rel/lycheeverse/lychee/releases/download/lychee-v${LYCHEE_VERSION}/lychee-${A}-unknown-linux-gnu.tar.gz" | tar -xz -C "$tmp/l" ; mv "$(find "$tmp/l" -type f -name lychee)" .; \
    H=$A; [ "$A" = aarch64 ] && H=arm64; \
    curl -fsSL -o hadolint "$gh_rel/hadolint/hadolint/releases/download/v${HADOLINT_VERSION}/hadolint-linux-${H}"; \
    curl -fsSL "$gh_rel/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-${T}.tar.gz" | tar -xz trivy; \
    curl -fsSL "$gh_rel/anchore/syft/releases/download/v${SYFT_VERSION}/syft_${SYFT_VERSION}_linux_${D}.tar.gz" | tar -xz syft; \
    curl -fsSL -o cosign "$gh_rel/sigstore/cosign/releases/download/v${COSIGN_VERSION}/cosign-linux-${D}"; \
    curl -fsSL "$gh_rel/orhun/git-cliff/releases/download/v${GIT_CLIFF_VERSION}/git-cliff-${GIT_CLIFF_VERSION}-${A}-unknown-linux-gnu.tar.gz" | tar -xz --strip-components=1 "git-cliff-${GIT_CLIFF_VERSION}/git-cliff"; \
    curl -fsSL "$gh_rel/astral-sh/uv/releases/download/${UV_VERSION}/uv-${A}-unknown-linux-gnu.tar.gz" | tar -xz --strip-components=1; \
    curl -fsSL "$gh_rel/koalaman/shellcheck/releases/download/v${SHELLCHECK_VERSION}/shellcheck-v${SHELLCHECK_VERSION}.linux.${A}.tar.xz" | tar -xJ --strip-components=1 "shellcheck-v${SHELLCHECK_VERSION}/shellcheck"; \
    curl -fsSL "https://download.docker.com/linux/static/stable/${A}/docker-${DOCKER_VERSION}.tgz" | tar -xz --strip-components=1 docker/docker; \
    mkdir -p /usr/local/lib/docker/cli-plugins; \
    curl -fsSL -o /usr/local/lib/docker/cli-plugins/docker-buildx "$gh_rel/docker/buildx/releases/download/v${BUILDX_VERSION}/buildx-v${BUILDX_VERSION}.linux-${D}"; \
    chmod 755 /usr/local/lib/docker/cli-plugins/docker-buildx; \
    install -m 755 just gitleaks lychee hadolint trivy syft cosign git-cliff uv uvx shellcheck docker /usr/local/bin/; \
    cd /; rm -rf "$tmp"

# Python tools, one isolated environment each (mypy with the pydantic plugin).
ENV UV_TOOL_DIR=/opt/uv-tools UV_TOOL_BIN_DIR=/usr/local/bin
RUN uv tool install "pre-commit==${PRE_COMMIT_VERSION}" \
 && uv tool install "mypy==${MYPY_VERSION}" --with "pydantic==${PYDANTIC_VERSION}" \
 && uv tool install "import-linter==${IMPORT_LINTER_VERSION}"

# CI checkouts belong to another user than the job.
RUN git config --system --add safe.directory '*'

# Playwright's system libraries (browsers come with each project's lockfile).
RUN npx -y playwright install-deps chromium && rm -rf /var/lib/apt/lists/*

# frappe (UID 1000 in the Frappe images)
USER 1000
RUN npm install -g "markdownlint-cli2@${MARKDOWNLINT_VERSION}" && npm cache clean --force

# Recipes check this to know they already run inside the CI image.
ENV FRAPPE_CI_IMAGE=1
WORKDIR /home/frappe
