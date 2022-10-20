# EXPERIENCE CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP

### project 14

Some notes on [devops](/devops-notes.md)

![](https://soms-public-assets.s3.amazonaws.com/images/p14_CI_CD-Pipeline-For-PHP-ToDo-Application.png)

**Environment summary**
What we want to achieve, is having Nginx to serve as a reverse proxy for our sites and tools.
**NOTE:** System Integration Test (SIT) and User Acceptance Test (UAT) can be done with one server, since both are basically are intra-application tests(the application itself). We don't really need any extra installation to do a SIT/UAT test.

However, Pentest, some other tools and specific configurations will be needed. In some cases, it will used for **Performance and Load** testing.
![](https://soms-public-assets.s3.amazonaws.com/images/p14-environment-setup.png)

**CI Setup**
![](https://soms-public-assets.s3.amazonaws.com/images/p14-CI-Environment.png)

**Pentest Setup**
![](https://soms-public-assets.s3.amazonaws.com/images/p14-Pentest-Environment.png)

#### Create Domains

| Server      | Domain |
| ----------- | ----------- |
| Jenkins     | https://ci.infradev.chisomejim.link   |
| Sonarqube   | https://sonar.infradev.chisomejim.link |
| Artifactory | https://artifacts.infradev.chisomejim.link |
| Production Tooling | https://tooling.chisomejim.link |
| Pre-Prod Tooling | https://tooling.preprod.chisomejim.link |
| Pentest Tooling | https://tooling.pentest.chisomejim.link |
| UAT Tooling | https://tooling.uat.chisomejim.link |
| SIT Tooling | https://tooling.sit.chisomejim.link |
| Dev Tooling | https://tooling.dev.chisomejim.link |
| Production TODO-WebApp | https://todo.chisomejim.link |
| Pre-Prod TODO-WebApp | https://todo.preprod.chisomejim.link |
| Pentest TODO-WebApp | https://todo.pentest.chisomejim.link |
| UAT TODO-WebApp | https://todo.uat.chisomejim.link |
| SIT TODO-WebApp | https://todo.sit.chisomejim.link |
| Dev TODO-WebApp | https://todo.dev.chisomejim.link |
