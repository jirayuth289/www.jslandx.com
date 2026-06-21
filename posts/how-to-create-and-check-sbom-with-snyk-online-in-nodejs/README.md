# How to Generate and Check SBOM with Snyk in Node.js Online

To increase confidence in our Node.js project on production, we must scan third-party dependencies, source code, and image containers.

This guide focuses on scanning third-party dependencies to prevent security vulnerabilities.

---

## Steps

1. Use [@cyclonedx/cdxgen](https://www.npmjs.com/package/@cyclonedx/cdxgen) to generate an SBOM file.

2. Install the latest version:

```
npm i @cyclonedx/cdxgen
```

3. Generate the SBOM:

```
cdxgen -o bom.json
```

4. Open the [SBOM security checker](https://snyk.io/code-checker/sbom-security/) on Snyk.

5. Paste the file content to check.

6. Review the issue report.
