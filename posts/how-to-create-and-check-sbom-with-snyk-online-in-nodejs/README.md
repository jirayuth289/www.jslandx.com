# How to generate and check SBOM with snyk in nodejs online

To increase confidence in our nodejs project on production, we must scan third-party dependencies, source code, and image containers.
<br/><br/>
Focus on scanning third-party dependencies to prevent security vulnerabilities by following the step.
<br/>
1. we use @cyclonedx/cdxgen to generate SBOM file https://www.npmjs.com/package/@cyclonedx/cdxgen
<br/><br/>
2. install latest version
<br/><br/>
```
npm i @cyclonedx/cdxgen
```
<br/>
3. use command
```
cdxgen -o bom.json
```
<br/><br/>
4. go to SBOM security checker https://snyk.io/code-checker/sbom-security/
<br/><br/>
5. paste content file to check
<br/><br/>
6. check the issue report