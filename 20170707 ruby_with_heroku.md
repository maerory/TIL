## 20170707 Today I learned

### Using Heroku - the basics (Getting started with heroku ruby ver.)

1. Install Heroku at heroku website and log in on your terminal with `heroku login`
2. At your project directory where git is already initiated `heroku create` to prepare Heroku to recieve the project, by `git push heroku master` you upload the project to heroku server (`heroku open ` allows for easy access to the web application)
3. `heroku logs --tail` : goes into log mode and streams all the incoming request to the project
4. `Procfile` : All heroku projects have procfile which contains commands for starting the app
5. `heroic ps` : check the number of currently running dynos. Dynos - light weight container that runs the commands in profile. `heroku ps:scale web=<num of dynos>` to specify how many dynos to run
6. You can also use local server to run the app!

   - Install all the dependencies in the gemfile
   - `bundle exec rake db:create db:migrate` for creating database and migrating
   - `heroku local web` to run it locally
7. Update the project with normal github commands:
   - `git push heroku master`: rather than pushing to origin, you push to heroku
8. Run console/command remotely - `heroku run rails console` for rails console, `heroku run bash` to run bash commands 
9. To set configuration variables for heroku - `heroku config:set <VAR>=<VALUE>`, and check the config vars with `heroku config` (config also lists which url your project's apps are using to connect)
10. `heroku addons` : check heroku addon to your project, in the ruby heroku, there is a free-tier postgre SQL installed for your project
11. `heroku pg` : heroku postgre info, `heroku pg:sql` to enter sql command terminal
12. `heroku run rake db:migrate` : migrate your heroku project's db before you access pages that uses the DB