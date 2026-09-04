FROM public.ecr.aws/x8v8d7g8/mars-base:latest
WORKDIR /app

# Git time-travel: clone, then make the repo's default branch point AT the base
# commit with no future history — a real branch checkout (not a detached HEAD),
# future commits/tags gc'd away so the reference solution can't leak from history.
ARG BASE_SHA=554f368809e0d891a699667faf0cfbb20057eeb2
RUN git clone https://github.com/Kludex/starlette . \
 && DEFAULT="$(git remote show origin | sed -n 's/.*HEAD branch: //p')" \
 && git checkout -B "$DEFAULT" "$BASE_SHA" \
 && git remote remove origin \
 && for b in $(git for-each-ref --format='%(refname:short)' refs/heads | grep -vx "$DEFAULT"); do git branch -D "$b" || true; done \
 && for t in $(git tag); do git merge-base --is-ancestor "$t" HEAD 2>/dev/null || git tag -d "$t"; done \
 && git reflog expire --expire=now --all \
 && git gc --prune=now \
 && { test ! -s .gitmodules || git submodule update --init --recursive; }
# No lockfile upstream: pinned with `uv pip compile (as of 2024-03-05; inputs: pyproject.toml, requirements.txt)`,
# i.e. every release newer than the base commit is excluded, resolved for Linux / CPython 3.12
# (the mars-base default interpreter), so the image matches what upstream CI saw at this commit.
RUN pip install --no-cache-dir \
        "anyio==4.3.0" \
        "attrs==23.2.0" \
        "babel==2.14.0" \
        "build==1.1.1" \
        "certifi==2024.2.2" \
        "cffi==1.16.0" \
        "charset-normalizer==3.3.2" \
        "click==8.1.7" \
        "colorama==0.4.6" \
        "coverage==7.4.3" \
        "cryptography==42.0.5" \
        "docutils==0.20.1" \
        "ghp-import==2.1.0" \
        "h11==0.14.0" \
        "httpcore==1.0.4" \
        "httpx==0.27.0" \
        "idna==3.6" \
        "importlib-metadata==7.0.1" \
        "iniconfig==2.0.0" \
        "itsdangerous==2.1.2" \
        "jaraco-classes==3.3.1" \
        "jeepney==0.8.0" \
        "jinja2==3.1.3" \
        "keyring==24.3.1" \
        "markdown-it-py==3.0.0" \
        "markdown==3.5.2" \
        "markupsafe==2.1.5" \
        "mdurl==0.1.2" \
        "mergedeep==1.3.4" \
        "mkautodoc==0.2.0" \
        "mkdocs-material-extensions==1.3.1" \
        "mkdocs-material==9.5.12" \
        "mkdocs==1.5.3" \
        "more-itertools==10.2.0" \
        "mypy-extensions==1.0.0" \
        "mypy==1.8.0" \
        "nh3==0.2.15" \
        "outcome==1.3.0.post0" \
        "packaging==23.2" \
        "paginate==0.5.6" \
        "pathspec==0.12.1" \
        "pkginfo==1.10.0" \
        "platformdirs==4.2.0" \
        "pluggy==1.4.0" \
        "pycparser==2.21" \
        "pygments==2.17.2" \
        "pymdown-extensions==10.7.1" \
        "pyproject-hooks==1.0.0" \
        "pytest==8.0.2" \
        "python-dateutil==2.9.0.post0" \
        "python-multipart==0.0.9" \
        "pyyaml-env-tag==0.1" \
        "pyyaml==6.0.1" \
        "readme-renderer==43.0" \
        "regex==2023.12.25" \
        "requests-toolbelt==1.0.0" \
        "requests==2.31.0" \
        "rfc3986==2.0.0" \
        "rich==13.7.1" \
        "ruff==0.1.15" \
        "secretstorage==3.3.3" \
        "six==1.16.0" \
        "sniffio==1.3.1" \
        "sortedcontainers==2.4.0" \
        "trio==0.24.0" \
        "twine==5.0.0" \
        "types-contextvars==2.4.7.3" \
        "types-dataclasses==0.6.6" \
        "types-pyyaml==6.0.12.12" \
        "typing-extensions==4.10.0" \
        "urllib3==2.2.1" \
        "watchdog==4.0.0" \
        "zipp==3.17.0" \
    && pip install --no-cache-dir --no-deps -e . \
    && test -z "$(git status --porcelain)"

# v1.1 node-id scoring: pytest ships a native JUnit XML reporter (--junitxml),
# so no extra reporter dependency is required.

# Disable git commit hooks (pre-commit, husky etc.): dev-workflow tooling, not task content.
# Broken hook environments otherwise block the agent's (and oracle's) commits.
RUN cd /app && git config core.hooksPath /dev/null

CMD ["/bin/bash"]
