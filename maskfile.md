# poly-helm-mode Tasks

## install-cask

> Install Cask if it's not already installed.

```sh
set -e

if ! command -v cask > /dev/null; then
  echo "Cask is not installed. Installing..."
  git clone https://github.com/cask/cask.git /tmp/cask
  make -C /tmp/cask install
  rm -rf /tmp/cask
else
  echo "Cask is already installed."
fi
```

## test

> Run the test suite. This requires Cask to be installed.

```sh
set -e

# Use EMACS environment variable if set, otherwise default to "emacs"
EMACS=${EMACS:-emacs}

if ! command -v cask > /dev/null; then
  echo "Cask is not installed. Please run 'mask install-cask' first."
  exit 1
fi

cask install
cask exec "$EMACS" --batch -L . -L test -l poly-helm-mode-tests.el -f ert-run-tests-batch-and-exit
```

## lint-package

> Run the package linter -- see https://github.com/purcell/package-lint

```sh
set -e

# Use EMACS environment variable if set, otherwise default to "emacs"
EMACS=${EMACS:-emacs}
DEPS="cl-lib let-alist compat package-lint"

INIT_PACKAGES="(progn \
  (require 'package) \
  (push '(\"melpa\" . \"https://melpa.org/packages/\") package-archives) \
  (package-initialize) \
  (dolist (pkg '(${DEPS})) \
    (unless (package-installed-p pkg) \
      (unless (assoc pkg package-archive-contents) \
	(package-refresh-contents)) \
      (package-install pkg))) \
  (unless package-archive-contents (package-refresh-contents)) \
  )"

${EMACS} -Q --eval "${INIT_PACKAGES}" -batch -f package-lint-batch-and-exit poly-helm-mode.el
```
