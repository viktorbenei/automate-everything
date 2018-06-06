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
