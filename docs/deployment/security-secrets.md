# Security and Secrets Management

This guide covers comprehensive security practices for DealSphere, including data encryption, secrets management, access controls, and security monitoring for both data at rest and in transit.

## Overview

DealSphere implements defense-in-depth security principles with multiple layers of protection:

- **Data Encryption**: End-to-end encryption for data at rest and in transit
- **Secrets Management**: Centralized and secure handling of sensitive configuration
- **Access Controls**: Role-based authentication and authorization
- **Network Security**: TLS/SSL, VPNs, and network segmentation
- **Security Monitoring**: Real-time threat detection and audit logging
- **Compliance**: GDPR, SOC 2, and industry best practices

## Data Security

### Data at Rest Encryption

#### Database Encryption

**PostgreSQL Transparent Data Encryption (TDE)**

```sql
-- Enable encryption at database level
CREATE DATABASE dealsphere_prod WITH ENCRYPTION;

-- Column-level encryption for sensitive data
CREATE TABLE user_credentials (
    id UUID PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    -- Encrypted sensitive fields
    ssn_encrypted BYTEA,  -- Encrypted SSN
    payment_info_encrypted BYTEA,  -- Encrypted payment info
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Application-level encryption functions
CREATE OR REPLACE FUNCTION encrypt_sensitive_data(plain_text TEXT, encryption_key TEXT)
RETURNS BYTEA AS $$
BEGIN
    RETURN pgp_sym_encrypt(plain_text, encryption_key);
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION decrypt_sensitive_data(encrypted_data BYTEA, encryption_key TEXT)
RETURNS TEXT AS $$
BEGIN
    RETURN pgp_sym_decrypt(encrypted_data, encryption_key);
END;
$$ LANGUAGE plpgsql;
```

**AWS RDS Encryption**

```yaml
# terraform/rds.tf
resource "aws_db_instance" "dealsphere_prod" {
  identifier = "dealsphere-prod-db"

  # Encryption configuration
  storage_encrypted = true
  kms_key_id       = aws_kms_key.database_key.arn

  # Backup encryption
  backup_retention_period = 30
  backup_window          = "03:00-04:00"
  copy_tags_to_snapshot  = true

  # Performance Insights encryption
  performance_insights_enabled = true
  performance_insights_kms_key_id = aws_kms_key.database_key.arn

  # Additional security settings
  deletion_protection = true
  skip_final_snapshot = false
  final_snapshot_identifier = "dealsphere-prod-final-snapshot"
}

resource "aws_kms_key" "database_key" {
  description = "KMS key for DealSphere database encryption"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          AWS = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:root"
        }
        Action = "kms:*"
        Resource = "*"
      }
    ]
  })

  tags = {
    Name = "dealsphere-database-key"
    Environment = "production"
  }
}
```

#### File Storage Encryption

**AWS S3 Server-Side Encryption**

```yaml
# terraform/s3.tf
resource "aws_s3_bucket" "dealsphere_storage" {
  bucket = "dealsphere-files-prod"
}

resource "aws_s3_bucket_server_side_encryption_configuration" "dealsphere_storage" {
  bucket = aws_s3_bucket.dealsphere_storage.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.s3_key.arn
    }
    bucket_key_enabled = true
  }
}

resource "aws_s3_bucket_public_access_block" "dealsphere_storage" {
  bucket = aws_s3_bucket.dealsphere_storage.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_kms_key" "s3_key" {
  description = "KMS key for DealSphere S3 encryption"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          AWS = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:root"
        }
        Action = "kms:*"
        Resource = "*"
      }
    ]
  })
}
```

**Application-Level File Encryption**

```java
// File encryption service
@Service
public class FileEncryptionService {
    private final KmsClient kmsClient;
    private final String dataKeyId;

    public FileEncryptionService(KmsClient kmsClient,
                               @Value("${aws.kms.data-key-id}") String dataKeyId) {
        this.kmsClient = kmsClient;
        this.dataKeyId = dataKeyId;
    }

    public byte[] encryptFile(byte[] fileData) throws Exception {
        // Generate data key
        GenerateDataKeyRequest keyRequest = GenerateDataKeyRequest.builder()
            .keyId(dataKeyId)
            .keySpec(DataKeySpec.AES_256)
            .build();

        GenerateDataKeyResponse keyResponse = kmsClient.generateDataKey(keyRequest);
        byte[] plaintextKey = keyResponse.plaintext().asByteArray();
        byte[] encryptedKey = keyResponse.ciphertextBlob().asByteArray();

        // Encrypt file with data key
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        SecretKeySpec keySpec = new SecretKeySpec(plaintextKey, "AES");
        cipher.init(Cipher.ENCRYPT_MODE, keySpec);

        byte[] encryptedData = cipher.doFinal(fileData);
        byte[] iv = cipher.getIV();

        // Combine encrypted key, IV, and encrypted data
        ByteArrayOutputStream output = new ByteArrayOutputStream();
        output.write(encryptedKey.length);
        output.write(encryptedKey);
        output.write(iv.length);
        output.write(iv);
        output.write(encryptedData);

        return output.toByteArray();
    }

    public byte[] decryptFile(byte[] encryptedFileData) throws Exception {
        ByteArrayInputStream input = new ByteArrayInputStream(encryptedFileData);

        // Extract encrypted key
        int keyLength = input.read();
        byte[] encryptedKey = new byte[keyLength];
        input.read(encryptedKey);

        // Extract IV
        int ivLength = input.read();
        byte[] iv = new byte[ivLength];
        input.read(iv);

        // Extract encrypted data
        byte[] encryptedData = input.readAllBytes();

        // Decrypt data key
        DecryptRequest decryptRequest = DecryptRequest.builder()
            .ciphertextBlob(SdkBytes.fromByteArray(encryptedKey))
            .build();

        DecryptResponse decryptResponse = kmsClient.decrypt(decryptRequest);
        byte[] plaintextKey = decryptResponse.plaintext().asByteArray();

        // Decrypt file data
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        SecretKeySpec keySpec = new SecretKeySpec(plaintextKey, "AES");
        GCMParameterSpec gcmSpec = new GCMParameterSpec(128, iv);
        cipher.init(Cipher.DECRYPT_MODE, keySpec, gcmSpec);

        return cipher.doFinal(encryptedData);
    }
}
```

### Data in Transit Encryption

#### TLS/SSL Configuration

**Application TLS Configuration**

```yaml
# application-security.yml
server:
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: ${SSL_KEYSTORE_PASSWORD}
    key-store-type: PKCS12
    key-alias: dealsphere

    # TLS version and cipher configuration
    protocol: TLS
    enabled-protocols: TLSv1.2,TLSv1.3
    ciphers:
      - TLS_AES_256_GCM_SHA384
      - TLS_CHACHA20_POLY1305_SHA256
      - TLS_AES_128_GCM_SHA256
      - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
      - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

    # Client certificate authentication (optional)
    client-auth: want
    trust-store: classpath:truststore.p12
    trust-store-password: ${SSL_TRUSTSTORE_PASSWORD}

management:
  server:
    ssl:
      enabled: true
      key-store: classpath:keystore.p12
      key-store-password: ${SSL_KEYSTORE_PASSWORD}
```

**Nginx TLS Termination**

```nginx
# nginx/ssl.conf
server {
    listen 443 ssl http2;
    server_name dealsphere.com;

    # SSL Certificate configuration
    ssl_certificate /etc/ssl/certs/dealsphere.crt;
    ssl_certificate_key /etc/ssl/private/dealsphere.key;

    # SSL Security configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # HSTS (HTTP Strict Transport Security)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' https:; connect-src 'self' https:; frame-ancestors 'self';" always;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/ssl/certs/chain.crt;

    location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name dealsphere.com;
    return 301 https://$server_name$request_uri;
}
```

#### Database Connection Security

```yaml
# Database SSL configuration
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}?sslmode=require&sslcert=/etc/ssl/certs/client-cert.pem&sslkey=/etc/ssl/private/client-key.pem&sslrootcert=/etc/ssl/certs/ca-cert.pem
    hikari:
      connection-test-query: SELECT 1
      connection-timeout: 30000
      validation-timeout: 5000
      leak-detection-threshold: 60000
```

## Secrets Management

### AWS Secrets Manager Integration

#### Secrets Storage

```bash
# Store database credentials
aws secretsmanager create-secret \
    --name "dealsphere/prod/database" \
    --description "Production database credentials" \
    --secret-string '{
        "username": "dealsphere_user",
        "password": "super-secure-db-password-123!",
        "host": "dealsphere-prod-cluster.cluster-abc123.us-west-2.rds.amazonaws.com",
        "port": 5432,
        "database": "dealsphere_prod",
        "ssl_mode": "require"
    }'

# Store JWT secrets
aws secretsmanager create-secret \
    --name "dealsphere/prod/jwt" \
    --description "JWT signing and encryption keys" \
    --secret-string '{
        "signing_key": "jwt-signing-key-256-bit-random-string",
        "encryption_key": "jwt-encryption-key-256-bit-random",
        "refresh_key": "jwt-refresh-key-256-bit-random-string"
    }'

# Store third-party API keys
aws secretsmanager create-secret \
    --name "dealsphere/prod/external-apis" \
    --description "External API credentials" \
    --secret-string '{
        "stripe_secret_key": "sk_live_...",
        "sendgrid_api_key": "SG.abc123...",
        "aws_access_key": "AKIA...",
        "aws_secret_key": "abc123..."
    }'

# Store encryption keys
aws secretsmanager create-secret \
    --name "dealsphere/prod/encryption" \
    --description "Application encryption keys" \
    --secret-string '{
        "data_encryption_key": "32-byte-random-encryption-key-here",
        "file_encryption_key": "32-byte-file-encryption-key-here",
        "field_encryption_key": "32-byte-field-encryption-key"
    }'
```

#### Secrets Rotation

```yaml
# terraform/secrets-rotation.tf
resource "aws_secretsmanager_secret" "database_credentials" {
  name = "dealsphere/prod/database"
  description = "Production database credentials"

  replica {
    region = "us-east-1"
  }
}

resource "aws_secretsmanager_secret_rotation" "database_rotation" {
  secret_id           = aws_secretsmanager_secret.database_credentials.id
  rotation_lambda_arn = aws_lambda_function.rotation_lambda.arn

  rotation_rules {
    automatically_after_days = 30
  }
}

resource "aws_lambda_function" "rotation_lambda" {
  filename         = "rotation-lambda.zip"
  function_name    = "dealsphere-secret-rotation"
  role            = aws_iam_role.rotation_lambda_role.arn
  handler         = "lambda_function.lambda_handler"
  runtime         = "python3.9"
  timeout         = 30

  environment {
    variables = {
      SECRETS_MANAGER_ENDPOINT = "https://secretsmanager.us-west-2.amazonaws.com"
      EXCLUDE_CHARACTERS       = " %+~`#$&*()|[]{}:;<>?!'/\\\"@"
    }
  }
}
```

### Application Secrets Integration

```java
// Secrets manager integration
@Configuration
@EnableConfigurationProperties
public class SecretsConfiguration {

    @Bean
    public SecretsManagerClient secretsManagerClient() {
        return SecretsManagerClient.builder()
            .region(Region.US_WEST_2)
            .build();
    }

    @Bean
    @ConfigurationProperties(prefix = "app.secrets")
    public SecretsProperties secretsProperties() {
        return new SecretsProperties();
    }
}

@Service
public class SecretsService {
    private final SecretsManagerClient secretsClient;
    private final ObjectMapper objectMapper;
    private final Map<String, Object> secretsCache = new ConcurrentHashMap<>();

    public SecretsService(SecretsManagerClient secretsClient) {
        this.secretsClient = secretsClient;
        this.objectMapper = new ObjectMapper();
    }

    @Cacheable(value = "secrets", key = "#secretName")
    public <T> T getSecret(String secretName, Class<T> type) {
        try {
            GetSecretValueRequest request = GetSecretValueRequest.builder()
                .secretId(secretName)
                .build();

            GetSecretValueResponse response = secretsClient.getSecretValue(request);
            String secretValue = response.secretString();

            if (type == String.class) {
                return type.cast(secretValue);
            } else {
                return objectMapper.readValue(secretValue, type);
            }
        } catch (Exception e) {
            throw new SecretRetrievalException("Failed to retrieve secret: " + secretName, e);
        }
    }

    public DatabaseCredentials getDatabaseCredentials() {
        return getSecret("dealsphere/prod/database", DatabaseCredentials.class);
    }

    public JwtSecrets getJwtSecrets() {
        return getSecret("dealsphere/prod/jwt", JwtSecrets.class);
    }

    public String getApiKey(String serviceName) {
        ExternalApiSecrets secrets = getSecret("dealsphere/prod/external-apis", ExternalApiSecrets.class);
        return secrets.getApiKey(serviceName);
    }

    @EventListener(ContextRefreshedEvent.class)
    public void warmUpSecrets() {
        // Pre-load critical secrets at startup
        getDatabaseCredentials();
        getJwtSecrets();
    }
}

// Data classes for secrets
@Data
public class DatabaseCredentials {
    private String username;
    private String password;
    private String host;
    private int port;
    private String database;
    private String sslMode;
}

@Data
public class JwtSecrets {
    private String signingKey;
    private String encryptionKey;
    private String refreshKey;
}
```

### Kubernetes Secrets Management

#### External Secrets Operator

```yaml
# external-secrets/secret-store.yml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
  namespace: dealsphere-prod
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-west-2
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa
---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
  namespace: dealsphere-prod
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: database-credentials
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: dealsphere/prod/database
      property: username
  - secretKey: password
    remoteRef:
      key: dealsphere/prod/database
      property: password
  - secretKey: host
    remoteRef:
      key: dealsphere/prod/database
      property: host
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-secrets-sa
  namespace: dealsphere-prod
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT:role/external-secrets-role
```

## Access Control and Authentication

### JWT Security Implementation

```java
// Enhanced JWT service with encryption
@Service
public class JwtService {
    private final JwtSecrets jwtSecrets;
    private final Algorithm signingAlgorithm;
    private final JWEAlgorithm encryptionAlgorithm;

    public JwtService(SecretsService secretsService) {
        this.jwtSecrets = secretsService.getJwtSecrets();
        this.signingAlgorithm = Algorithm.HMAC256(jwtSecrets.getSigningKey());
        this.encryptionAlgorithm = JWEAlgorithm.A256KW;
    }

    public String generateAccessToken(UserPrincipal user) {
        try {
            // Create JWT claims
            String token = JWT.create()
                .withIssuer("dealsphere")
                .withSubject(user.getId().toString())
                .withClaim("email", user.getEmail())
                .withClaim("roles", user.getRoles().stream()
                    .map(Role::getName)
                    .collect(Collectors.toList()))
                .withClaim("organizationId", user.getOrganizationId().toString())
                .withIssuedAt(new Date())
                .withExpiresAt(new Date(System.currentTimeMillis() + JWT_EXPIRATION))
                .withJWTId(UUID.randomUUID().toString())
                .sign(signingAlgorithm);

            // Encrypt the JWT
            return encryptJWT(token);
        } catch (Exception e) {
            throw new TokenGenerationException("Failed to generate access token", e);
        }
    }

    public String generateRefreshToken(UserPrincipal user) {
        try {
            String token = JWT.create()
                .withIssuer("dealsphere")
                .withSubject(user.getId().toString())
                .withClaim("type", "refresh")
                .withIssuedAt(new Date())
                .withExpiresAt(new Date(System.currentTimeMillis() + REFRESH_EXPIRATION))
                .withJWTId(UUID.randomUUID().toString())
                .sign(Algorithm.HMAC256(jwtSecrets.getRefreshKey()));

            return encryptJWT(token);
        } catch (Exception e) {
            throw new TokenGenerationException("Failed to generate refresh token", e);
        }
    }

    public DecodedJWT validateAndDecodeToken(String encryptedToken) {
        try {
            // Decrypt the JWT
            String token = decryptJWT(encryptedToken);

            // Verify signature and decode
            JWTVerifier verifier = JWT.require(signingAlgorithm)
                .withIssuer("dealsphere")
                .build();

            return verifier.verify(token);
        } catch (Exception e) {
            throw new TokenValidationException("Invalid token", e);
        }
    }

    private String encryptJWT(String jwt) throws Exception {
        // Use JWE with A256KW algorithm
        JWEHeader header = new JWEHeader(encryptionAlgorithm, EncryptionMethod.A256GCM);
        Payload payload = new Payload(jwt);

        JWEObject jweObject = new JWEObject(header, payload);
        jweObject.encrypt(new AESEncrypter(jwtSecrets.getEncryptionKey().getBytes()));

        return jweObject.serialize();
    }

    private String decryptJWT(String encryptedJWT) throws Exception {
        JWEObject jweObject = JWEObject.parse(encryptedJWT);
        jweObject.decrypt(new AESDecrypter(jwtSecrets.getEncryptionKey().getBytes()));

        return jweObject.getPayload().toString();
    }
}
```

### Multi-Factor Authentication

```java
// TOTP-based MFA implementation
@Service
public class MfaService {
    private final UserRepository userRepository;
    private final SecretGenerator secretGenerator;

    public MfaSetupResponse setupMfa(UUID userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException("User not found"));

        // Generate secret key
        String secretKey = secretGenerator.generate();

        // Store encrypted secret
        user.setMfaSecret(encryptSecret(secretKey));
        user.setMfaEnabled(false); // Enable after verification
        userRepository.save(user);

        // Generate QR code
        String qrCodeUrl = generateQrCodeUrl(user.getEmail(), secretKey);

        return MfaSetupResponse.builder()
            .secretKey(secretKey)
            .qrCodeUrl(qrCodeUrl)
            .backupCodes(generateBackupCodes(userId))
            .build();
    }

    public boolean verifyMfaToken(UUID userId, String token) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException("User not found"));

        if (!user.isMfaEnabled()) {
            throw new MfaNotEnabledException("MFA not enabled for user");
        }

        String secretKey = decryptSecret(user.getMfaSecret());

        // Verify TOTP token (allow for time skew)
        for (int i = -1; i <= 1; i++) {
            long timeWindow = System.currentTimeMillis() / 30000 + i;
            String expectedToken = generateTotp(secretKey, timeWindow);

            if (token.equals(expectedToken)) {
                return true;
            }
        }

        // Check backup codes
        return verifyBackupCode(userId, token);
    }

    private String generateTotp(String secretKey, long timeWindow) {
        try {
            byte[] data = ByteBuffer.allocate(8).putLong(timeWindow).array();
            Mac mac = Mac.getInstance("HmacSHA1");
            mac.init(new SecretKeySpec(Base32.decode(secretKey), "HmacSHA1"));
            byte[] hash = mac.doFinal(data);

            int offset = hash[hash.length - 1] & 0xF;
            int code = ((hash[offset] & 0x7F) << 24) |
                      ((hash[offset + 1] & 0xFF) << 16) |
                      ((hash[offset + 2] & 0xFF) << 8) |
                      (hash[offset + 3] & 0xFF);

            return String.format("%06d", code % 1000000);
        } catch (Exception e) {
            throw new MfaException("Failed to generate TOTP", e);
        }
    }
}
```

## Network Security

### VPC and Network Segmentation

```yaml
# terraform/vpc.tf
resource "aws_vpc" "dealsphere_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "dealsphere-vpc"
    Environment = "production"
  }
}

# Public subnets for load balancers
resource "aws_subnet" "public_subnets" {
  count = 3

  vpc_id            = aws_vpc.dealsphere_vpc.id
  cidr_block        = "10.0.${count.index + 1}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  map_public_ip_on_launch = true

  tags = {
    Name = "dealsphere-public-${count.index + 1}"
    Type = "public"
  }
}

# Private subnets for application servers
resource "aws_subnet" "private_subnets" {
  count = 3

  vpc_id            = aws_vpc.dealsphere_vpc.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "dealsphere-private-${count.index + 1}"
    Type = "private"
  }
}

# Database subnets (isolated)
resource "aws_subnet" "database_subnets" {
  count = 3

  vpc_id            = aws_vpc.dealsphere_vpc.id
  cidr_block        = "10.0.${count.index + 20}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "dealsphere-database-${count.index + 1}"
    Type = "database"
  }
}

# Security groups
resource "aws_security_group" "alb_sg" {
  name_prefix = "dealsphere-alb-"
  vpc_id      = aws_vpc.dealsphere_vpc.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "app_sg" {
  name_prefix = "dealsphere-app-"
  vpc_id      = aws_vpc.dealsphere_vpc.id

  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb_sg.id]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]  # VPC only
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "database_sg" {
  name_prefix = "dealsphere-db-"
  vpc_id      = aws_vpc.dealsphere_vpc.id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app_sg.id]
  }

  tags = {
    Name = "dealsphere-database-sg"
  }
}
```

### Web Application Firewall (WAF)

```yaml
# terraform/waf.tf
resource "aws_wafv2_web_acl" "dealsphere_waf" {
  name  = "dealsphere-waf"
  scope = "REGIONAL"

  default_action {
    allow {}
  }

  # Rate limiting rule
  rule {
    name     = "RateLimitRule"
    priority = 1

    action {
      block {}
    }

    statement {
      rate_based_statement {
        limit              = 2000
        aggregate_key_type = "IP"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "RateLimitRule"
      sampled_requests_enabled   = true
    }
  }

  # SQL injection protection
  rule {
    name     = "SQLInjectionRule"
    priority = 2

    action {
      block {}
    }

    statement {
      sqli_match_statement {
        field_to_match {
          body {}
        }
        text_transformation {
          priority = 0
          type     = "URL_DECODE"
        }
        text_transformation {
          priority = 1
          type     = "HTML_ENTITY_DECODE"
        }
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "SQLInjectionRule"
      sampled_requests_enabled   = true
    }
  }

  # XSS protection
  rule {
    name     = "XSSRule"
    priority = 3

    action {
      block {}
    }

    statement {
      xss_match_statement {
        field_to_match {
          body {}
        }
        text_transformation {
          priority = 0
          type     = "URL_DECODE"
        }
        text_transformation {
          priority = 1
          type     = "HTML_ENTITY_DECODE"
        }
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "XSSRule"
      sampled_requests_enabled   = true
    }
  }

  # IP whitelist rule
  rule {
    name     = "IPWhitelistRule"
    priority = 4

    action {
      allow {}
    }

    statement {
      ip_set_reference_statement {
        arn = aws_wafv2_ip_set.allowed_ips.arn
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "IPWhitelistRule"
      sampled_requests_enabled   = true
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "dealsphere-waf"
    sampled_requests_enabled   = true
  }
}

resource "aws_wafv2_ip_set" "allowed_ips" {
  name  = "allowed-ips"
  scope = "REGIONAL"

  ip_address_version = "IPV4"
  addresses = [
    "203.0.113.0/24",  # Office IP range
    "198.51.100.0/24"  # Additional trusted IPs
  ]
}
```

## Security Monitoring and Compliance

### Security Event Logging

```java
// Security audit logger
@Component
public class SecurityAuditLogger {
    private static final Logger securityLogger = LoggerFactory.getLogger("SECURITY");
    private final ObjectMapper objectMapper;

    public void logLoginAttempt(String email, String ipAddress, boolean success, String reason) {
        SecurityEvent event = SecurityEvent.builder()
            .eventType("LOGIN_ATTEMPT")
            .timestamp(Instant.now())
            .userId(email)
            .ipAddress(ipAddress)
            .success(success)
            .reason(reason)
            .userAgent(getCurrentUserAgent())
            .sessionId(getCurrentSessionId())
            .build();

        logSecurityEvent(event);
    }

    public void logAccessViolation(String userId, String resource, String action, String reason) {
        SecurityEvent event = SecurityEvent.builder()
            .eventType("ACCESS_VIOLATION")
            .timestamp(Instant.now())
            .userId(userId)
            .resource(resource)
            .action(action)
            .success(false)
            .reason(reason)
            .ipAddress(getCurrentIpAddress())
            .build();

        logSecurityEvent(event);
    }

    public void logDataAccess(String userId, String dataType, String operation, int recordCount) {
        SecurityEvent event = SecurityEvent.builder()
            .eventType("DATA_ACCESS")
            .timestamp(Instant.now())
            .userId(userId)
            .dataType(dataType)
            .operation(operation)
            .recordCount(recordCount)
            .success(true)
            .ipAddress(getCurrentIpAddress())
            .build();

        logSecurityEvent(event);
    }

    private void logSecurityEvent(SecurityEvent event) {
        try {
            String jsonEvent = objectMapper.writeValueAsString(event);
            securityLogger.info(jsonEvent);
        } catch (Exception e) {
            securityLogger.error("Failed to log security event", e);
        }
    }
}

@Data
@Builder
public class SecurityEvent {
    private String eventType;
    private Instant timestamp;
    private String userId;
    private String ipAddress;
    private String userAgent;
    private String sessionId;
    private String resource;
    private String action;
    private String dataType;
    private String operation;
    private Integer recordCount;
    private boolean success;
    private String reason;
}
```

### Compliance Monitoring

```yaml
# CloudTrail for API audit logging
resource "aws_cloudtrail" "dealsphere_audit" {
  name                         = "dealsphere-audit-trail"
  s3_bucket_name              = aws_s3_bucket.audit_logs.bucket
  include_global_service_events = true
  is_multi_region_trail       = true
  enable_logging              = true

  event_selector {
    read_write_type           = "All"
    include_management_events = true

    data_resource {
      type   = "AWS::S3::Object"
      values = ["${aws_s3_bucket.dealsphere_storage.arn}/*"]
    }
  }

  insight_selector {
    insight_type = "ApiCallRateInsight"
  }
}

# Config for compliance monitoring
resource "aws_config_configuration_recorder" "dealsphere_config" {
  name     = "dealsphere-config-recorder"
  role_arn = aws_iam_role.config_role.arn

  recording_group {
    all_supported                 = true
    include_global_resource_types = true
  }
}

resource "aws_config_config_rule" "s3_bucket_public_read_prohibited" {
  name = "s3-bucket-public-read-prohibited"

  source {
    owner             = "AWS"
    source_identifier = "S3_BUCKET_PUBLIC_READ_PROHIBITED"
  }

  depends_on = [aws_config_configuration_recorder.dealsphere_config]
}

resource "aws_config_config_rule" "encrypted_volumes" {
  name = "encrypted-volumes"

  source {
    owner             = "AWS"
    source_identifier = "ENCRYPTED_VOLUMES"
  }

  depends_on = [aws_config_configuration_recorder.dealsphere_config]
}
```

This comprehensive security guide ensures data protection at rest and in transit, secure secrets management, and compliance with industry security standards for the DealSphere platform.