This app provides an example of how to setup a production and stagin environment for a Remix-Fly.io-GitHubAction.

The setup is as follows:
- we are having two environments
  - production
     - deployed when we push on main branch
  - stage
     - deployed everytime when the pull request is created on the main branch 
 
- Each environment has its own GitHub Action(yml file), and deploy configuration for the Fly.io(toml file)


# Prerequisite:
- You have following accounts
  - GitHub
  - Fly.io
- You know....
  - ...how to create new repository, branch in repository, create pull request
  - ...how to create an app in fly.io
 
# Steps:
- Create new remix app from the terminal using command ```npx create-remix@latest```code 
- Initialise the fly app using this command ```fly launch```
  - this command will deploy all the code to fly io app
  - use name ```my-remix-app-production``` for this deployment.
  - Once this is done, you will find one file called ```fly.toml``` created in the root directory of the app
  - rename this file to ```fly-deploy.toml``` and paste all the following code in it
    ```
    app = 'my-remix-app-production'
    primary_region = 'ams'
    
    [build]
    
    [http_service]
      internal_port = 3000
      force_https = true
      auto_stop_machines = 'stop'
      auto_start_machines = true
      min_machines_running = 0
      processes = ['app']
    
    [[vm]]
      size = 'shared-cpu-1x'
    ```
  - now create ```.github/workflows``` folder at the root of the project and then create ```fly-deploy.yml``` file in it and paste following code in it. This is a github action that will trigger deploy everythime when there is a push on the main branch
    ```
    name: Fly Deploy
    on:
      push:
        branches:
          - main
    jobs:
      deploy:
        name: Deploy app
        runs-on: ubuntu-latest
        environment:
          name: production
          url: https://my-remix-app-production.fly.dev
        concurrency: deploy-group    # optional: ensure only one action runs at a time
        steps:
          - uses: actions/checkout@v4
          - uses: superfly/flyctl-actions/setup-flyctl@master
          - run: flyctl deploy -a my-remix-app-production -c fly-deploy.toml --remote-only
            env:
              FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
    ```
  - Create Fly api token using command ```fly tokens create deploy -c fly-deploy.toml``` and keep this token somewhere, we will need this in some time
 
- At this stage we are done with ```production``` environment.
  - To be precide we have created following things
    - fly app with name ```my-remix-app-production```
    - ```fly-deploy.toml``` file
    - ```fly-deploy.yml``` file
    - fly api token for the ```fly-deploy.toml``` file
- Lets create same for the stage environemnt

- Initialise another fly app using this command ```fly launch```
  - this command will deploy all the code to fly io app
  - This time it will ask to use the existin file, you can select ```yes``` and continue
  - use name ```my-remix-app-stage``` for this deployment.
  - Once this is done, create a copy of the```fly-deploy.toml```, give name ```fly-deploy-stage.toml``` and change the app name in this file to ```my-remix-app-stage```
  - It should look like the following(only ```app``` name is changes, and rest of all is same)
    ```
    app = 'my-remix-app-stage'
    primary_region = 'ams'
    
    [build]
    
    [http_service]
      internal_port = 3000
      force_https = true
      auto_stop_machines = 'stop'
      auto_start_machines = true
      min_machines_running = 0
      processes = ['app']
    
    [[vm]]
      size = 'shared-cpu-1x'
    ```
  - now go to ```.github/workflows``` folder at the root of the project and then create ```fly-deploy-stage.yml``` file in it and paste following code in it. This is a github action that will trigger deploy everythime when there is a pull request on the main branch
    ```
    name: Fly Deploy to Stage
    on:
      pull_request:
        branches:
          - main
    jobs:
      deploy:
        name: Deploy app to Stage
        runs-on: ubuntu-latest
        environment:
          name: stage
          url: https://my-remix-app-stage.fly.dev
        concurrency: deploy-group-stage    # optional: ensure only one action runs at a time
        steps:
          - uses: actions/checkout@v4
          - uses: superfly/flyctl-actions/setup-flyctl@master
          - run: flyctl deploy -a my-remix-app-stage -c fly-deploy-stage.toml --remote-only
            env:
              FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
    ```
  - Create Fly api token using command ```fly tokens create deploy -c fly-deploy-stage.toml``` and again, keep this token somewhere, we will need this in some time
 
  - At this stage we are done with all the code related setup, next we will setup the GitHub.
    - publish this repo to GitHub
    - Go to Settings -> Secrets And Variabels -> Actions -> Manage Environments
    - Now create two environemnts
      - production
      - stage
    - Now create ```FLY_API_TOKEN``` secret in each environemnt and paste the correct key in each environment.(we have created this in the previsous steps)
   
      
  - At this stage we are done and we can try pushign somethig to main to test the production, and create a PR to main to test the staging
