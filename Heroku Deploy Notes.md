# Deploying to Heroku

1. Create a git repo
2. Create a heroku app: `heroku create appname`
3. Push to heroku repo: `git push heroku master`
4. `heroku addons:add mongohq`
5. Check if you have a `MONGOHQ_URL` when you run `heroku config`
6. Add code inside `mongoid.yml`
	
		production:
		      sessions:
		        default:
		          uri: <%= ENV['MONGOHQ_URL'] %>
		          
7. Add gem `gem 'rails_12factor'`
8. Inside `production.rb` change `config.assets.compile` to `true`.
9. Replace `config.assets.precompile += %w( search.js )` with `config.assets.precompile += %w[*.png *.jpg *.jpeg *.gif]`
10. Precompile locally: (Push to github) then run `RAILS_ENV=production bundle exec rake assets:precompile` in terminal.
11. If you have seeded data run `heroku run rake db:seed`
12. Push to Heroku
13. `heroku open` and mazeltav!