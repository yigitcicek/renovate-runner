# renovate-runner

The intention of this project is to provide a pipeline which is easy to set up and reflects the current app settings as close as possible.

You will need to:

1. Create a new private project to host the runner
2. Configure credentials using CI variables
3. Create a new `master` pipeline that includes this project's template
4. Set up a schedule to run the pipeline regularly

## Create a new runner Project

We recommend you use a dedicated private project to host the Renovate runner.
Easiest is to start with a new empty project.

## Configure CI/CD variables

At a minimum you need to add a GitLab [Personal Access Token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html#creating-a-personal-access-token) (scopes: `read_user`, `api` and `write_repository`) as `RENOVATE_TOKEN` to CI/CD variables.

It is also recommended to configure a [GitHub.com Personal Access Token](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) (minimum scopes) as `GITHUB_COM_TOKEN` so that your bot can make authenticated requests to github.com for Changelog retrieval as well as for any dependency that uses GitHub tags.
Without such a token, github.com's API will rate limit requests and make such lookups unreliable.

Finally, you need to decide how your bot should decide which projects to run against.
The default settings will run against any projects which satisfies these two characteristics:
- The bot's token has Developer or higher access rights
- The project has a Renovate configuration file already (e.g. `renovate.json`)

If you wish for your bot to run against *every* project which it has access to, including onboarding any which don't yet have a config, then add this variable: `RENOVATE_EXTRA_FLAGS="--onboarding=true"`.

If you wish to manually specify which projects that your bot runs again, then add this variable: `RENOVATE_EXTRA_FLAGS="--autodiscover=false group1/repo5 user3/repo1"` (i.e. providing a list of every repository with a space in-between).

## Create a GitLab CI file

Create a `.gitlab-ci.yml` file in the repository like the following:

```yaml
include:
  - project: 'renovate-bot/renovate-runner'
    file: '/templates/.gitlab-ci.yml'

variables:
  LOG_LEVEL: debug

renovate:on-schedule:
  only:
    - schedules
  script:
    - renovate $RENOVATE_EXTRA_FLAGS

```

## Configure the Schedule

Add a schedule (`CI / CD` > `Schedules`) to run Renovate regularly.
Best practise it to run it hourly.

The following sample run it every hour on third minute: `3 * * * *`.

## Other config options

We've changed some renovate defaults for GitLab to better reflect the App's default behavior, so please see [here](./templates/.gitlab-ci.yml#L3) for changed options.

For other self-hosted gitlab samples you can checkout [here](https://github.com/renovatebot/docker-renovate/blob/master/docs/gitlab.md).
