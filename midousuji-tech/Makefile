SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
.DEFAULT_GOAL := help

.PHONY: default show-cwd delete-ds-store

default:## Display available tasks
	@echo "Available tasks:"
	@echo "  make show-cwd         - Show current working directory"
	@echo "  make delete-ds-store  - Delete all .DS_Store files"


show-cwd: ## Show current working directory
	@echo "current directory: $(shell pwd)"


delete-ds-store: ## Delete all .DS_Store files
	find . -name '.DS_Store' -type f -ls -delete

help: ## Show this help message
	@echo "Usage: make [target]"
	@echo ""
	@echo "Available targets:"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(filter-out .env,$(MAKEFILE_LIST)) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
