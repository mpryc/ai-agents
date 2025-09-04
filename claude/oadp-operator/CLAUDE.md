# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About This Project

OADP (OpenShift API for Data Protection) is an operator that installs and manages Velero on OpenShift clusters for backup and restore operations. It's a Go-based Kubernetes operator using the operator-sdk framework.

## Common Development Commands

### Building and Testing
- `make build` - Build the manager binary
- `make test` - Run unit tests with linting and dependency checks
- `make lint` - Run Go linters using golangci-lint
- `make lint-fix` - Fix linting issues automatically
- `make vet` - Run go vet
- `make fmt` - Run go fmt

### Code Generation
- `make generate` - Generate DeepCopy methods
- `make manifests` - Generate CRDs and RBAC manifests
- `make bundle` - Generate operator bundle manifests

### Local Development and Deployment
- `make deploy-olm` - Build and deploy operator via OLM to openshift-adp namespace
- `make undeploy-olm` - Uninstall operator deployed via OLM
- `make docker-build` - Build container image
- `make docker-push` - Push container image

### Testing
- `make test-e2e` - Run end-to-end tests (requires cloud credentials setup)
- `make test-e2e-setup` - Set up E2E test environment
- `make test-e2e-cleanup` - Clean up after E2E tests
- `make install-ginkgo` - Install Ginkgo test framework

### Single Test Commands
- `ginkgo run -mod=mod tests/e2e/ -- --help` - Get E2E test help
- Run specific tests with: `make test-e2e GINKGO_ARGS="--ginkgo.focus='Test Name'"`
- `go test -mod=mod ./internal/controller/...` - Run unit tests for controllers
- `go test -mod=mod ./pkg/...` - Run unit tests for packages

## Project Structure

### Key Directories
- `api/v1alpha1/` - API types and CRD definitions
- `internal/controller/` - Controller implementations
- `pkg/` - Shared packages (credentials, storage, utils)
- `config/` - Kubernetes manifests and kustomization files
- `tests/e2e/` - End-to-end test suites
- `docs/` - Documentation and design documents
- `must-gather/` - Must-gather debugging tool

### Key Packages
- `pkg/credentials/` - Cloud credential management and STS token handling
- `pkg/storage/` - Cloud storage abstraction layer (AWS S3, Azure Blob, GCP)
- `pkg/common/` - Shared utilities and constants
- `pkg/bucket/` - Bucket validation and management utilities

### Core Controllers
- `dataprotectionapplication_controller.go` - Main DPA controller that orchestrates OADP installation
- `velero.go` - Velero deployment management and configuration
- `bsl.go` / `vsl.go` - Backup/Volume Snapshot Location controllers
- `nonadmin_controller.go` - Non-admin user backup functionality
- `cloudstorage_controller.go` - Cloud storage resource management
- `dataprotectiontest_controller.go` - Test resource validation

### Custom Resources
- **DataProtectionApplication (DPA)** - Main CR for configuring OADP
- **CloudStorage** - Cloud storage configuration
- **DataProtectionTest** - Test resource for validation
- **NonAdmin*** - Resources for non-admin user workflows

## Architecture Notes

This operator follows the standard Kubernetes operator pattern using controller-runtime. It manages Velero installations and extends them with OpenShift-specific functionality.

### Key Integrations
- **Velero** - Core backup/restore engine (forked OpenShift version)
- **Cloud Providers** - AWS, Azure, GCP support via plugins
- **Storage** - Integrates with various storage backends and CSI
- **Virtualization** - KubeVirt VM backup support via plugins

### Important Patterns
- Uses feature flags for optional functionality (configured via DPA spec.configuration.velero.defaultPlugins)
- Supports multiple cloud provider backends simultaneously
- Implements STS (Security Token Service) flows for cloud authentication
- Provides both admin and non-admin user workflows
- Controller reconciliation follows standard Kubernetes patterns with status updates
- Plugin architecture allows extensibility for different cloud providers and features
- Velero configuration is managed through generated ConfigMaps and Secrets

## Environment Variables for Testing

### Required for E2E Tests
- `OADP_CRED_FILE` - Path to backup storage credentials
- `OADP_BUCKET` - Bucket name for backups
- `CI_CRED_FILE` - Path to snapshot storage credentials
- `VSL_REGION` - Volume snapshot location region
- `BSL_REGION` - Backup storage location region (default: us-east-1)

### Test Configuration
- `OADP_TEST_NAMESPACE` - Test namespace (default: openshift-adp)
- `TEST_VIRT=true` - Run virtualization tests
- `TEST_UPGRADE=true` - Run upgrade tests
- `TEST_CLI=true` - Run CLI-based tests
- `SKIP_MUST_GATHER=true` - Skip must-gather compilation

## Development Notes

- Go version: 1.23.0+ (see go.mod)
- Uses operator-sdk v1.35.0
- Supports multiple container tools (docker/podman)
- Includes extensive E2E test coverage for cloud providers
- Must be logged into OpenShift cluster for most development tasks
- Run `make lint` and `make test` before submitting PRs
- Controller unit tests use envtest framework for Kubernetes API simulation

## Cloud Provider Setup

Each cloud provider requires specific credential file formats and region configurations. See `docs/developer/testing/TESTING.md` for detailed setup instructions.