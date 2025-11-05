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



jobs_runtime = []
total_workflows_run = 0

earliest = datetime.strptime('2025-10-01T00:00:00Z', '%Y-%m-%dT%H:%M:%SZ')
latest = datetime.strptime('2025-11-01T00:00:00Z', '%Y-%m-%dT%H:%M:%SZ')
should_stop = False

page = 0
while True:
    if should_stop == True:
        break
    page = page + 1
    runs = sess.get(f'{api_url}/repos/{org}/{repo}/actions/runs?per_page=100&page={page}').json()
    print(f'\n[ Processing ] ... {page} => {len(runs.get("workflow_runs"))}')
    total_workflows_run = runs.get('total_count')
    if not runs.get('workflow_runs') or len(runs.get('workflow_runs')) <= 0:
        break
    for run in runs.get('workflow_runs'):
        if not run.get('run_started_at'):
            continue
        try:
            run_started_at = datetime.strptime(run.get('run_started_at'), '%Y-%m-%dT%H:%M:%SZ')
        except Exception as err:
            continue
        #print(run.get('name'), run.get('id'), run_started_at)
        last_started_at = run_started_at
        if run_started_at < earliest:
            should_stop = True
            break
        if run_started_at >= latest:
            continue
        jobs = sess.get(f'{api_url}/repos/{org}/{repo}/actions/runs/{run.get("id")}/jobs?per_page=100').json()
        for job in jobs.get('jobs'):
            if not job.get('started_at') or not job.get('completed_at'):
                continue
            started_at = datetime.strptime(job.get('started_at'), '%Y-%m-%dT%H:%M:%SZ')
            finished_at = datetime.strptime(job.get('completed_at'), '%Y-%m-%dT%H:%M:%SZ')
            elapsed = (finished_at - started_at).total_seconds()
            if elapsed <= 0:
                continue
            name = f'{run.get("name")} : {job.get("name")}'
            data = {
                'run_id': run.get('id'),
                'job_id': job.get('id'),
                'workflow_name': run.get('name'),
                'job_name': job.get('name'),
                'started_at': job.get('started_at'),
                'finished_at': job.get('completed_at'),
                'elapsed': elapsed,
                'conclusion': job.get('conclusion'),
                'labels': ','.join(job.get('labels')),
                'runner_id': job.get('runner_id'),
                'runner_name': job.get('runner_name')
            }
            jobs_runtime.append(data)

if True:
    job_stats = {}
    for rt in jobs_runtime:
        _job_name = f'{rt.get("workflow_name")} : {rt.get("job_name")}'
        if _job_name not in job_stats:
            job_stats[_job_name] = []
        job_stats[_job_name].append(rt.get('elapsed'))
    max_job_name = '-'
    max_job_runtime = 0
    avg_job_name = '-'
    avg_job_runtime = 0
    for j_n in job_stats:
        item = job_stats[j_n]
        avg_runtime = int(sum(item)/len(item))
        max_runtime = max(item)
        if max_job_runtime < max_runtime:
            max_job_runtime = max_runtime
            max_job_name = j_n
        if avg_job_runtime < avg_runtime:
            avg_job_runtime = avg_runtime
            avg_job_name = j_n
        #print(f'{j_n} => AVG: {int(sum(item)/len(item))}, MAX: {max(item)} ')
    print(f'Total jobs from {earliest} to {latest}: {len(jobs_runtime)}')
    print(f'Number of unique runners: {len(list(set([x.get("runner_name") for x in jobs_runtime])))}')
    print(f'Highest Maximum runtime job: {max_job_name} => {max_job_runtime}')
    print(f'Highest Average runtime job: {avg_job_name} => {avg_job_runtime}')
```






