
# Up and Running: ZEIT Now Environments
[title-image](https://raw.githubusercontent.com/mthomps4/posts/master/posts/up_and_running/images/dominik-schroder-FIKD9t5_5zQ-unsplash.jpg)

Photo by [Dominik Schröder](https://unsplash.com/@wirhabenzeit?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

If you’re a JavaScript developer you’ve probably heard mention of [zeit.co / Now](https://zeit.co/) and all their plans to have a zero-config setup for app deployments. One of their main goals is to provide a seamless solution to deploy your JavaScript apps such as a Next.js app without all the DevOps hassles and shell scripts. While that’s not all they do, it’s what I’ll be expanding on here. Short, simply install Now and run the command `now` from the root of your project and this will push and deploy a version of your app to Zeit's hosting site. Integrating further with GitHub, and Now will also deploy a live branch of your site based on commits and PRs of your feature branch for review.

Out of the box, I'll have to say this was nice. For quick apps and sites, this is pretty neat and works well. My issue presented itself when I wanted to scale this into multiple environments for team collaboration. How do I set up different environments? How do I manage the environment variables? While the live test branches are great, they still pointed to production. That was a no go for me. There had to be a way to leverage Now's tools to set up a proper development, staging, and production environment. After some digging around and various threads through Spectrum, I found a few solutions to try out.

> Note: I’ve been keeping watch over these threads in ZEIT’s Spectrum chat. While this is an example of my current setup, I am aware they are still actively adding features and sifting through requests. Should this setup change I’ll create a follow-up post.

**Solution 1.** Create separate instances within the ZEIT dashboard and use the CLI tools to deploy to the correct instance.

**Solution 2.** Use the same instance in ZEIT and have specific `now.json` files that would alias a specific domain URL when deploying manually through CLI or automagically through CI/CD.

While learning, I had a repo ever so cleverly named `next-now-test` that I'll be referencing throughout the rest of this article. (I was tired, names are hard) This is a bare-bones NextJS app that uses Next's serverless API. If you haven't used NextJs before no worries you want to need to worry about that for this setup, but if you are interested you can dive into their [learning section](https://nextjs.org/learn/basics/getting-started) here or stay tuned for my next entry "Up and Running: NextJS and TypeORM"

#

# **The Code**

At first, looking at the two solutions, Solution 2 seemed like a better option. Maybe it was the 4-month-old sleep I was getting… Either solution seems viable at this point as you would manage multiple instances inside of Heroku or other providers as well. Regardless, at the time I didn’t want to manage multiple instances inside of Zeit. My primary goal was to set this up to automagically-deploy through CI/CD and have aliased URLs.

For solution 2 to work at the root of my project I would need three now.json files to support Development, Staging, and Production. I would also need to set some ENV’s with Now’s CLI tool.

- now.json (the default JSON file used for branch deploys — Development)
- staging.now.json (Staging ENV’s)
- prod.now.json (Prod ENV’s)

To keep thing’s simple I created one `APP_ENV` to display the environment name on the main landing page. Below is an example of each JSON that includes the alias URL as well (more on that in a sec).

    now.json (Branch deploys no alias needed){  "name": "next-now-test",  "version": 2,  "env": {    "APP_ENV": "@dev_app_env", // set with now secrets see below   }}staging.now.json{  "name": "next-now-test",  "version": 2,  "alias": ["staging-next-now-test.now.sh"],  "env": {    "APP_ENV": "@staging_app_env",  }}prod.now.json {  "name": "next-now-test",  "version": 2,  "alias": ["production-next-now-test.now.sh"],  "env": {    "APP_ENV": "@prod_app_env",  }}

`Now` manages their environment variables through their CLI tooling. These are provided out of the box when you install Now to your machine. If you've interacted with Heroku or Aptible's CLI this is very similar. Now CLI gives you the CRUD commands needed to review, add, remove 'secrets'. As you would expect these are per project instance. For this app, I added three ENV's with `now secrets add ...`:

- dev_app_env = “DEVELOPMENT”
- staging_app_env = “STAGING”
- prod_app_env = “PRODUCTION”

You’ll note in the now.json examples above we are able to reference these with the `@` symbol. `"APP_ENV": "@staging_app_env"` If the @ symbol wasn't present `Now` would just assume this is a string value and move on. We also are making these semantic with `dev_` `staging_` `prod_`. If we were to need real ENVs, say a database_url we would want to do the same.

# **CI / CD**

Alright, that’s a lot of words for nothing so far but we are almost there. Let’s get this wired up.

A recap of our solution goals here, when I create a PR from any branch Now will create a named branch deploy using our Development ENV’s. If that PR is merged into a branch called `staging` we would like Now to deploy and alias `[staging-now-next-test.now.sh](<http://staging-now-next-test.now.sh>)` using our staging ENV's and likewise with anything going into master for production.

Feature Branch (development) → PR into Staging (staging) → PR into master (production)

I decided to use GitLab here but you can use any CI/CD tool you’d like as we will just be running a few commands based on branch names. If you recall integrating Now with GitHub already deploys a named branch when a PR is made. When deploying Now will look for the root `now.json` file. We've left our `now.json` file setup for Development so our setup here is already complete! You should see a note from Now inside the PR as well as inside [Zeit.co](http://zeit.co/) under the project itself. Below is an example of both the PR named branch and a commit specific branch deployed by Now. During development, I was using a branch called `testing`.

All we need is to trigger specific deploys for our staging and production environments. If you see below in our config we have two commands setup if something is merged into staging or master, `yarn ci:deploy:staging` and `:master` respectively.

*.gitlab-ci.yml*

    ... deploy-staging:  stage: deploy  script:    - yarn ci:deploy:staging  only:    - staging    - merge_requestsdeploy-production:  stage: deploy  script:    - yarn ci:deploy:prod  only:    - master    - merge_requests

*package.json scripts*

    ... "ci:deploy:staging": "now -A staging.now.json --target=staging --token=$NOW_TOKEN --scope=mthomps4  && now alias -A staging.now.json  --token=$NOW_TOKEN","ci:deploy:prod": "now -A prod.now.json --target=production --token=$NOW_TOKEN --scope=mthomps4 && now alias -A prod.now.json  --token=$NOW_TOKEN"

Let’s look at one of these in detail, ignoring NOW_TOKEN all of these commands can be run using the CLI.

> now -A staging.now.json — target=staging — scope=mthomps4 — token=$NOW_TOKEN

&& now alias -A staging.now.json — token=$NOW_TOKEN”

- `now -A staging.now.json` - now means 'hey deploy this' the -A command is saying 'hey deploy this and use this now.json file'.
- `—target=staging` Honestly, I'm not sure this is needed anymore. This is the counter to `now —prod`. All we are doing here is telling Zeit/Now to not deploy this to their given production URL and our aliased production URL.
- `—scope=mthomps4` Think organizations here. If you are using Now personally and are not roped into any other company/organization accounts you can skip this line. When you are a part of another Zeit organization/team you would navigate these in the CLI using `now switch` before running any other now commands. This tells `Now` to do the same and ensure you're under the right profile before it deployed.
- `&& now alias -A staging.now.json` Using the file provided with -A please alias all of our domains provided to THIS deploy. `now alias` will look into the now.json file provided for an alias array and map those domains to the recent deploy. `"alias": ["staging-next-now-test.now.sh"],`
- `—token=$NOW_TOKEN` $NOW_TOKEN is an auth token provided by Zeit found under `[https://zeit.co/account/tokens](https://zeit.co/account/tokens)` You'll need to create a token, and then save this as an ENV for GitLab or other CI to use.

**That’s it!**

With these two yarn commands, GitLab CI/CD will deploy and alias our domains as expected. Merging a branch into staging will update `[staging-next-now-test.now.sh](<http://staging-next-now-test.now.sh>)` and merging to master will update `production-next-now-test.now.sh`.

![https://miro.medium.com/max/30/1*_W-g11bFKTwQmrfx4w9AGw.png?q=20](https://miro.medium.com/max/30/1*_W-g11bFKTwQmrfx4w9AGw.png?q=20)

![https://miro.medium.com/max/1235/1*_W-g11bFKTwQmrfx4w9AGw.png](https://miro.medium.com/max/1235/1*_W-g11bFKTwQmrfx4w9AGw.png)

