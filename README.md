# Gitlab CI Templates
This repository contains templates than can be used with [Gitlab's include](https://docs.gitlab.com/ee/ci/yaml/includes.html) keyword. A description of the templates is available below.

## build-release-deploy.yml
This contains jobs for executing the standard CI/CD process for projects within the HCA. 

### Stages:
1. Build (builds the image and pushes to quay.io using the short commit hash. e.g. ebi-ait/ingest-core:abc123
2. Release dev (only on dev branch. Pulls the image and releases using the latest-dev tag or one defined with `$DEV_RELEASE_IMAGE`)
3. Release master (only on master branch. Pulls the image and releases using the latest tag or one defined with `$MASTER_RELEASE_IMAGE`)
4. Deploy dev (only on dev branch. Deploys to the dev cluster)
5. Integration dev (runs integration tests on dev environment using [ingest-integration-tests](https://gitlab.ebi.ac.uk/hca/ingest-integration-tests/))
6. Deploy staging (only on master branch. Deploys to the staging cluster)
7. Integration staging (runs integration tests on staging environment using [ingest-integration-tests](https://gitlab.ebi.ac.uk/hca/ingest-integration-tests/))
8. Deploy prod (only on master branch and when manually triggered. Deploys to the prod cluster)
9. Integration prod (runs integration tests on prod environment using [ingest-integration-tests](https://gitlab.ebi.ac.uk/hca/ingest-integration-tests/))
10. Merge dev to master - merges dev branch to master branch

### Dev flow
1. Create a feature branch and commit changes to it
    - naming convention: `feature/dcp-xyz-title` or `bugfix/dcp-xyz-title` where xyz is the ticket number
2. PR feature branch -> dev
    - naming convention: `dcp-xyz title` where xyz is the ticket number
    - put in the description the dcp-xyz and ebi-ait/dcp-ingest-central#xyz automated link keywords so that zenhub and github link the pr to the ticket
3. Build (and unit tests if specified in project including this template) will be run automatically
4. PR approved and merged to dev
    - get reviews from the team
    - address the review
    - PR checks run successfully on gitlab
5. Deploy to dev environment and run integration tests
    - this happens automatically after merging the PR to dev
6. Merge dev -> master
7. Deploy to staging environment and run integration tests
8. Manual wrangler testing on staging environment
9. Wrangler approval
10. Manually trigger a deploy to prod environment
11. Integration tests on prod are automatically run

### Variables
These can be overriden in the project `.gitlab-ci.yaml` that includes this template.

- `APP_NAME`
  - By default is [`CI_PROJECT_NAME`](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)
- `DEV_REPLICAS`
  - Default is `1`
- `STAGING_REPLICAS`
  - Default is `1`
- `PROD_REPLICAS`
  - Default is `1`
- `IMAGE_NAME`
  - Default is e.g. `quay.io/ebi-ait/$CI_PROJECT_NAME:$CI_COMMIT_SHORT_SHA` so e.g. `quay.io/ebi-ait/ingest-core:abc123
- `DEV_RELEASE_IMAGE`
  - Default is `quay.io/ebi-ait/$CI_PROJECT_NAME:latest-dev`
- `MASTER_RELEASE_IMAG`
  - Default is `quay.io/ebi-ait/$CI_PROJECT_NAME:latest`

You will also need to ensure `QUAY_USERNAME` and `QUAY_PASSWORD` are set in the group CI/CD variables in Gitlab UI
