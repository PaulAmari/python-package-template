#* Variables
SHELL := /usr/bin/env bash
PYTHON := python
PYTHONPATH := `pwd`

#* Docker variables
IMAGE := {{ cookiecutter.project_name.lower().replace(' ', '_').replace('-', '_') }}
VERSION := latest

#* Poetry
.PHONY: poetry-download
poetry-download:
	curl -sSL https://install.python-poetry.org | $(PYTHON) -

.PHONY: poetry-remove
poetry-remove:
	curl -sSL https://install.python-poetry.org | $(PYTHON) - --uninstall

#* Installation
.PHONY: install
install:
	poetry lock -n && poetry export --without-hashes > requirements.txt
	poetry install -n
	-poetry run mypy --install-types --non-interactive ./

.PHONY: pre-commit-install
pre-commit-install:
	poetry run pre-commit install

.PHONY: trivy-install
trivy-install:
	sudo apt-get install trivy

#* Formatters
.PHONY: format-codestyle
format-codestyle:
	poetry run docformatter -r -in-place --black --pre-summary-newline ./
	poetry run pyupgrade --exit-zero-even-if-changed --py39-plus **/*.py
	poetry run ruff format --config pyproject.toml ./

.PHONY: formatting
formatting: format-codestyle

#* Testing

.PHONY: tests
tests:
	poetry run tox -e py39,py310,py311,coverage -p

.PHONY: coverage
coverage:
	poetry run tox -e coverage

.PHONY: lint
lint:
	poetry run tox -e lint


.PHONY: format
format:
	poetry run tox -e format

.PHONY: safety
safety:
	poetry run tox -e safety

.PHONY: type
type:
	poetry run tox -e type

.PHONY: checks
checks:
	poetry run tox -e codestyle,lint,safety,type -p

.PHONY: update-dev-deps
update-dev-deps:
	poetry add -G dev bandit@latest mypy@latest pre-commit@latest pylint@latest pytest@latest pyupgrade@latest safety@latest coverage@latest coverage-badge@latest pytest-html@latest pytest-cov@latest
	poetry add -G dev --allow-prereleases ruff@latest

#* SBOM
.PHONY: generate-sbom
generate-sbom:
	mkdir -p ./analysis/sbom
	poetry run cyclonedx-py environment > $(ANALYSIS_PATH)/sbom/cyclonedx_sbom.json
	syft {{ cookiecutter.project_name.lower().replace(' ', '_').replace('-', '_') }}:latest -o=cyclonedx-json --file=./analysis/sbom/syft_sbom.json

#* Vulnerabilities Analysis
.PHONY: check-vulnerabilities
check-vulnerabilities:
	mkdir -p $(ANALYSIS_PATH)/cve
	trivy sbom $(ANALYSIS_PATH)/sbom/syft_sbom.json -f json -o ./analysis/cve/trivy_cve.json
	trivy sbom $(ANALYSIS_PATH)/sbom/syft_sbom.json -f table

#* Licenses Analysis
.PHONY: licenses
licenses:
	mkdir -p $(ANALYSIS_PATH)/license
	poetry run pip-licenses --summary --from=classifier --with-system -f csv --output-file $(ANALYSIS_PATH)/license/license_summary.csv
	poetry run pip-licenses -o license -f csv --output-file $(ANALYSIS_PATH)/license/licenses.csv

.PHONY:  check-licenses
check-licenses:
	poetry run pip-licenses --fail-on="UNKNOWN"

#* Docker
# Example: make docker-build VERSION=latest
# Example: make docker-build IMAGE=some_name VERSION=0.1.0
.PHONY: docker-build
docker-build:
	@echo Building docker $(IMAGE):$(VERSION) ...
	docker build \
		-t $(IMAGE):$(VERSION) . \
		-f ./docker/Dockerfile --no-cache

.PHONY: docker-build-alpine
docker-build-alpine:
	@echo Building docker $(IMAGE):$(VERSION) ...
	docker build \
		-t $(IMAGE):$(VERSION) . \
		-f ./docker/alpine/Dockerfile --no-cache

# Example: make docker-remove VERSION=latest
# Example: make docker-remove IMAGE=some_name VERSION=0.1.0
.PHONY: docker-remove
docker-remove:
	@echo Removing docker $(IMAGE):$(VERSION) ...
	docker rmi -f $(IMAGE):$(VERSION)

#* Cleaning
.PHONY: pycache-remove
pycache-remove:
	find . | grep -E "(__pycache__|\.pyc|\.pyo$$)" | xargs rm -rf

.PHONY: dsstore-remove
dsstore-remove:
	find . | grep -E ".DS_Store" | xargs rm -rf

.PHONY: mypycache-remove
mypycache-remove:
	find . | grep -E ".mypy_cache" | xargs rm -rf

.PHONY: ipynbcheckpoints-remove
ipynbcheckpoints-remove:
	find . | grep -E ".ipynb_checkpoints" | xargs rm -rf

.PHONY: pytestcache-remove
pytestcache-remove:
	find . | grep -E ".pytest_cache" | xargs rm -rf

.PHONY: build-remove
build-remove:
	rm -rf build/

.PHONY: cleanup
cleanup: pycache-remove dsstore-remove mypycache-remove ipynbcheckpoints-remove pytestcache-remove
