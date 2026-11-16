
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

