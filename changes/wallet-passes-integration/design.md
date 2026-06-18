# Design: Wallet Passes Integration (Google & Apple Wallet)

## Architecture Overview

```
                 ┌──────────────────────────────────────┐
                 │       elitepass-reservas / POS       │
                 └──────────────────┬───────────────────┘
                                    │
                         GET /api/v1/passes/download
                                    │
                                    ▼
                 ┌──────────────────────────────────────┐
                 │        elitepass-billeteraQR         │
                 │            (Microservicio)           │
                 └──────┬────────────────────────┬──────┘
                        │                        │
             Apple Sign (openssl/zip)    Google JWT Sign
                        │                        │
                        ▼                        ▼
               [ Apple Wallet App ]     [ Google Wallet App ]
                   (.pkpass)                  (JWT URL)
```

## Technical Decisions

### 1. Standalone Microservice: `elitepass-billeteraQR`
- **Framework:** Node.js with TypeScript and Express.
- **Ports & Routing:** Runs on port `3300` (proxied under `/billetera-api/`).
- **Isolation:** Separates signature credentials (certificates, private keys, service account JSON files) from the main Next.js repository.

### 2. Apple Wallet (.pkpass) Generation
- **Format:** Zip archive containing `pass.json`, static assets (logo, icon, strip image), a `manifest.json` (SHA-1 hashes of all files), and a detached PKCS#7 cryptographic signature (`signature`).
- **Certificates Required:**
  - `pass.com.genial-it.elitepass.pem`: Pass Type ID certificate from Apple Developer account.
  - `privateKey.pem`: Private key matching the certificate.
  - `wwdr.pem`: Apple Worldwide Developer Relations G3 certification authority.
- **Library:** `passkit-generator` (clean TypeScript API to load certificates and zip the assets).

### 3. Google Wallet Pass Generation
- **Model:** Google Wallet API uses a Class/Object model. We will define a `GenericClass` or `LoyaltyClass` on the Google Pay Console.
- **Flow:**
  - Create a Service Account on Google Cloud Console.
  - Share the Google Pay Developer Console with the service account email.
  - On pass request, `elitepass-billeteraQR` signs a JWT containing the user-specific Pass Object payload using the service account private key.
  - Returns a redirection URL: `https://pay.google.com/gp/v/save/<JWT>`.

### 4. Data Mapping & Customization
- **Primary Field:** Client Name (`nombre` + `apellido`).
- **Secondary Fields:** 
  - VIP / Rol Status (VIP, Interno, Externo).
  - Main Organization / Company name.
- **Back Fields (Apple) / Info Modules (Google):**
  - List of affiliated companies (`empresas` list).
  - Date of creation/activation.
- **Barcode/QR Code:**
  - Format: `QR_CODE`.
  - Content: Universal QR payload matching the scanner's expected structure (usually user ID or JWT).
- **Styling & Custom Images:**
  - Custom background/strip image loaded dynamically based on organization branding or default acromatics (zinc/grayscale) design tokens.
  - User avatar loaded from Azure Blob Storage and converted to WebP/PNG to fit the pass layout.
