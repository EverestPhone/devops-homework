# devops-homework — Pipeline Repository

CI/CD pipelines and documentation for the DevOps Engineer homework assignment.

## Repository structure

```
devops-homework/
├── taskB/
│   └── Jenkinsfile          # PipelineB — Doxygen doc generation + artifact
├── taskC/
│   └── Jenkinsfile          # PipelineC — Extends B + Python warnings parser
└── taskD/
    └── README.md            # Written answers to homework questions
```

## Branch layout

| Branch | Contents |
|--------|---------|
| `main` | This README |
| `TaskB` | `taskB/Jenkinsfile` |
| `TaskC` | `taskC/Jenkinsfile` |
| `TaskD` | `taskD/README.md` |

## Related repositories

| Repo | Description |
|------|-------------|
| [grpc (fork)](https://github.com/EverestPhone/grpc) | RepoA — forked C++ source for Doxygen |
| [doxygen-log-parser](https://github.com/EverestPhone/doxygen-log-parser) | RepoC — Python warnings parser |

#### Python Development
- [ ] Python 3.8+ installed
- [ ] Virtual environment created
- [ ] Dependencies installed (`pip install -r requirements.txt`)
- [ ] All 9 unit tests passing
- [ ] Coverage > 95%
- [ ] Sample data tested
- [ ] CSV output validated

### Phase 2: GitHub Setup (Est. 1-2 hours)

#### RepoA (Fork)
- [ ] grpc repository forked
- [ ] Accessible from your account
- [ ] Can clone locally
- [ ] Upstream configured (optional)

#### RepoC (Python Parser)
- [ ] Repository created
- [ ] Code pushed to main branch
- [ ] .gitignore configured
- [ ] LICENSE file added
- [ ] README.md complete
- [ ] GitHub Actions workflow added

#### Pipeline Repository
- [ ] Repository created
- [ ] Main branch with main README
- [ ] TaskB branch with Jenkinsfile
- [ ] TaskC branch with Jenkinsfile
- [ ] TaskD branch with answers
- [ ] Branch protection rules configured
- [ ] Webhooks configured

#### Git Configuration
- [ ] User name configured
- [ ] Email configured
- [ ] SSH key or token configured
- [ ] Can push/pull without password

Windows Installation

```powershell
# Download Jenkins MSI from https://www.jenkins.io/download/

# Or use Chocolatey (if installed)
choco install jenkins

# Start Jenkins service
Start-Service Jenkins

# Access Jenkins
Start-Process "http://localhost:8080"
```

## Jenkins requirements

- Jenkins LTS
- Doxygen installed on agent: `apt-get install -y doxygen`
- Python 3 + pip: `apt-get install -y python3 python3-pip`
- Plugins: `git`, `workflow-aggregator`, `ws-cleanup`

#### Essential Plugins
- [ ] Pipeline (workflow-aggregator)
- [ ] Git plugin 
- [ ] GitHub plugin 
- [ ] Docker plugin 
- [ ] GitHub Branch Source
- [ ] ws-cleanup
- [ ] Timestamper

#### Credentials
- [ ] GitHub credentials added
- [ ] SSH key configured (optional)
- [ ] Personal access token added
- [ ] Can authenticate with GitHub

#### PipelineB Job
- [ ] Job created in Jenkins
- [ ] GitHub repo configured
- [ ] Branch: TaskB
- [ ] Jenkinsfile path: taskB/Jenkinsfile
- [ ] Manual build tested
- [ ] Artifacts generated
- [ ] Build time acceptable

#### PipelineC Job
- [ ] Job created in Jenkins
- [ ] GitHub repo configured
- [ ] Branch: TaskC
- [ ] Jenkinsfile path: taskC/Jenkinsfile
- [ ] Manual build tested
- [ ] Artifacts generated
- [ ] Warnings CSV generated

#### Automation
- [ ] GitHub webhooks working
- [ ] Builds trigger on push
- [ ] Build logs available
- [ ] Artifacts archived

#### Unit Tests
- [ ] All 9 tests pass
- [ ] Coverage > 95%
- [ ] Test output clear
- [ ] No warnings/errors

#### Integration Tests
- [ ] Real Doxygen output parsed
- [ ] CSV format valid
- [ ] Edge cases handled
- [ ] Performance acceptable

#### Pipeline Tests
- [ ] PipelineB runs successfully
- [ ] PipelineC runs successfully
- [ ] End-to-end workflow tested
- [ ] Artifacts validated

#### Manual Testing
- [ ] Sample files tested
- [ ] Large files tested
- [ ] Error cases handled
- [ ] Timeout/performance checked

## Quick start

See [taskD/README.md](taskD/README.md) for full answers, testing approach, and Git LFS guidance.
