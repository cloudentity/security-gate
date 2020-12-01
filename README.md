# security-gate
Cloudentity development artifacts security gate, suppressions, whitelist etc

## Security Gates and tools used

Vulnerabilities with HIGH and CRITICAL are given top priority for fixes. All the vulnerabilities may not
be directly exploitable and details will be published if any new vulnerability shows up in the tracker that
requires a new patch(in case it is exploitable)

### OWASP dependency check

Cloudentity build pipeline has the OWASP dependency check plugin enabled to ensure any new vulnerabilities
reported are rectified in product software release and patches

OWASP Maven plugin used by Cloudentity projects

```
 <plugin>
        <groupId>org.owasp</groupId>
        <artifactId>dependency-check-maven</artifactId>
        <version>6.0.3</version>
        <configuration>
          <skipProvidedScope>false</skipProvidedScope>
          <skipRuntimeScope>false</skipRuntimeScope>
          <formats>
            <format>HTML</format>
            <format>JUNIT</format>
          </formats>
          <cveUrlBase>https://nvdcve.artifactory.syntegrity.com/nvdcve-1.0-%d.json.gz</cveUrlBase>
          <cveUrlModified>https://nvdcve.artifactory.syntegrity.com/nvdcve-1.0-modified.json.gz</cveUrlModified>
          <suppressionFiles>
            <suppressionFile>https://raw.githubusercontent.com/cloudentity/security-gate/dev/owasp/dependency-check/project-suppression.xml</suppressionFile>
          </suppressionFiles>
        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>aggregate</goal>
            </goals>
          </execution>
        </executions>
      </plugin>

```

NVDCDE MIRROR: https://nvd.nist.gov/vuln/data-feeds#JSON_FEED

### OWASP Dependency Track

[OWASP dependency track](https://owasp.org/www-project-dependency-track/) is used to keep track of past 
and current vulnerabilities across software modules used within Cloudentity software. This allows to
identity, keep track and reduce risk from the use of third party and opensource components.
 Daily build and release build artifacts are automatically published to an internally hosted OWASP DT
 server for analysis and version audits. 
[Software bill of materials - SBOM](https://owasp.org/www-community/Component_Analysis#software-bill-of-materials-sbom) is generated 
in [CycloneDX](https://cyclonedx.org/) format and then uploaded to OWASP DT server for [Component Analysis](https://owasp.org/www-community/Component_Analysis)
 
. Create SBOM
```
 <plugin>
    <groupId>org.cyclonedx</groupId>
    <artifactId>cyclonedx-maven-plugin</artifactId>
    <version>1.6.4</version>
    <inherited>false</inherited>
    <executions>
      <execution>
        <phase>deploy</phase>
        <goals>
          <goal>makeAggregateBom</goal>
        </goals>
      </execution>
    </executions>
    <configuration>
      <schemaVersion>1.1</schemaVersion>
      <includeBomSerialNumber>true</includeBomSerialNumber>
      <includeCompileScope>true</includeCompileScope>
      <includeProvidedScope>true</includeProvidedScope>
      <includeRuntimeScope>true</includeRuntimeScope>
      <includeSystemScope>true</includeSystemScope>
      <includeTestScope>false</includeTestScope>
      <includeDependencyGraph>true</includeDependencyGraph>
      <includeLicenseText>false</includeLicenseText>
    </configuration>
  </plugin>
  ```
  
. Publish SBOM to OWASP DT 
  ```
    <plugin>
      <groupId>io.github.pmckeown</groupId>
      <artifactId>dependency-track-maven-plugin</artifactId>
      <version>0.8.1</version>
      <inherited>false</inherited>
      <executions>
        <execution>
          <phase>deploy</phase>
          <goals>
            <goal>delete-project</goal>
            <goal>upload-bom</goal>
          </goals>
        </execution>
      </executions>
      <configuration>
        <dependencyTrackBaseUrl>${owaspdt.url}</dependencyTrackBaseUrl>
        <apiKey>${owaspdt.api.key}</apiKey>
      </configuration>
    </plugin>
```

### Golint checker

[Golint checker](https://github.com/cloudentity/acp/blob/dev/.golangci.yml) is used within projects written 
in Golang to check for package vulenerabilities.This is also designed to run on every commit, build and release
pipelines within the software development model.

### Anchore

[Anchore](https://anchore.com/opensource/) is used for analysis and inspection of Cloudentity container images. 
Anchore is integrated into daily build and release build pipelines within the software development model.
