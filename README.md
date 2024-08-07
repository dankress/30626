# 30626

Reproduction for [Renovate discussion 30626](https://github.com/renovatebot/renovate/discussions/30626).

## Current behavior

Renovate will try to upgrade the temurin jammy 21 jdk to version 22 even though that version is not LTS

## Expected behavior

I expected renovate to use the Java LTS workaround and only upgrade to version 21, as that is LTS

## Dockerfile 

```

# Stage 1: Build
#tag::build[]
FROM company.com/dockerhub/library/eclipse-temurin:21-jdk-jammy@sha256:7b9c017c1c7272e8768a59422a7c37a9c870c9eae9926f715d4278bc5c3c3b9d as build
ENV BUILDDIR=/tmp/build
RUN mkdir -p ${BUILDDIR}
COPY ./ ${BUILDDIR}/
WORKDIR ${BUILDDIR}/
# chmod required because Azure Agent user has no privileges
RUN chmod -R 777 ${BUILDDIR} && ./gradlew bootJar
#end::build[]
#tag::run[]
# Stage 2: App
FROM company.com/dockerhub/library/eclipse-temurin:21-jre-jammy@sha256:870aae69d4521fdaf26e952f8026f75b37cb721e6302d4d4d7100f6b09823057 as app
WORKDIR /opt
ARG VERSION
ENV VERSION=$VERSION
ENV BUILDDIR=/tmp/build
COPY --from=build ${BUILDDIR}/build/libs/service-$VERSION.jar .
ENTRYPOINT ["sh", "-c", "java -jar ./service-$VERSION.jar"]
#end::run[]
```
## Repository config renovate.json5

```
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>OTIC-2340_MFB/renovate"]
}
```

## default.json in renovate repository

```
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "osvVulnerabilityAlerts": true,
  "updateLockFiles": true,
  "extends": [
    "config:best-practices",
    ":semanticPrefixFixDepsChoreOthers",
    "group:monorepos",
    "group:recommended",
    "replacements:all",
    "workarounds:all",
    "security:openssf-scorecard"
  ],
  "ignoreDeps": [],
  "packageRules": [
    {
      "description": "Trigger breaking release for major updates",
      "matchUpdateTypes": ["major"],
      "semanticCommitType": "feat",
      "commitMessageSuffix": "BREAKING CHANGE: Major update"
    },
    {
      "description": "Automatically merge minor updates without notifying anyone",
      "matchUpdateTypes": ["minor", "patch", "digest"],
      "automerge": true,
      "automergeType": "branch"
    }
  ],
  "baseBranches": ["master"]
}
```

## Logs

```

DEBUG: getLabels(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, latest) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getManifestResponse(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, latest, get) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getManifestResponse(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, sha256:ff770fdb023e381d20c962331137e199006ef97028003770d50b825ff2cc6c40, get) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: found labels in manifest (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
       "labels": {
         "org.opencontainers.image.ref.name": "ubuntu",
         "org.opencontainers.image.version": "24.04"
       }
DEBUG: getDigest(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, 22-jdk-jammy) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getDigest(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, 22-jre-jammy) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getManifestResponse(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, sha256:7b9c017c1c7272e8768a59422a7c37a9c870c9eae9926f715d4278bc5c3c3b9d, head) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getManifestResponse(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, sha256:870aae69d4521fdaf26e952f8026f75b37cb721e6302d4d4d7100f6b09823057, head) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getManifestResponse(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, 22-jre-jammy, head) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getManifestResponse(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, 22-jdk-jammy, head) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: Got docker digest sha256:bbfa10d59f7b90568aa19a935b6112554798261b13e915cbb06e990ea7322c36 (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getDigest(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, 21-jre-jammy) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getManifestResponse(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, 21-jre-jammy, head) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: Got docker digest sha256:aa5d66004611f64f47891a7c5ab34edd6eac75b05b8a1f484b45928118a4f3eb (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getDigest(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, 21-jdk-jammy) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getManifestResponse(https://company_container_registry_harbor.com, dockerhub/library/eclipse-temurin, 21-jdk-jammy, head) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: Got docker digest sha256:870aae69d4521fdaf26e952f8026f75b37cb721e6302d4d4d7100f6b09823057 (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: Got docker digest sha256:7b9c017c1c7272e8768a59422a7c37a9c870c9eae9926f715d4278bc5c3c3b9d (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: PackageFiles.add() - Package file saved for base branch (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: Package releases lookups complete (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: branchifyUpgrades (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: detectSemanticCommits() (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: getCommitMessages (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: semanticCommits: detected "angular" (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: semanticCommits: enabled (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: 2 flattened updates found: company_container_registry_harbor/dockerhub/library/eclipse-temurin, company_container_registry_harbor/dockerhub/library/eclipse-temurin (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: Returning 1 branch(es) (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: config.repoIsOnboarded=true (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
DEBUG: packageFiles with updates (repository=OTIC-2340_MFB/devops-template-gradle, baseBranch=master)
       "config": {
         "dockerfile": [
           {
             "deps": [
               {
                 "depName": "company_container_registry_harbor/dockerhub/library/eclipse-temurin",
                 "currentValue": "21-jdk-jammy",
                 "currentDigest": "sha256:7b9c017c1c7272e8768a59422a7c37a9c870c9eae9926f715d4278bc5c3c3b9d",
                 "replaceString": "company_container_registry_harbor/dockerhub/library/eclipse-temurin:21-jdk-jammy@sha256:7b9c017c1c7272e8768a59422a7c37a9c870c9eae9926f715d4278bc5c3c3b9d",
                 "autoReplaceStringTemplate": "{{depName}}{{#if newValue}}:{{newValue}}{{/if}}{{#if newDigest}}@{{newDigest}}{{/if}}",
                 "datasource": "docker",
                 "depType": "stage",
                 "updates": [
                   {
                     "bucket": "major",
                     "newVersion": "22",
                     "newValue": "22-jdk-jammy",
                     "newMajor": 22,
                     "newMinor": null,
                     "newPatch": null,
                     "updateType": "major",
                     "newDigest": "sha256:aa5d66004611f64f47891a7c5ab34edd6eac75b05b8a1f484b45928118a4f3eb",
                     "branchName": "renovate/company_container_registry_harbor-dockerhub-library-eclipse-temurin-22.x"
                   }
                 ],
                 "packageName": "company_container_registry_harbor/dockerhub/library/eclipse-temurin",
                 "versioning": "docker",
                 "warnings": [],
                 "registryUrl": "https://company_container_registry_harbor.com",
                 "lookupName": "dockerhub/library/eclipse-temurin",
                 "currentVersion": "21",
                 "isSingleVersion": true,
                 "fixedVersion": "21-jdk-jammy"
               },
               {
                 "depName": "company_container_registry_harbor/dockerhub/library/eclipse-temurin",
                 "currentValue": "21-jre-jammy",
                 "currentDigest": "sha256:870aae69d4521fdaf26e952f8026f75b37cb721e6302d4d4d7100f6b09823057",
                 "replaceString": "company_container_registry_harbor/dockerhub/library/eclipse-temurin:21-jre-jammy@sha256:870aae69d4521fdaf26e952f8026f75b37cb721e6302d4d4d7100f6b09823057",
                 "autoReplaceStringTemplate": "{{depName}}{{#if newValue}}:{{newValue}}{{/if}}{{#if newDigest}}@{{newDigest}}{{/if}}",
                 "datasource": "docker",
                 "depType": "final",
                 "updates": [
                   {
                     "bucket": "major",
                     "newVersion": "22",
                     "newValue": "22-jre-jammy",
                     "newMajor": 22,
                     "newMinor": null,
                     "newPatch": null,
                     "updateType": "major",
                     "newDigest": "sha256:bbfa10d59f7b90568aa19a935b6112554798261b13e915cbb06e990ea7322c36",
                     "branchName": "renovate/company_container_registry_harbor-dockerhub-library-eclipse-temurin-22.x"
                   }
                 ],
                 "packageName": "company_container_registry_harbor/dockerhub/library/eclipse-temurin",
                 "versioning": "docker",
                 "warnings": [],
                 "registryUrl": "https://company_container_registry_harbor.com",

```



## Link to the Renovate issue or Discussion

Reproduction for [Renovate discussion 30626](https://github.com/renovatebot/renovate/discussions/30626).
