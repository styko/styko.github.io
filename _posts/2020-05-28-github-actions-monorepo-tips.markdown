---
layout: post
title:  "Github actions monorepo tips"
description: Github actions monorepo tips
date:   2020-05-28 17:03:36 +0200
categories: Github actions monorepo
---
I wanted to increase my knowledge about monorepos and play little bit with on Github actions to create CI/CD pipeline.

Tip #1
when you want to execute action only when particular directory was changed on push/pull_request paths can be used.

```yaml
name: realestate_email_aggregator_fe_vuejs

on:
  push:
    paths:
      - 'realestate_email_aggregator_fe_vuejs/**'
  pull_request:
    paths:
      - 'realestate_email_aggregator_fe_vuejs/**'
```

Tip #2
in order to define folder where job is executed. This can be defined for all or just one particullar job.

```yaml
defaults:
  run:
    working-directory: realestate_email_aggregator_fe_vuejs
```
More about that here:
[https://dev.to/shofol/run-your-github-actions-jobs-from-a-specific-directory-1i9e](https://dev.to/shofol/run-your-github-actions-jobs-from-a-specific-directory-1i9e)
[https://github.community/t/use-working-directory-for-entire-job/16747](https://github.community/t/use-working-directory-for-entire-job/16747)

Tip #3
in order to SonarCloud analyze your project this action can be used [https://github.com/SonarSource/sonarcloud-github-action](https://github.com/SonarSource/sonarcloud-github-action) .
You also need to provide project base dir.

```yaml
    - name: SonarCloud Scan
      uses: sonarsource/sonarcloud-github-action@master
      with:
        projectBaseDir: realestate_email_aggregator_fe_vuejs
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

You need to have sonar-project.properties at directory level where is your project
```
sonar.organization=styko
sonar.projectKey=realestate_email_aggregator_fe_vuejs
sonar.language=js
sonar.sources=src
sonar.tests=tests
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

Tip #4
in order to SonarCloud actually read your coverage 

```yaml
- name: fix code coverage paths
  run: |
    sed -i 's+/home/runner/work/realestate_email_aggregator/realestate_email_aggregator+/github/workspace+g' coverage/lcov.info
```
Since sonar was starting a Docker container and set the workdir to /github/workspace and therefore could not locate the files from the lcov.info.

More about that here:
[https://community.sonarsource.com/t/code-coverage-doesnt-work-with-github-action/16747/5](https://community.sonarsource.com/t/code-coverage-doesnt-work-with-github-action/16747/5)
[https://stackoverflow.com/questions/58871955/sonarcloud-code-coverage-remains-0-0-in-github-actions-build](https://stackoverflow.com/questions/58871955/sonarcloud-code-coverage-remains-0-0-in-github-actions-build)


