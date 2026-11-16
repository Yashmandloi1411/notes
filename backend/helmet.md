# Helmet.js Security Headers – Express.js

## 1. Introduction

[Helmet.js](https://helmetjs.github.io/) is a Node.js middleware that helps secure Express applications by setting HTTP security headers.

These headers instruct browsers how to handle website resources, scripts, cookies, frames, and cross-origin requests.

### Installation

```bash
npm install express helmet
```

---

## 2. Complete Helmet.js Configuration

The following example configures security headers for an Express.js application.

```javascript
const express = require("express");
const helmet = require("helmet");

const app = express();

// ============================================
// 1. STACK FINGERPRINTING
// Remove X-Powered-By: Express header
// ============================================
app.disable("x-powered-by");

// ============================================
// 2. HELMET SECURITY HEADERS
// ============================================
app.use(
  helmet({

    // 1. CROSS-SITE SCRIPTING (XSS)
    // Content Security Policy restricts where scripts,
    // styles, images and other resources can load from.
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:", "https:"],
        connectSrc: ["'self'"],
        objectSrc: ["'none'"],
        baseUri: ["'self'"],
        frameAncestors: ["'self'"],
      },
    },

    // 2. CLICKJACKING
    // X-Frame-Options: SAMEORIGIN
    // Prevents other websites from embedding your site
    // inside an iframe (except your own origin).
    xFrameOptions: {
      action: "sameorigin",
    },

    // 3. MIME SNIFFING
    // X-Content-Type-Options: nosniff
    // Prevents browsers from guessing file content types.
    xContentTypeOptions: true,

    // 4. SSL STRIPPING / MITM MITIGATION
    // Strict-Transport-Security (HSTS)
    // Tells browsers to use HTTPS for future visits.
    // Enable only when HTTPS is correctly configured.
    strictTransportSecurity: {
      maxAge: 31536000,
      includeSubDomains: true,
    },

    // 5. STACK FINGERPRINTING
    // Helmet removes X-Powered-By automatically.
    xPoweredBy: true,

    // 6. DNS PREFETCHING
    // X-DNS-Prefetch-Control: off
    // Disables browser DNS prefetching.
    xDnsPrefetchControl: {
      allow: false,
    },

    // 7. REFERRER LEAKAGE
    // Controls how much referrer information is shared.
    referrerPolicy: {
      policy: "strict-origin-when-cross-origin",
    },

    // 8. SPECTRE / MEMORY ATTACK MITIGATION
    // Helps isolate browsing contexts across origins.
    crossOriginOpenerPolicy: {
      policy: "same-origin",
    },

    // 9. CROSS-ORIGIN RESOURCE RESTRICTION
    // Controls which origins can load your resources.
    crossOriginResourcePolicy: {
      policy: "same-origin",
    },

    // 11. CROSS-ORIGIN EMBEDDING
    // Enables Cross-Origin-Embedder-Policy.
    // WARNING: May block third-party images, scripts,
    // fonts and other resources if not configured correctly.
    crossOriginEmbedderPolicy: true,

    // 12. IE LEGACY XSS FILTER
    // Helmet sets X-XSS-Protection: 0.
    // This disables the old buggy browser XSS filter.
    // It does NOT enable XSS protection.
    xXssProtection: true,

    // 13. FLASH / LEGACY PLUGIN ATTACKS
    // X-Permitted-Cross-Domain-Policies: none
    xPermittedCrossDomainPolicies: {
      permittedPolicies: "none",
    },

    // 14. IE FILE DOWNLOAD XSS
    // X-Download-Options: noopen
    // Legacy Internet Explorer protection.
    xDownloadOptions: true,

  })
);

// ============================================
// 10. ROGUE FEATURE ACCESS
// Permissions-Policy is configured manually.
// Restricts camera, microphone and geolocation.
// ============================================
app.use((req, res, next) => {
  res.setHeader(
    "Permissions-Policy",
    "camera=(), microphone=(), geolocation=()"
  );

  next();
});

// ============================================
// EXPRESS MIDDLEWARE
// ============================================
app.use(express.json());

// ============================================
// TEST ROUTE
// ============================================
app.get("/", (req, res) => {
  res.send("Server running with Helmet security!");
});

// ============================================
// START SERVER
// ============================================
const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## 3. Security Headers and Their Purpose

| # | Attack / Threat | HTTP Header | Purpose |
|---|---|---|---|
| 1 | Cross-Site Scripting (XSS) | `Content-Security-Policy` | Restricts resource and script sources |
| 2 | Clickjacking | `X-Frame-Options` | Controls iframe embedding |
| 3 | MIME Sniffing | `X-Content-Type-Options` | Prevents MIME type guessing |
| 4 | SSL Stripping / MITM | `Strict-Transport-Security` | Enforces HTTPS for future visits |
| 5 | Stack Fingerprinting | `X-Powered-By` | Removes Express technology disclosure |
| 6 | DNS Leakage / Prefetching | `X-DNS-Prefetch-Control` | Controls DNS prefetching |
| 7 | Referrer Leakage | `Referrer-Policy` | Controls referrer information sharing |
| 8 | Spectre / Memory Attacks | `Cross-Origin-Opener-Policy` | Isolates browsing contexts |
| 9 | Cross-Origin Data Theft | `Cross-Origin-Resource-Policy` | Restricts cross-origin resource loading |
| 10 | Rogue Feature Access | `Permissions-Policy` | Restricts camera, microphone and geolocation |
| 11 | Cross-Origin Embedding | `Cross-Origin-Embedder-Policy` | Controls cross-origin resource embedding |
| 12 | IE Legacy XSS | `X-XSS-Protection` | Disables the old browser XSS filter |
| 13 | Flash / Plugin Attacks | `X-Permitted-Cross-Domain-Policies` | Restricts legacy plugin cross-domain policies |
| 14 | IE File Download XSS | `X-Download-Options` | Restricts automatic file opening in legacy IE |

---

## 4. Important Security Concepts

### A. Content Security Policy (CSP)

CSP controls which sources the browser is allowed to load scripts, styles, images, and other resources from.

Example:

```javascript
contentSecurityPolicy: {
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    objectSrc: ["'none'"],
  },
}
```

- `'self'`: Allow resources from the same origin.
- `'none'`: Do not allow resources from any source.

**Note:** CSP is an additional XSS defense. It does not replace safe React rendering, input validation, output encoding, or HTML sanitization.

### B. Strict Transport Security (HSTS)

HSTS tells the browser to use HTTPS for future requests to your website.

```javascript
strictTransportSecurity: {
  maxAge: 31536000,
  includeSubDomains: true,
}
```

- `maxAge`: Remember the HTTPS policy for one year.
- `includeSubDomains`: Apply the policy to subdomains too.

**Note:** Configure HTTPS correctly before enabling HSTS. HSTS does not replace HTTP-to-HTTPS redirection.

### C. Cross-Origin Resource Policy (CORP)

Controls whether other origins can load your resources.

```javascript
crossOriginResourcePolicy: {
  policy: "same-origin",
}
```

### D. Permissions Policy

Restricts browser features such as camera, microphone, and geolocation.

```javascript
res.setHeader(
  "Permissions-Policy",
  "camera=(), microphone=(), geolocation=()"
);
```

---

## 5. How to Verify Security Headers

After starting your Express server:

1. Open your application in Chrome.
2. Press `F12`.
3. Open the **Network** tab.
4. Reload your application.
5. Select the document request.
6. Open **Response Headers**.

Check for headers such as:

```http
Content-Security-Policy
Strict-Transport-Security
X-Frame-Options
X-Content-Type-Options
Referrer-Policy
Cross-Origin-Opener-Policy
Cross-Origin-Resource-Policy
Permissions-Policy
```

---

## 6. Important Production Notes

- Customize CSP according to your frontend, API, images, scripts, and external resources.
- Enable HSTS only when HTTPS is correctly configured.
- Use `includeSubDomains` only when all relevant subdomains support HTTPS.
- COEP can block external resources. Enable it only when your application needs cross-origin isolation.
- CORP does not replace CORS, authentication, or authorization.
- Helmet does not replace input validation, authentication, authorization, rate limiting, or secure coding practices.
- Legacy headers mainly exist for older browsers and should not be treated as modern attack prevention.

---

## 7. Official References

- [Helmet.js Documentation](https://helmetjs.github.io/)
- [Helmet.js GitHub Repository](https://github.com/helmetjs/helmet)
- [MDN – HTTP Security Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)

**Key takeaway:** Helmet is a security middleware for Express.js that provides an important layer of browser-side protection through HTTP security headers. It is not a complete solution for every web security vulnerability.
