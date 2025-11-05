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

earliest = datetime.strptime('2025-11-5T13:25:00Z', '%Y-%m-%dT%H:%M:%SZ')
latest = datetime.strptime('2025-11-05T14:40:00Z', '%Y-%m-%dT%H:%M:%SZ')
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
        workflow_name = run.get('name')
        run_id = run.get('id')
        if not run.get('run_started_at'):
            continue
        try:
            run_started_at = datetime.strptime(run.get('run_started_at'), '%Y-%m-%dT%H:%M:%SZ')
        except Exception as err:
            continue
        #print(run.get('name'), run_id, run_started_at)
        last_started_at = run_started_at
        if run_started_at < earliest:
            should_stop = True
            break
        if run_started_at >= latest:
            continue
        jobs = sess.get(f'{api_url}/repos/{org}/{repo}/actions/runs/{run_id}/jobs?per_page=100').json()
        for job in jobs.get('jobs'):
            job_name = job.get('name')
            job_id = job.get('id')
            if not job.get('started_at') or not job.get('completed_at'):
                continue
            started_at = datetime.strptime(job.get('started_at'), '%Y-%m-%dT%H:%M:%SZ')
            finished_at = datetime.strptime(job.get('completed_at'), '%Y-%m-%dT%H:%M:%SZ')
            elapsed = (finished_at - started_at).total_seconds()
            if elapsed <= 0:
                continue
            name = f'{workflow_name} : {job_name}'
            data = {
                'run_id': run_id,
                'job_id': job_id,
                'workflow_name': workflow_name,
                'job_name': job_name,
                'started_at': started_at,
                'finished_at': finished_at,
                'elapsed': elapsed,
                'conclusion': job.get('conclusion'),
                'labels': str(job.get('labels')),
                'runner_id': job.get('runner_id'),
                'runner_name': job.get('runner_name')
            }
            jobs_runtime.append(data)
```
