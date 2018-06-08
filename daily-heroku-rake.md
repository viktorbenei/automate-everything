# Daily Heorku Rake task with notifications

## Starting point

- a Rails project, with a `rake` task which collects statistics
- a Heroku project
- running daily (e.g. with cron or just manuall) a `heroku run rake stats:daily_report -a $HEROKU_APP_ID > ~/Desktop/daily_stats.log`
- this spins up a one-off heroku dyno, runs the rake task and saves the result into the `~/Desktop/daily_stats.log` stats file
- as a daily "stats summary" I usually `tail ~/Desktop/daily_stats.log` to get just the brief summary of the stats (total number of builds, % of success/failed builds, ...)

## The issue

Having to run this manually every day, or using `cron` on a specific PC/Mac.
A reminder / notification would also be great, to make sure I check these stats.


## Automating it with Bitrise!

Register a repo on [bitrise.io](https://www.bitrise.io). This can be any random open repo you have if you
don't plan to store the config in the repo. If you want to store the config in the repo then of course first
create a git repository then register that on bitrise.io

When the scanner fails (as this is not a mobile/app project) just select **Restart without scanner**,
which will skip the project type detection (scanner) and will let you define the whole config.
In the **Project build configuration** section select **Manual**, then **Other/Manual** as the project type.
For the stack the `Android & Docker, on Ubuntu` one should be perfect.

**Settings tab**: Once the project is registered I changed the title, as the title by default is infered from the repository's URL.
I also changed turned off any email notification in case of `On Successful Builds`, as I'm only interested in failed builds.

Add a **Scheduled Build**.

Added bonus: you can get the full stats log file for any previous build in Artifacts!

## Final config

```yaml
---
format_version: '5'
default_step_lib_source: https://github.com/bitrise-io/bitrise-steplib.git
project_type: other
trigger_map:
- push_branch: "*"
  workflow: primary
- pull_request_source_branch: "*"
  workflow: primary
workflows:
  primary:
    steps:
    - activate-ssh-key@4.0.1:
        run_if: '{{getenv "SSH_RSA_PRIVATE_KEY" | ne ""}}'
    - git-clone@4.0.11: {}
    - script@1.1.5:
        title: Install Heroku CLI
        inputs:
        - content: |-
            #!/usr/bin/env bash
            set -ex

            curl https://cli-assets.heroku.com/install-ubuntu.sh | sh

            heroku --version
    - script@1.1.5:
        title: Heroku run rake
        inputs:
        - content: |-
            #!/usr/bin/env bash
            set -ex

            # run the rake task and save its output into a file
            heroku run rake stats:daily_report -a "$HEROKU_APP_ID" > "$BITRISE_DEPLOY_DIR/daily_stats.log"

            # expose the summary as env var
            tail "$BITRISE_DEPLOY_DIR/daily_stats.log" | envman add --key BITRISE_DAILY_STATS_SUMMARY
    - slack@2.7.2:
        inputs:
        - text: |-
            Daily Stats:

            ```
            $BITRISE_DAILY_STATS_SUMMARY
            ```
        - webhook_url: "$SLACK_WEBHOOK_URL"
    - deploy-to-bitrise-io@1.3.12: {}
app:
  envs:
  - HEROKU_API_KEY: "$HEROKU_API_KEY"
    opts:
      description: set this in Secrets (.bitrise.secrets.yml)
  - HEROKU_APP_ID: "$HEROKU_APP_ID"
    opts:
      description: set this in Secrets (.bitrise.secrets.yml)
  - SLACK_WEBHOOK_URL: "$SLACK_WEBHOOK_URL"
    opts:
      description: set this in Secrets (.bitrise.secrets.yml)

```