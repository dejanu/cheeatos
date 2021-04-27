## Cheatsheet collection



### GCP

**List/Set gcp project**

```bash
gcloud projects create <PROJECT_ID> --organization=<ORGANIZATION_ID>
gcloud config list project
gcloud config set project <my-project>
```
**Get project ID**
```bash
# export project id as environment variable
export PROJECT_ID=$(gcloud config get-value core/project)
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export PROOJECT_ID=$(gcloud config list --format 'value(core.project)')
```
***
