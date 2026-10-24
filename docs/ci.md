# Build and test

Install the .NET SDK selected by `global.json` (6.0.428). All four projects
target .NET 6. The two NUnit test projects use EF Core InMemory; the tests do
not need SQL Server or application secrets.

From the repository root:

```text
dotnet restore Homies.sln
dotnet build Homies.sln --configuration Release --no-restore
dotnet test Homies.sln --configuration Release --no-build --no-restore --logger trx --results-directory TestResults
```

The solution includes both `Homies.Tests` and `Homies.IntegrationTests`, so the
last command runs all supplied tests. TRX files are written to `TestResults`.

## GitHub Actions

`.github/workflows/ci.yml` runs these commands on pushes to `develop` and
`feature/**`. It installs the selected SDK on Ubuntu 22.04 and uploads TRX
files as the `test-results` artifact, including when tests fail.

## Jenkins

1. Install Jenkins with the Pipeline, Git, and GitHub plugins.
2. Provide a Windows or Linux agent with Git and .NET SDK 6.0.428 on `PATH`.
   Linux agents need the operating system dependencies for .NET 6.
3. Create a **Pipeline** job, select **Pipeline script from SCM**, choose Git,
   enter this repository's clone URL, set **Branches to build** to `*/main`,
   and set **Script Path** to `Jenkinsfile`.
4. Run the job once to register its triggers. Jenkins polls SCM every two
   minutes, so a local Jenkins instance can detect pushes without a public
   endpoint. For immediate builds when Jenkins is reachable from GitHub,
   configure a repository webhook with the Jenkins URL followed by
   `/github-webhook/`, JSON content type, and push events. Use the matching
   webhook secret if Jenkins requires one; `githubPush()` handles these events.
5. Push to `main`. Jenkins checks out SCM, restores, builds, and runs both test
   projects. It fails on a nonzero command exit and archives available TRX
   results from the test stage. The branch guard also supports a Multibranch
   Pipeline's `BRANCH_NAME` when that job type is used.

The Jenkinsfile uses `sh` on Linux and `bat` on Windows. A repository file
alone does not create the Jenkins job or the GitHub webhook.

## Verified exam run on 24 October 2026

The solution passed all 9 supplied tests: 4 unit tests and 5 integration
tests. The Release build completed with zero errors and 32 existing warnings.
GitHub Actions completed successfully on both configured branch patterns:

- [develop run](https://github.com/BojidarDermendjiev/SEDO-09-2026-Regular-Exam-2/actions/runs/40070140468)
- [feature/ci-pipelines run](https://github.com/BojidarDermendjiev/SEDO-09-2026-Regular-Exam-2/actions/runs/40070137661)

A local Jenkins Pipeline job named `SEDO-Exam-2` is configured to track main.
Build 3 restored, built, passed all 9 tests, and archived both TRX reports.
It was started manually after publishing the pipeline. The SCM poll schedule
is configured separately to detect future main pushes while Jenkins runs.
The controller listens on localhost, so its build URL is accessible only on
the machine running Jenkins.
