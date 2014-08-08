# User Authentication

This will help you build a simple user authentication system using `bcrypt`. 

## Setting Up

1. Unleash `gem 'bcrypt', '~> 3.1.7'` inside your Gemfile
2. Create a User model with `rails g model User [your properties]`
3. Inside the User model `user.rb`: 
	
		  include ActiveModel::SecurePassword
		  field :email, type: String
		  field :password_digest, type: String
		
		  has_secure_password
		  
	
4. Inside the `users_controller.rb`: 

		 # Prepare to show the sign-up form
		 def new
		 	@user = User.new
		 	@is_signup = true
		 end
		 # Actually build the user
		 def create
		 	user = User.new(params.require(:user).permit(:name, :email, :password, :password_confirmation))
		 	if user.save
		 		redirect_to users_path
		 	end
		 end
		 ...
		 def destroy
			 User.find(params[:id]).destroy
		     # Exactly the same idea as this little number:
		    redirect_to users_path
		 end
		 
5. Inside `routes.rb` add `resource :session, only: [:new, :create, :destroy]`
6. If not created already, create a sessions controller. Add inside add `sessions_controller.rb`:

		class SessionsController < ApplicationController
			def new
				@user = User.new
				@is_login = true
			end
		
			def create
				u = User.where(email: params[:user][:email]).first
				# First make sure we actually found a user
				# Then, see if their password matches
				if u && u.authenticate(params[:user][:password])
					# sets a  cookie at the user that holds the logging in user ID
					session[:user_id] = u.id.to_s
					redirect_to users_path
				else
					# if wrong, then reloads to try logging in
					redirect_to new_session_path
				end
			end
		
			def destroy
				# Kill all of our cookies, log out
				reset_session
				redirect_to new_session_path
			end
		end
		
7. Inside the `application_controller.rb`: 

		  helper_method :current_user
		
		  def current_user
		  	@current_user ||= User.find(session[:user_id]) if session[:user_id]
		  end
		  
	The method helper_method is to explicitly share some methods defined in the controller to make them available for the view. This is used for any method that you need to access from both controllers and helpers/views (standard helper methods are not available in controllers). This is a common use case!	  
		  
## Adding User Authentication in the View
At this point it's up to you how you want the user to login, signup, etc. Here's how I setup my rudimentary views with user auth.

1. Show whether the user is logged in: <small>`application.html.erb`</small> 

		<div class="pull-right">
				<% if current_user %>
					Welcome <%= current_user.name %>!
					<%= link_to "Log out", session_path, method: :delete %>
				<% else %>
					<%= link_to "Sign up", new_user_path %>
					<%= link_to "Log In", new_session_path %>
				<% end %>
		</div>
		
2. Create a signup form: <small>(<i>views > users > </i>`new.html.erb`)</small>

		<h1>Tell Us About Yourself</h1>

		<%= form_for @user do |f| %>
			Name: <%= f.text_field :name, placeholder: "name" %><br />
			Email: <%= f.text_field :email, placeholder: "@" %><br />
			Password: <%= f.password_field :password, placeholder: "password" %><br />
			Confirm Password: <%= f.password_field :password_confirmation, placeholder: "password" %><br />
			<%= f.submit %>
		<% end %>


3. Create a login form: If there isn't a sessions folder in the views, make one. Inside the sessions folder, create `new.html.erb`: <small>(<i>views > sessions ></i>`new.html.erb`)<small>

		<h1>Login</h1>
		
		<%= form_for @user, { url: session_path } do |f| %>
			Email: <%= f.text_field :email, placeholder: "@" %><br />
			Password: <%= f.password_field :password, placeholder: "password" %><br />
			<%= f.submit %>
		<% end %>