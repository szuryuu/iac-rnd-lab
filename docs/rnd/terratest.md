# Terratest Guide

This repository uses **Terratest** (Go library) to validate Terraform infrastructure code automatically.

## Directory Structure
Tests are located within the `test/` directory of each environment or module.

``text
terraform/
├── environments/
│   └── staging/
│       ├── main.tf
│       └── test/
│           └── main_test.go  <-- Test definition
``

## Prerequisites
- **Go:** Version 1.18+
- **Terraform:** Installed and in PATH.

## Running Tests

### 1. Initialize Module
Navigate to the test directory:
``bash
cd terraform/environments/staging/test
``

### 2. Initialize Go Module (First Run)
If `go.mod` is missing:
``bash
go mod init staging-test
go get github.com/gruntwork-io/terratest/modules/terraform
go get github.com/stretchr/testify/assert
``

### 3. Execute Tests
Run the Go test. The `-v` flag provides verbose output (Terraform logs).
``bash
go test -v -timeout 30m
``

> **Warning:** This will create real resources in your cloud provider and incur costs. The test includes a `defer TerraformDestroy` function to clean up resources automatically after the test finishes, even if it fails.

## Example Test Code (`main_test.go`)

``go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformStaging(t *testing.T) {
	t.Parallel()

	terraformOptions := &terraform.Options{
		// The path to where our Terraform code is located
		TerraformDir: "../",
		
		// Variables to pass to our Terraform code using -var options
		Vars: map[string]interface{}{
			"environment": "test_sandbox",
		},
	}

	// Clean up resources with "terraform destroy" at the end of the test
	defer terraform.Destroy(t, terraformOptions)

	// Run "terraform init" and "terraform apply"
	terraform.InitAndApply(t, terraformOptions)

	// Run "terraform output" to get the value of an output variable
	vmIP := terraform.Output(t, terraformOptions, "vm_public_ip")

	// Verify we got a valid IP address
	assert.NotEmpty(t, vmIP)
}
``
