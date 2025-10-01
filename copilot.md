## Cheatsheet collection

* [Home](index.md)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](gcp.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* [Kubernetes](k8s.md)
* [Istio](istio.md)
* [OIDC](openID.md)
* [PostgreSQL](postgres.md)
* <ins>[GitHub Copilot](copilot.md)</ins>

---



## Manual for using Github copilot in VSCode

# Use the simplest first (DO NOT USE COMMENTS FOR PROMPTING):
* Ghost text
* Inline chat
* Chat Panel (for something more detailed)

# Don't recreate existing prompts
* Use [slash command](https://docs.github.com/en/copilot/using-github-copilot/copilot-chat/asking-github-copilot-questions-in-your-ide#slash-commands) for things like `/fix`, `/doc`,`/explain`,`/test/` (in the chat prompt box type `/` and check all available slash commands). Slash commands in GitHub Copilot Chat allow you to quickly specify the intent of your query

# Context is everything
* Add files to chat `#`
* Add chat participants `@` (filter things out)

# Chat
GitHub Copilot agents are custom tools that you can build and integrate with GitHub Copilot Chat to provide additional functionalities tailored to your specific needs:

@workspace: This agent allows you to extend the context of whatever questions you ask Copilot to the whole project
@terminal: This agent is useful for command-line related questions. For example, you could ask it to find the largest file in a directory or explain the last command you ran.