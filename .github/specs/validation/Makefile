# SDD TOON Validation Makefile
# Purpose: Validate TOON specifications and ensure SDD compliance

.PHONY: all validate validate-specs validate-features validate-acceptance validate-architecture \
        check-structure lint-toon report clean help

# Default target
all: validate

# ============================================================================
# CONFIGURATION
# ============================================================================

# This Makefile runs from .github/specs/validation/
# Parent directory (..) is .github/specs/
SPECS_DIR := ..
FEATURES_DIR := $(SPECS_DIR)/features
ACCEPTANCE_DIR := $(SPECS_DIR)/acceptance
ARCHITECTURE_DIR := $(SPECS_DIR)/architecture
REPORT_DIR := $(SPECS_DIR)/.sdd-reports

# TOON file patterns
FEATURE_FILES := $(wildcard $(FEATURES_DIR)/*.feature.toon)
ACCEPTANCE_FILES := $(wildcard $(ACCEPTANCE_DIR)/*.acceptance.toon)
ARCHITECTURE_FILES := $(wildcard $(ARCHITECTURE_DIR)/*.architecture.toon)

# Colors for output
GREEN := \033[0;32m
YELLOW := \033[0;33m
RED := \033[0;31m
NC := \033[0m # No Color

# ============================================================================
# MAIN VALIDATION TARGETS
# ============================================================================

## validate: Run all validation checks
validate: check-structure validate-specs lint-toon
	@echo "$(GREEN)✓ All validations passed$(NC)"

## validate-specs: Validate all TOON specification files
validate-specs: validate-features validate-acceptance validate-architecture
	@echo "$(GREEN)✓ All specs validated$(NC)"

## validate-features: Validate feature TOON files
validate-features:
	@echo "Validating feature specifications..."
	@for file in $(FEATURE_FILES); do \
		echo "  Checking $$file..."; \
		$(MAKE) _validate-feature-file FILE=$$file || exit 1; \
	done
	@echo "$(GREEN)✓ Feature specs valid$(NC)"

## validate-acceptance: Validate acceptance TOON files
validate-acceptance:
	@echo "Validating acceptance specifications..."
	@for file in $(ACCEPTANCE_FILES); do \
		echo "  Checking $$file..."; \
		$(MAKE) _validate-acceptance-file FILE=$$file || exit 1; \
	done
	@echo "$(GREEN)✓ Acceptance specs valid$(NC)"

## validate-architecture: Validate architecture TOON files
validate-architecture:
	@echo "Validating architecture specifications..."
	@for file in $(ARCHITECTURE_FILES); do \
		echo "  Checking $$file..."; \
		$(MAKE) _validate-architecture-file FILE=$$file || exit 1; \
	done
	@echo "$(GREEN)✓ Architecture specs valid$(NC)"

# ============================================================================
# STRUCTURE CHECKS
# ============================================================================

## check-structure: Verify required directory structure exists
check-structure:
	@echo "Checking SDD directory structure..."
	@test -d $(SPECS_DIR) || (echo "$(RED)✗ Missing $(SPECS_DIR) directory$(NC)" && exit 1)
	@test -d $(FEATURES_DIR) || mkdir -p $(FEATURES_DIR)
	@test -d $(ACCEPTANCE_DIR) || mkdir -p $(ACCEPTANCE_DIR)
	@test -d $(ARCHITECTURE_DIR) || mkdir -p $(ARCHITECTURE_DIR)
	@echo "$(GREEN)✓ Directory structure OK$(NC)"

# ============================================================================
# TOON LINTING
# ============================================================================

## lint-toon: Lint all TOON files for syntax issues
lint-toon:
	@echo "Linting TOON files..."
	@$(MAKE) _lint-toon-syntax
	@echo "$(GREEN)✓ TOON syntax OK$(NC)"

_lint-toon-syntax:
	@for file in $(FEATURE_FILES) $(ACCEPTANCE_FILES) $(ARCHITECTURE_FILES); do \
		if [ -f "$$file" ]; then \
			grep -n "^[^#│├└─\t ]" "$$file" | grep -v "^$$" > /dev/null && \
			echo "$(YELLOW)⚠ $$file may have formatting issues$(NC)"; \
		fi \
	done || true

# ============================================================================
# INTERNAL VALIDATION HELPERS
# ============================================================================

# TOON v3.0 uses YAML-like format with lowercase keys
# Supports both singular (command:) and plural array (commands[N]) forms
# Example: feature:, meta:, command: OR commands[N], result:, scenarios:

_validate-feature-file:
	@test -f $(FILE) || (echo "$(RED)✗ File not found: $(FILE)$(NC)" && exit 1)
	@grep -q "^feature:" $(FILE) || (echo "$(RED)✗ Missing feature: header in $(FILE)$(NC)" && exit 1)
	@# command: (singular object) OR commands[ (plural array) are both valid
	@(grep -q "^command:" $(FILE) || grep -q "^commands\[" $(FILE)) || (echo "$(RED)✗ Missing command: or commands[N] section in $(FILE)$(NC)" && exit 1)
	@# result: (singular object) OR result (without colon for inline) are valid
	@grep -q "^result" $(FILE) || (echo "$(RED)✗ Missing result section in $(FILE)$(NC)" && exit 1)
	@grep -q "^scenarios" $(FILE) || (echo "$(RED)✗ Missing scenarios section in $(FILE)$(NC)" && exit 1)
	@# batch-decomposition is MANDATORY for SDD workflow
	@grep -q "^batch-decomposition:" $(FILE) || (echo "$(RED)✗ Missing batch-decomposition: section in $(FILE)$(NC)" && exit 1)

_validate-acceptance-file:
	@test -f $(FILE) || (echo "$(RED)✗ File not found: $(FILE)$(NC)" && exit 1)
	@grep -q "^acceptance:" $(FILE) || (echo "$(RED)✗ Missing acceptance: header in $(FILE)$(NC)" && exit 1)
	@grep -q "^meta:" $(FILE) || (echo "$(RED)✗ Missing meta: section in $(FILE)$(NC)" && exit 1)
	@grep -q "^acceptance-criteria" $(FILE) || (echo "$(RED)✗ Missing acceptance-criteria in $(FILE)$(NC)" && exit 1)
	@grep -q "^test-cases:" $(FILE) || (echo "$(RED)✗ Missing test-cases: in $(FILE)$(NC)" && exit 1)

_validate-architecture-file:
	@test -f $(FILE) || (echo "$(RED)✗ File not found: $(FILE)$(NC)" && exit 1)
	@grep -q "^architecture:" $(FILE) || (echo "$(RED)✗ Missing architecture: header in $(FILE)$(NC)" && exit 1)
	@grep -q "^meta:" $(FILE) || (echo "$(RED)✗ Missing meta: section in $(FILE)$(NC)" && exit 1)
	@grep -q "^layer-structure" $(FILE) || (echo "$(RED)✗ Missing layer-structure in $(FILE)$(NC)" && exit 1)
	@grep -q "^component-contracts:" $(FILE) || (echo "$(RED)✗ Missing component-contracts: in $(FILE)$(NC)" && exit 1)

# ============================================================================
# REPORTING
# ============================================================================

## report: Generate validation report
report: $(REPORT_DIR)
	@echo "Generating SDD validation report..."
	@echo "# SDD Validation Report" > $(REPORT_DIR)/validation-report.md
	@echo "Generated: $$(date)" >> $(REPORT_DIR)/validation-report.md
	@echo "" >> $(REPORT_DIR)/validation-report.md
	@echo "## Feature Specs" >> $(REPORT_DIR)/validation-report.md
	@echo "Found: $$(echo $(FEATURE_FILES) | wc -w | tr -d ' ') files" >> $(REPORT_DIR)/validation-report.md
	@echo "" >> $(REPORT_DIR)/validation-report.md
	@echo "## Acceptance Specs" >> $(REPORT_DIR)/validation-report.md
	@echo "Found: $$(echo $(ACCEPTANCE_FILES) | wc -w | tr -d ' ') files" >> $(REPORT_DIR)/validation-report.md
	@echo "" >> $(REPORT_DIR)/validation-report.md
	@echo "## Architecture Specs" >> $(REPORT_DIR)/validation-report.md
	@echo "Found: $$(echo $(ARCHITECTURE_FILES) | wc -w | tr -d ' ') files" >> $(REPORT_DIR)/validation-report.md
	@echo "$(GREEN)✓ Report generated: $(REPORT_DIR)/validation-report.md$(NC)"

$(REPORT_DIR):
	@mkdir -p $(REPORT_DIR)

## coverage: Check spec coverage
coverage:
	@echo "Checking specification coverage..."
	@echo "Feature specs: $$(echo $(FEATURE_FILES) | wc -w | tr -d ' ')"
	@echo "Acceptance specs: $$(echo $(ACCEPTANCE_FILES) | wc -w | tr -d ' ')"
	@echo "Architecture specs: $$(echo $(ARCHITECTURE_FILES) | wc -w | tr -d ' ')"
	@if [ "$$(echo $(FEATURE_FILES) | wc -w)" -gt "$$(echo $(ACCEPTANCE_FILES) | wc -w)" ]; then \
		echo "$(YELLOW)⚠ Some features may be missing acceptance specs$(NC)"; \
	fi
	@if [ "$$(echo $(FEATURE_FILES) | wc -w)" -gt "$$(echo $(ARCHITECTURE_FILES) | wc -w)" ]; then \
		echo "$(YELLOW)⚠ Some features may be missing architecture specs$(NC)"; \
	fi

# ============================================================================
# CLEANUP
# ============================================================================

## clean: Clean generated reports
clean:
	@rm -rf $(REPORT_DIR)
	@echo "$(GREEN)✓ Cleaned$(NC)"

# ============================================================================
# HELP
# ============================================================================

## help: Show this help message
help:
	@echo "SDD TOON Validation Makefile"
	@echo ""
	@echo "Usage: make [target]"
	@echo ""
	@echo "Targets:"
	@grep -E '^## ' $(MAKEFILE_LIST) | sed 's/## /  /' | sort
	@echo ""
	@echo "Examples:"
	@echo "  make validate           # Run all validations"
	@echo "  make validate-features  # Validate only feature specs"
	@echo "  make report             # Generate validation report"
	@echo "  make coverage           # Check spec coverage"
