## DEVOPS NOTES

#### Compile Languages
Java, .NET or other compiled programming languages require a build stage to create an executable file. The resultant executable contains all the codes embedded, and the necessary library dependencies, which the application needs to run and work successfully.

#### Interpreted Languages

PHP, JavaScript or Python and other interpreted languages work directly without being built into an executable file. That is why we can easily deploy the entire code from git into var/www/html and immediately the webserver was able to render the pages in a browser.
There is a smarter way to package the entire application code, and track release versions. We can package the entire code and all its dependencies into an archive such as .tar.gz or .zip, so that it can be easily unpacked on a respective environment’s servers.

#### Continous Integration CI/ Continous Delivery CD

Continous Integration Involves
- Run tests locally
- Commit and push to Repository
- Compile code in CI
- Run further test in CI - static analysis, code coverage analysis,code smell analysis, compliance analysis, gnerate documentation from source code
- Continous Delivery/Deployment - Deployment of the Artifact. Continous delivery means the code is manually checked into production, while Continous deployment means it is fully automated

![](https://soms-public-assets.s3.amazonaws.com/p14-CI-Pipeline.png)

#### principles that define a reliable and robust CI/CD pipeline

- Maintain a code repository
- Automate build process
- Make builds self-tested
- Everyone commits to the baseline every day
- Every commit to baseline should be built
- Every bug-fix commit should come with a test case
- Keep the build fast
- Test in a clone of production environment
- Make it easy to get the latest deliverables
- Everyone can see the results of the latest build
- Automate deployment (if you are confident enough in your CI/CD pipeline and willing to go for a fully automated Continuous Deployment)

#### 13 DevOps Success Metrics

Ultimately, the goals of DevOps are enhanced Velocity, Quality, and Performance.
1. Deployment frequency: Tracking how often you do deployments is a good DevOps metric. Smaller deployments as often as possible
2. Lead time: the amount of time that occurs between starting on a work item until it is deployed.
3. Customer tickets(issues): serves as a good indicator of application quality and performance problems.
4. Percentage of passed automated tests: tracking how well automated tests work is a good DevOps metrics. It is good to know how often code changes break tests.
5. Defect escape rate: DevOps metric to track how often those defects make it to production
6. Availability: Tracking downtimes and unplanned outages. Most software companies build status pages to track/report this
7. Service level agreements: Tracking compliant Service Level Agreements, requirements implementation, customer expectations fulfillments
8. Failed Deployments: If you have issues with failed deployments, be sure to track this metric over time. This could also be seen as tracking *Mean Time To Failure (MTTF).
9. Error rate:  Tracking error rates within the application is super important. Not only they serve as an indicator of quality problems, but also ongoing performance and uptime related issues. Presenting error rate metrics simply gives greater insights into where to focus attention.
10. Application usage and traffic: After a deployment, we want to see if the number of transactions or users accessing our system looks normal. Sudden Spike or lag is not normal
11. Application performance: configure monitoring tools like Retrace, DataDog, New Relic, or AppDynamics to look for performance problems, hidden errors, and other issues before and after deployment
12. Mean time to detection (MTTD): Metric on how fast you are able to identify issues.
13. Mean time to recovery (MTTR): This metric helps us track how long it takes to recover from failures. It is typically measured in hours and may refer to business hours.

#### Testings
SIT - System Integration Test
UAT - User Interface Test
PENTEST - Penetration Testing - conduct security related tests. In some cases, it will also be used for Performance and Load testing.