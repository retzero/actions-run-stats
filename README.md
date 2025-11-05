# actions-run-stats

```python
import requests
from datetime import datetime

api_url = 'https://api.github.com'
org = 'mlflow'
repo = 'mlflow'
sess = requests.session()
sess.auth = (id, token)
sess.headers.update({'Accept': 'application/vnd.github+json'})

runs = sess.get(f'{api_url}/repos/{org}/{repo}/actions/runs?per_page=100').json()

jobs_runtime = {}

total_workflows_run = runs.get('total_count')
for run in runs.get('workflow_runs'):
    workflow_name = run.get('name')
    run_id = run.get('id')
    print(run.get('name'), run_id)
    jobs = sess.get(f'{api_url}/repos/{org}/{repo}/actions/runs/{run_id}/jobs?per_page=100').json()
    for job in jobs.get('jobs'):
        job_name = job.get('name')
        if not job.get('started_at') or not job.get('completed_at'):
            continue
        started_at = datetime.strptime(job.get('started_at'), '%Y-%m-%dT%H:%M:%SZ')
        finished_at = datetime.strptime(job.get('completed_at'), '%Y-%m-%dT%H:%M:%SZ')
        elapsed = (finished_at - started_at).total_seconds()
        if elapsed <= 0:
            continue
        name = f'{workflow_name} : {job_name}'
        if name not in jobs_runtime:
            jobs_runtime[name] = []
        jobs_runtime[name].append(elapsed)

max_job_name = '-'
max_job_runtime = 0
avg_job_name = '-'
avg_job_runtime = 0

for j_n in jobs_runtime:
    item = jobs_runtime[j_n]
    avg_runtime = int(sum(item)/len(item))
    max_runtime = max(item)
    if max_job_runtime < max_runtime:
        max_job_runtime = max_runtime
        max_job_name = j_n
    if avg_job_runtime < avg_runtime:
        avg_job_runtime = avg_runtime
        avg_job_name = j_n
    print(f'{j_n} => AVG: {int(sum(item)/len(item))}, MAX: {max(item)} ')
```
