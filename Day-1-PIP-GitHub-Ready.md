# Day 1 PIP – Completion & Evidence Document

**Technologies:** Git • Python/Boto3 • AWS CLI • Bash/Shell Scripting  
**Purpose:** Day 1 PIP task implementation, verification, commands, explanations, and screenshot evidence.

> **Screenshot instructions:** Replace every `📸 INSERT SCREENSHOT HERE` section with the relevant screenshot from the terminal, GitHub settings, AWS console, or code.

---

# TASK 1 – GIT

## 1. Branching Strategy

I followed a simple environment-based branching strategy using **feature**, **dev**, and **main** branches. Feature branches are used for individual changes, `dev` is used for development/integration, and `main` represents the production branch.

### Commands

```bash
git branch
git checkout -b feature/<feature-name>
git checkout dev
git checkout main
```

### Workflow

```text
feature/<name> → dev → main (production)
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 2. Branch Protection / Rulesets

I configured repository rules for important branches so changes cannot be merged without proper review. I configured **2 reviewer approvals** and enabled **dismiss stale approvals when new commits are pushed**, ensuring the latest changes are reviewed.

### Evidence

GitHub:

```text
Repository
  → Settings
  → Rulesets / Branch Rules
  → Required approvals: 2
  → Dismiss stale approvals when new commits are pushed
```

📸 **INSERT SCREENSHOT HERE**

---

## 3. Cherry-pick

I used `git cherry-pick` to bring one specific commit from another branch without merging the complete branch. This is useful when I need only one particular fix or change in another branch.

### Command

```bash
git cherry-pick <commit-id>
```

### Example

```bash
git log --oneline
git cherry-pick abc1234
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 4. Interactive Rebase

I used interactive rebase to clean and organize local commit history. It allows me to squash, reorder, edit, or rename commits before sharing the branch.

### Command

```bash
git rebase -i HEAD~3
```

### Example

```text
pick    commit-1
squash  commit-2
reword  commit-3
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 5. Git Bisect

I used `git bisect` to identify which commit introduced a bug or unwanted change. I mark a known commit as good and another as bad, and Git automatically narrows the search.

### Commands

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

After testing each checkout:

```bash
git bisect good
```

or:

```bash
git bisect bad
```

Finish with:

```bash
git bisect reset
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 6. Revert / Rollback

I used `git revert` to safely undo a previously committed change while preserving the existing Git history. It creates a new commit that reverses the earlier commit.

### Command

```bash
git revert <commit-id>
```

### Why Revert?

For shared branches, revert is safer than rewriting history because the original commit remains in the history.

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 7. Git Workflow Verification

I verified the Git workflow, branches, commit history, advanced Git operations, repository protection rules, and rollback process.

### Commands

```bash
git status
git branch
git log --oneline --graph --all
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

# TASK 2 – PYTHON / BOTO3

## 1. Multi-account Resource Discovery

I implemented a resource discovery tool for **EC2, RDS, and VPC** resources. The tool accepts AWS account IDs and supports cross-account access using **STS AssumeRole**; the current live demonstration was verified in my AWS account.

### Command

```bash
python3 -m scripts.resource_discovery \
  --region ap-south-1 \
  --account 153099270680 \
  --output json \
  --max-workers 5
```

### What It Does

```text
AWS Account
    ↓
STS / AWS Credentials
    ↓
EC2 + RDS + VPC discovery
    ↓
JSON / Text output
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 2. Async Boto3 / Concurrent Processing

Boto3 API calls are blocking, so I used `ThreadPoolExecutor` from `concurrent.futures` to run independent AWS operations concurrently while `asyncio` coordinates the workflow. This reduces waiting compared with processing EC2, RDS, and VPC sequentially.

### Verification

```bash
grep -n "asyncio\|concurrent.futures\|ThreadPoolExecutor" \
scripts/resource_discovery.py
```

### Concept

```text
Sequential:
EC2 → wait → RDS → wait → VPC

Concurrent:
EC2 ─┐
RDS ─┼→ run concurrently → result
VPC ─┘
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 3. Structured JSON Logging

I implemented structured JSON logging with fields such as **timestamp, severity, correlation ID, and message**. Structured logs are easier to search, troubleshoot, monitor, and send to centralized logging systems.

### Example

```json
{
  "timestamp": "...",
  "severity": "INFO",
  "correlation_id": "...",
  "message": "EC2 inventory completed"
}
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 4. Five Production-grade Scripts

I implemented five operational scripts:

1. EC2 Inventory
2. Cost Analyzer
3. Security Group Auditor
4. AMI Cleanup
5. Snapshot Manager

The scripts include CLI arguments, AWS API calls, error handling, and safe execution controls.

### Verification

```bash
python3 -m scripts.ec2_inventory --help
python3 -m scripts.cost_analysis --help
python3 -m scripts.security_group_audit --help
python3 -m scripts.ami_cleanup --help
python3 -m scripts.snapshot_manager --help
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 5. Retry Logic – Exponential Backoff + Jitter

I implemented retry logic for temporary AWS/API failures such as throttling. **Exponential backoff** increases the waiting time after each failure, while **jitter** adds a small random delay.

### Example

```text
Attempt 1 → wait about 2 seconds
Attempt 2 → wait about 4 seconds
Attempt 3 → wait about 8 seconds
Attempt 4 → wait about 16 seconds
```

Jitter adds a random amount to these delays so multiple workers do not retry at exactly the same time.

### Why I Used It

AWS APIs can temporarily throttle requests or experience transient failures. Retry with backoff gives the service time to recover, while jitter helps prevent a large number of clients from retrying together.

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 6. AWS SDK Error Handling

I handled AWS SDK errors using `ClientError` for AWS service API failures and `BotoCoreError` for lower-level SDK/network-related failures. Retryable errors such as throttling are treated differently from permanent errors such as `AccessDenied`.

### Verification

```bash
grep -n "ClientError\|BotoCoreError\|Throttl\|AccessDenied" \
common/*.py scripts/*.py
```

### Important

I do not use a made-up generic `ServiceError` class. In normal Boto3/botocore handling, service API failures are represented by `ClientError`, while lower-level botocore failures use `BotoCoreError`.

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 7. CLI Arguments, Help & Code Quality

I used `argparse` for command-line argument parsing and added help documentation to the scripts. I also verified the script help and achieved a **Pylint score of 9.73/10**, exceeding the required score of 8.0.

### Verification

```bash
python3 -m scripts.ec2_inventory --help
```

### Help Verification for Scripts

```bash
for f in scripts/*.py; do
    echo "===== $f ====="
    python3 -m "$(echo "$f" | sed 's|/|.|g; s|\.py$||')" --help >/dev/null \
      && echo "HELP OK"
done
```

### Pylint

```text
Pylint Score: 9.73/10
Required: > 8.0
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

# TASK 2 – SAFETY / DRY-RUN EVIDENCE

## AMI Cleanup – Dry Run

AMI cleanup was tested in dry-run mode. The command showed what would be deleted without actually deregistering the AMI.

### Command

```bash
python3 -m scripts.ami_cleanup \
  --region ap-south-1 \
  --older-than 0 \
  --dry-run
```

### Expected Behavior

```text
DRY-RUN: would deregister ami-xxxxxxxx
```

### Why Dry-run?

Dry-run provides a safe way to validate destructive operations before executing them.

📸 **INSERT AMI DRY-RUN SCREENSHOT HERE**

---

## Snapshot Manager

Snapshot Manager protects deletion behind the `--execute` option. Without `--execute`, the script does not perform deletion.

### Help

```bash
python3 -m scripts.snapshot_manager \
  --region ap-south-1 \
  --help
```

📸 **INSERT SNAPSHOT MANAGER SCREENSHOT HERE**

---

# TASK 3 – AWS CLI / SHELL SCRIPTING

## 1. 10+ AWS CLI Scripts

I created a Bash/AWS CLI automation library covering:

- EC2 inventory
- EC2 filtering by tags
- EC2 bulk tagging
- EC2 bulk restart
- Security group changes
- EC2 termination
- VPC analysis
- IAM policy simulation
- RDS multi-region discovery
- S3 bucket policy auditing
- EBS volume snapshots
- Tag enforcement
- Parallel EC2 processing
- AWS retry wrapper
- CloudWatch Logs Insights query examples

### Verification

```bash
ls -1 scripts/
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 2. JMESPath

I used JMESPath through AWS CLI `--query` to filter and extract specific information from AWS JSON responses. This avoids writing additional parsing code for simple filtering and reporting.

### EC2 Tag Filtering Example

```bash
aws ec2 describe-instances \
  --region ap-south-1 \
  --query 'Reservations[].Instances[].{
    ID:InstanceId,
    Type:InstanceType,
    State:State.Name,
    Tags:Tags
  }' \
  --output table
```

### RDS Across Regions

The RDS script queries multiple AWS regions and displays the available RDS resources.

### S3 Policy Audit

The S3 script checks bucket policies and handles cases where a bucket has no policy.

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 3. Bash Script Library

I created reusable Bash scripts for common operational tasks such as instance termination, EBS snapshots, and tag enforcement. Destructive operations are protected with safe execution controls.

### Scripts

```text
ec2_terminate.sh
volume_snapshot.sh
tag_enforcement.sh
```

### Example

```bash
./ec2_terminate.sh --help
./volume_snapshot.sh --help
./tag_enforcement.sh --help
```

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 4. Parallel Processing

I implemented parallel EC2 processing using Bash background jobs and `wait -n`, with a configurable maximum concurrency. This allows many resources to be processed faster while limiting simultaneous AWS API calls.

### Command

```bash
./parallel_ec2.sh
```

### How It Works

```text
Resource 1 ─┐
Resource 2 ─┤
Resource 3 ─┼→ parallel jobs → results
Resource 4 ─┘
        ↓
  concurrency limit
```

### Why Use a Limit?

Processing hundreds of resources at once can create excessive API requests. A maximum concurrency value helps balance speed and AWS API stability.

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 5. CloudWatch Logs Insights Queries

I prepared CloudWatch Logs Insights queries for:

- Error-rate analysis
- Latency percentiles
- Cost tracking

### Error-rate Example

```text
fields @timestamp, @message
| filter @message like /ERROR|Error|error/
| stats count() as errors by bin(5m)
| sort @timestamp desc
```

### Latency Percentiles

```text
fields @timestamp, latency_ms
| filter ispresent(latency_ms)
| stats avg(latency_ms) as avg_latency,
        pct(latency_ms, 50) as p50,
        pct(latency_ms, 95) as p95,
        pct(latency_ms, 99) as p99
  by bin(5m)
```

### Cost Tracking

```text
fields @timestamp, account_id, service, cost
| filter ispresent(cost)
| stats sum(cost) as total_cost by account_id, service
| sort total_cost desc
```

### Why p95/p99?

Average latency can hide slow requests. p95 and p99 show the latency experienced by the slower portion of requests and are useful for performance monitoring.

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 6. Wrapper – Error Handling & Retry

I created an AWS CLI wrapper that retries failed commands up to a configurable maximum. It uses exponential backoff and jitter to handle temporary failures more safely.

### Command

```bash
./aws_retry_wrapper.sh sts get-caller-identity
```

### Failure Test

```bash
./aws_retry_wrapper.sh ec2 invalid-command --region ap-south-1
```

### Retry Behavior

```text
Attempt 1/5
     ↓ failure
backoff + jitter
     ↓
Attempt 2/5
     ↓ failure
backoff + jitter
     ↓
Attempt 3/5
...
     ↓
Attempt 5/5
     ↓
final failure
```

### Configuration

```bash
MAX_ATTEMPTS=5
BASE_DELAY=2
```

### Why Jitter?

Jitter adds randomness to the retry delay. If many workers fail at the same time, they do not all retry at exactly the same moment, reducing synchronized retry spikes.

### Evidence

📸 **INSERT SCREENSHOT HERE**

---

## 7. Documentation / GitHub Wiki

I created documentation for the Task 3 scripts, JMESPath queries, CloudWatch Logs Insights queries, examples, use cases, and task mapping.

### Documentation Files

```text
README.md
docs/task3_mapping.md
jmespath/queries.md
cloudwatch/logs_insights_queries.md
```

### GitHub Wiki Structure

Suggested Wiki pages:

```text
Home
Task 3 – Script Overview
EC2 Automation
JMESPath Queries
RDS Multi-region
S3 Policy Audit
Parallel Processing
Retry Wrapper
CloudWatch Logs Insights
Examples and Use Cases
```

### Evidence

📸 **INSERT GITHUB WIKI SCREENSHOT HERE**

---

# TASK 3 – TEST EVIDENCE

## EC2 Bulk Tagging

```bash
./ec2_bulk_tag.sh --execute
```

**Result:** EC2 instances were successfully tagged.

📸 **INSERT SCREENSHOT HERE**

---

## EC2 Tag Filtering

```bash
./ec2_filter_by_tag.sh
```

📸 **INSERT SCREENSHOT HERE**

---

## RDS Multi-region

```bash
./rds_multi_region.sh
```

**Verified regions:**

```text
ap-south-1
us-east-1
```

📸 **INSERT SCREENSHOT HERE**

---

## S3 Policy Audit

```bash
./s3_policy_audit.sh
```

The script handled the case where the bucket did not have a bucket policy.

📸 **INSERT SCREENSHOT HERE**

---

## Parallel EC2

```bash
./parallel_ec2.sh
```

📸 **INSERT SCREENSHOT HERE**

---

## CloudWatch Query Examples

```bash
./cloudwatch_query_examples.sh
```

📸 **INSERT SCREENSHOT HERE**

---

## AWS Retry Wrapper

```bash
./aws_retry_wrapper.sh sts get-caller-identity
```

Failure/retry test:

```bash
./aws_retry_wrapper.sh ec2 invalid-command --region ap-south-1
```

📸 **INSERT SCREENSHOT HERE**

---

# SHORT INTERVIEW ANSWERS

## Why did you use cherry-pick?

I used cherry-pick to move one specific commit from one branch to another without merging the complete branch. It is useful when only one particular fix is required.

## Why did you use rebase?

I used rebase to clean and organize local commit history before sharing changes. It allows me to squash, reorder, edit, or rename commits.

## Why did you use bisect?

I used bisect to identify which commit introduced a bug. Git tests the history between a known good and bad commit and narrows the search.

## Why did you use revert?

I used revert to safely undo a shared change while preserving Git history. It creates a new commit that reverses the earlier change.

## Why use asyncio and ThreadPoolExecutor?

Boto3 is blocking, so I used ThreadPoolExecutor to run independent AWS calls concurrently while asyncio coordinates the workflow. This reduces total waiting time.

## Why use exponential backoff?

Exponential backoff gives a temporary AWS/API failure time to recover by increasing the wait between retries, for example 2, 4, 8, and 16 seconds.

## Why use jitter?

Jitter adds a small random delay to retry timing. This prevents multiple workers from retrying at exactly the same time and reduces retry spikes.

## Why use dry-run?

Dry-run shows what a destructive operation would do without actually changing or deleting resources. It allows the operation to be reviewed safely first.

## Why use JMESPath?

JMESPath allows AWS CLI to filter and extract required fields directly from JSON responses. This reduces the need for additional parsing code.

## Why use parallel processing?

Parallel processing allows independent resources to be handled concurrently, reducing execution time while a concurrency limit controls API pressure.

## What is the purpose of structured logging?

Structured JSON logs provide consistent fields such as timestamp, severity, correlation ID, and message. This makes logs easier to search and troubleshoot.

## What is the difference between backoff and jitter?

Backoff controls how the retry delay increases after each failure. Jitter adds randomness to that delay so clients do not retry simultaneously.

## Why not retry AccessDenied?

AccessDenied is normally a permanent authorization problem, so retrying it repeatedly does not solve the issue. Retry logic should focus on temporary failures such as throttling or transient network/service errors.

---

# FINAL DAY 1 CHECKLIST

## Task 1 – Git

- [x] Branching strategy
- [x] Branch protection / rulesets
- [x] Two reviewer approvals
- [x] Dismiss stale reviews on new push
- [x] Cherry-pick
- [x] Interactive rebase
- [x] Git bisect
- [x] Revert / rollback
- [ ] Attach screenshots

## Task 2 – Python / Boto3

- [x] Multi-account EC2/RDS/VPC discovery implementation
- [x] Asyncio + ThreadPoolExecutor
- [x] Structured JSON logging
- [x] Five production-grade scripts
- [x] Retry logic
- [x] Exponential backoff
- [x] Jitter
- [x] AWS SDK error handling
- [x] argparse / help
- [x] Pylint 9.73/10
- [x] Dry-run verification
- [ ] Attach screenshots
- [ ] Final pytest verification if required
- [ ] Push/document in GitHub if required

## Task 3 – AWS CLI / Shell

- [x] 10+ AWS CLI/Bash scripts
- [x] JMESPath queries
- [x] EC2 tag filtering
- [x] RDS multi-region query
- [x] S3 policy audit
- [x] Bash automation library
- [x] Parallel processing
- [x] CloudWatch Logs Insights queries
- [x] Error handling and retry wrapper
- [ ] Publish/verify GitHub Wiki documentation
- [ ] Attach screenshots

---

# DAY 1 SUMMARY

## Git

Implemented a practical Git workflow with feature → dev → main branching, protected branches, two-reviewer approval requirements, stale-review dismissal, cherry-pick, rebase, bisect, revert, and rollback testing.

## Python / Boto3

Built AWS automation for EC2, RDS, and VPC discovery along with production-oriented scripts, concurrent processing, structured JSON logging, retry logic with exponential backoff and jitter, AWS SDK error handling, CLI help, dry-run safety, and code-quality verification.

## AWS CLI / Shell

Built an AWS CLI/Bash automation library covering EC2 operations, tagging, security groups, VPC analysis, IAM policy simulation, RDS multi-region queries, S3 policy auditing, snapshots, tag enforcement, parallel processing, retry wrappers, and CloudWatch Logs Insights queries.

---

# MENTOR NOTES / FEEDBACK

```text
______________________________________________________________________

______________________________________________________________________

______________________________________________________________________

______________________________________________________________________

______________________________________________________________________
```
