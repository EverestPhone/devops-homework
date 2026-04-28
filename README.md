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
| [grpc (fork)](https://github.com/YOUR_USERNAME/grpc) | RepoA — forked C++ source for Doxygen |
| [doxygen-log-parser](https://github.com/YOUR_USERNAME/doxygen-log-parser) | RepoC — Python warnings parser |

## Jenkins requirements

- Jenkins LTS
- Doxygen installed on agent: `apt-get install -y doxygen`
- Python 3 + pip: `apt-get install -y python3 python3-pip`
- Plugins: `git`, `workflow-aggregator`, `ws-cleanup`

## Quick start

See [taskD/README.md](taskD/README.md) for full answers, testing approach, and Git LFS guidance.
