jira-task-create-with-attachment-python-script.md
---

```py
import http.client
import json
import os
import base64
import mimetypes

def create_jira_issue(project_key, summary, description, issue_type):
    """Create a Jira issue and return the issue key."""
    issue_data = {
        "fields": {
            "project": {
                "key": project_key
            },
            "summary": summary,
            "description": description,
            "issuetype": {
                "name": issue_type
            }
        }
    }

    # Make sure to split the base URL correctly
    conn = http.client.HTTPSConnection(jira_base_url)
    headers = {
        'Content-Type': 'application/json',
        'Authorization': 'Basic ' + base64.b64encode(f"{username}:{api_token}".encode()).decode()
    }

    conn.request("POST", "/rest/api/2/issue", json.dumps(issue_data), headers)
    response = conn.getresponse()
    data = response.read()

    if response.status == 201:
        issue_key = json.loads(data)['key']
        print(f"Issue created: {issue_key}")
        return issue_key
    else:
        print(f"Failed to create issue: {response.status}, {data.decode()}")
        return None

def attach_file_to_issue(issue_key, file_path):
    """Attach a file to the specified Jira issue."""
    boundary = '-------boundary'
    file_name = os.path.basename(file_path)
    file_type = mimetypes.guess_type(file_path)[0] or 'application/octet-stream'

    with open(file_path, 'rb') as file:
        file_data = file.read()

    body = (
        f'--{boundary}\r\n'
        f'Content-Disposition: form-data; name="file"; filename="{file_name}"\r\n'
        f'Content-Type: {file_type}\r\n\r\n'
    ).encode() + file_data + f'\r\n--{boundary}--\r\n'.encode()

    headers = {
        "X-Atlassian-Token": "no-check",
        "Content-Type": f'multipart/form-data; boundary={boundary}',
        'Authorization': 'Basic ' + base64.b64encode(f"{username}:{api_token}".encode()).decode()
    }

    conn = http.client.HTTPSConnection(jira_base_url)
    conn.request("POST", f"/rest/api/2/issue/{issue_key}/attachments", body, headers)
    response = conn.getresponse()
    data = response.read()

    if response.status == 200:
        print("File attached successfully.")
    else:
        print(f"Failed to attach file: {response.status}, {data.decode()}")

# Usage example
summary = "JQx users list"
description = "This task finds the region details and provides the appropriate region roles and details."
issue_type = "Task"
filename = "jqx.json"
project_key = os.getenv('JIRA_PROJECT_NAME')
jira_base_url = os.getenv('JIRA_BASE_URL')
username = os.getenv('USERNAME')
api_token = os.getenv('API_TOKEN')

# Create the issue
issue_key = create_jira_issue(project_key, summary, description, issue_type)

# If issue is created successfully, attach the file
if issue_key:
    attach_file_to_issue(issue_key, filename)
```
