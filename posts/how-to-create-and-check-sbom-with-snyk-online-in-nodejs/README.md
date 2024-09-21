# How to generate and check SBOM with snyk in nodejs online

To increase confidence in our nodejs project on production, we must scan third-party dependencies, source code, and image containers.

Focus on scanning third-party dependencies to prevent security vulnerabilities by following the step.

1. we use @cyclonedx/cdxgen to generate SBOM file https://www.npmjs.com/package/@cyclonedx/cdxgen

2. install latest version

    ```npm i @cyclonedx/cdxgen```

3. use command

    ```cdxgen -o bom.json```

4. go to SBOM security checker https://snyk.io/code-checker/sbom-security/

5. paste content file to check

6. check the issue report