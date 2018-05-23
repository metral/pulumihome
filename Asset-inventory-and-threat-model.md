## Assets by location

### NPM
- Pulumi packages

### Travis
- Pulumi source code
- Pulumi AWS credentials

### GitHub
- Pulumi source code
- Pulumi organization membership
  - Used by service for authorization, may grant access to Pulumi infrastructure

### AWS
- General
  - Access to valuable resources (e.g. compute for cryptocurrency mining)

- KMS
  - Service key
    - Creates update tokens
    - Protects key material in DB that encrypts stack secrets

- S3
  - Checkpoints
    - Resource names, types, connections, configuration, endpoint names, IP addresses
    - Source code for serialized Lambdas (`__index.js`)
    - May include deployment secrets in plaintext, e.g. in Lambda text or container environment variables
  - Installer and downloads, e.g. Pulumi CLI

- Service database (RDS Aurora)
  - User data
    - GitHub login
    - GitHub access token with user:email and read:org scopes
    - Name
    - E-mail address
  - Stack data
    - Stack names
    - Stack key material (protected by KMS)
    - Tags
    - Update logs
  - Authorization data
    - User access levels
    - User-stack permissions

### Google Drive
- Passwords
- SSH keys

### LastPass
- Passwords

### Pulumi developer machine
- Credentials
  - Google Apps
  - LastPass
  - AWS
  - NPM
  - GitHub
  - Pulumi service, potentially as admin
- Other personal data

### User machine
- User AWS credentials
- Deployment secrets
- Code execution: Pulumi CLI, Node packages
- Other personal data
