# Pipeline Setup

This guide will help you set up the pipeline for a Java/Maven or Node.js project from scratch.

## 1. Prerequisites

- Developer or lead-developer access in NR Broker. To get access, contact the Product Owner.
- Ensure your NR Broker Account is linked to GitHub (see [How to Link Your Broker Account to GitHub](https://apps.nrs.gov.bc.ca/int/confluence/display/OSCAR/Linking+to+a+GitHub+account)).
- Install either [Node.js](https://nodejs.org/) **or** [Docker](https://www.docker.com/) / [Podman](https://podman.io/) (required for running `nr-repository-composer`).

## 2. Gather required information

Before proceeding, collect the following information:

- **Project:** Name of your project in NR Broker.
- **Service:** Name of the service in NR Broker.
- **License (SPDX):** The SPDX identifier for your project's license.
- **Client ID:** The Client ID for the NR Broker account.
- **Path to pom.xml:** Relative path to your `pom.xml` file.
- **Path to unit tests:** Optional path to your unit test workflow (e.g., `./.github/workflows/test.yaml`).
- **Publish to GitHub Packages:** Indicate if you want to publish to GitHub Packages (yes/no).
- **GitHub Owner with repo path:** Full GitHub org and repository path (e.g., `bcgov-c/nr-results`).
- **Deploy on-prem:** Indicate if on-premises deployment is required (yes/no).
- **Deploy Jasper Reports:** Indicate if Jasper Reports deployment is needed (yes/no).
- **Playbook path:** Path to your deployment playbook.
- **Tomcat Context:** Tomcat context path (e.g., `ext#results`).
- **Use alternative webapp directory:** Indicate if an alternative webapp directory should be used (yes/no).
- **Add Webade configuration:** Indicate if Webade configuration is required (yes/no).

**IMPORTANT:**

- The project and service names must match those in NR Broker.
- To find the Client ID, go to [NR Broker](https://broker.io.nrs.gov.bc.ca/) and search for the broker account. The Client ID is displayed in the Details panel. To copy the Client ID to the clipboard, click Details > Copy > Client ID.

## 3. Add pipeline files

To set up the pipeline, add the required files to the repository.

- Create a new branch from `main`.
- Run the following command in your project root to generate the required files for a Java/Maven project:
```sh
podman run --rm -it -v ${PWD}:/src --userns keep-id ghcr.io/bcgov/nr-repository-composer:latest nr-repository-composer:gh-maven-build
```
- Run the following command in your project root to generate the required files for a Node.js project:
```sh
podman run --rm -it -v ${PWD}:/src --userns keep-id ghcr.io/bcgov/nr-repository-composer:latest nr-repository-composer:gh-nodejs-build
```
- Answer the prompts using the information gathered in the previous step.

## 4. Configure Maven

For a Java/Maven project, configure Maven:

- Edit your `pom.xml`:
- Add a `<repositories>` section to your `pom.xml` to include Maven Central and NR Artifactory:

```xml
<repositories>
  <repository>
    <id>central</id>
    <url>https://repo.maven.apache.org/maven2/</url>
    <releases>
      <enabled>true</enabled>
    </releases>
    <snapshots>
      <enabled>false</enabled>
    </snapshots>
  </repository>
  <repository>
    <id>nr-artifactory</id>
    <name>NR Artifactory</name>
    <url>https://artifacts.developer.gov.bc.ca/artifactory/cc20-gen-maven-local</url>
    <releases>
      <enabled>true</enabled>
    </releases>
    <snapshots>
      <enabled>false</enabled>
    </snapshots>
  </repository>
</repositories>
```

- Add a GitHub profile for publishing artifacts and replace the org/repo with yours:

```xml
<profiles>
  <profile>
    <id>github</id>
    <distributionManagement>
      <repository>
        <id>github</id>
        <name>GitHub Packages</name>
        <url>https://maven.pkg.github.com/bcgov-c/nr-rept</url>
        <releases>
          <enabled>true</enabled>
        </releases>
        <snapshots>
          <enabled>true</enabled>
        </snapshots>
      </repository>
    </distributionManagement>
  </profile>
</profiles>
```

## 5. Merge pipeline files to main

- Commit and push the generated files.
- Open a pull request and merge to the `main` branch.
- Merging to the `main` branch will trigger a build.
- Build and deploy workflows will become available after merging.

## 6. Building packages

Building new packages follows the same general process:

- Create and push a new branch from `main`.
- Open a pull request to the `main` branch.
- Push changes as needed until the build is successful.
- Builds are triggered automatically for each push.
- When the build is successful, you may deploy the artifact (see [DEPLOY.md](DEPLOY.md)).

## References

- [nr-repository-composer](https://github.com/bcgov/nr-repository-composer)
- [Polaris Collection README](https://github.com/bcgov/nr-polaris-collection/blob/main/README.md)
