

# **HCP Boundary Documentation**

---

## **Initial Setup**

### **Configure Environment Variables**

\# Set Boundary address  
export BOUNDARY\_ADDR=https://4.193.226.1:9200

\# Skip TLS verification (for self-signed certificates)  
export BOUNDARY\_TLS\_INSECURE=true

\# Disable keyring (optional, for simpler token management)  
export BOUNDARY\_KEYRING\_TYPE=none

### **Initial Admin Login**

\# First login as admin (password auth)  
boundary authenticate password \\  
  \-login-name admin \\  
  \-addr $BOUNDARY\_ADDR

\# Save token  
export BOUNDARY\_TOKEN=\<token-from-output\>

---

## **OIDC Authentication**

### **Prerequisites**

Get these values from Azure Portal:

1. **Application (client) ID**  
2. **Client Secret**  
3. **Tenant ID**

**Reference:** [OIDC authentication with Microsoft Azure](https://developer.hashicorp.com/boundary/docs/configuration/identity-providers/azuread)

### **1\. Set Environment Variables**

export CLIENT\_ID=\<your-client-id\>  
export CLIENT\_SECRET=\<your-client-secret\>  
export TENANT\_ID=\<your-tenant-id\>

### **2\. Create OIDC Auth Method**

boundary auth-methods create oidc \\  
  \-issuer "https://login.microsoftonline.com/$TENANT\_ID/v2.0" \\  
  \-client-id "$CLIENT\_ID" \\  
  \-client-secret "$CLIENT\_SECRET" \\  
  \-signing-algorithm RS256 \\  
  \-api-url-prefix "$BOUNDARY\_ADDR" \\  
  \-name "azure-oidc" \\  
  \-description "Azure Entra ID OIDC Authentication"

\# Save the Auth Method ID from output  
\# Example: amoidc\_rFqhKQlrvn

### **3\. Activate Auth Method**

boundary auth-methods change-state oidc \\  
  \-id \<auth-method-id\> \\  
  \-state active-public

### **4\. Set as Primary Auth Method**

boundary scopes update \\  
  \-primary-auth-method-id \<auth-method-id\> \\  
  \-id global

### **5\. Test OIDC Login**

boundary authenticate oidc

\# Browser will open for Azure login  
\# After successful login, save credentials (token, user id)

---

## **SSH Credential Management**

### **1\. Create Credential Store**

boundary credential-stores create static \\  
  \-scope-id \<project-id\> \\  
  \-name "ssh-credentials-store" \\  
  \-description "Static SSH credentials for VMs"

\# Save the Store ID from output  
\# Example: csst\_jFN5t0sp5F

**Get Project ID:**

\# List projects to find project-id  
boundary scopes list \-scope-id global

### **2\. Upload SSH Private Key**

boundary credentials create ssh-private-key \\  
  \-credential-store-id \<store-id\> \\  
  \-username adminuser \\  
  \-private-key file://$HOME/.ssh/azure\_vm\_key \\  
  \-name "vm-ssh-key" \\  
  \-description "SSH key for dev VMs"

\# Save the Credential ID from output  
\# Example: credspk\_ZkiRfL2u3r

**Note:** If your key has a passphrase, you'll be prompted to enter it.

### **3\. Attach Credential to Target**

boundary targets add-credential-sources \\  
  \-id \<target-id\> \\  
  \-brokered-credential-source \<credential-id\>

\# Get target-id from  
boundary targets list \-keyring-type=none \-scope-id \<project-id\>

---

## **User Access Flow**

### **For System Administrator**

#### **1\. Create Role for SSH Access**

\# Create role  
boundary roles create \\  
  \-scope-id \<project-id\> \\  
  \-name "ssh-users" \\  
  \-description "Users who can SSH to VMs"

\# Add grants  
boundary roles add-grants \\  
  \-id \<role-id\> \\  
  \-grant "ids=\<target-id\>;actions=authorize-session,read"

#### **2\. Add User to Role**

**Option A: Add individual user**

\# User must login once first to get their user-id  
\# After user's first login, get their user-id:  
boundary users list \-scope-id global

\# Add user to role  
boundary roles add-principals \\  
  \-id \<role-id\> \\  
  \-principal \<user-id\>

**Option B: Use Managed Groups (Never Try)**

\# Create managed group for auto-assignment  
boundary managed-groups create oidc \\  
  \-auth-method-id \<oidc-auth-method-id\> \\  
  \-name "azure-ssh-users" \\  
  \-filter '"@yourcompany.com" in "/token/email"'

\# Add managed group to role  
boundary roles add-principals \\  
  \-id \<role-id\> \\  
  \-principal \<managed-group-id\>

### **For End Users**

#### **First Time Setup**

\# 1\. Set environment variables  
export BOUNDARY\_ADDR=https://4.193.226.1:9200  
export BOUNDARY\_TLS\_INSECURE=true  
export BOUNDARY\_KEYRING\_TYPE=none

\# 2\. Login via OIDC  
boundary authenticate oidc

\# 3\. Save token (displayed after login)  
export BOUNDARY\_TOKEN=\<token-from-output\>

\# 4\. Contact SysAdmin with your User ID (shown in login output)  
\# Example: User ID: u\_5jsTii2bFb

#### **Regular Usage**

\# 1\. Login (if token expired)  
boundary authenticate oidc  
export BOUNDARY\_TOKEN=\<token\>

\# 2\. Connect to VM  
boundary connect ssh \\  
  \-target-id \<target-id\> \\  
  \-token env://BOUNDARY\_TOKEN

\# Or shorter (if token already in env)  
boundary connect ssh \-target-id \<target-id\>

---

## **Cleanup**

### **Delete OIDC Auth Method**

boundary auth-methods delete \-id \<auth-method-id\>

### **Delete Credentials**

\# Delete specific credential  
boundary credentials delete \-id \<credential-id\>

\# Delete credential store  
boundary credential-stores delete \-id \<store-id\>  
---

## **Common Commands**

\# Login  
boundary authenticate oidc

\# List targets  
boundary targets list \-scope-id \<project-id\>

\# List roles  
boundary roles list \-scope-id \<project-id\>

\# List users  
boundary users list \-scope-id global

\# Connect to VM  
boundary connect ssh \-target-id \<target-id\>

\# Check token expiration  
boundary authenticate oidc \-format json | jq '.item.expiration\_time'

---

## 

## **References**

1. [OIDC authentication with Microsoft Azure | Boundary | HashiCorp Developer](https://developer.hashicorp.com/boundary/tutorials/identity-management/oidc-azure)   
2. [Broker static credentials to your first target | Boundary | HashiCorp Developer](https://developer.hashicorp.com/boundary/tutorials/get-started-hcp/hcp-getting-started-credentials)   
3. [Tutorials | Boundary | HashiCorp Developer](https://developer.hashicorp.com/boundary/tutorials) 


