# github-actions

Manual Trigger

- Generate Fine-grain token first. Read and write access to actions and workflows
- use 'export GITHUB_TOKEN="..."' in terminal to set this token
- Command to trigger workflow:
  gh workflow run manual.yml -f name="Devanshu Yadav" -f greeting=Bonjour -f age=22 -f data="$(cat mydata)"
  where mydata is a file which contains base64 encoded text of the message to be printed.
