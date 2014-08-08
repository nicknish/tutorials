##Railscast #250: Authentication from Scratch


### Signup

1. `rails g resource user name email password_digest`<br /> 
`password_digest` is the default name for the `has_secure_password` rails feature.
2. `rake db:migrate`
3. Inside User model, we add `has_secure_password`, a feature added in Rails 3. Also `attr_accessible` any fields the user can change in the form, for example:

		attr_accessible :name, :email, :password, :password_confirmation
		
		validates_uniqueness_of :email


4. Enable `bcrypt` gem (bundle)
5. Inside Users_Controller we want to add pretty straightforward signup capability:

		def new
			@user = User.new
		end
		
		def create
			@user = User.new(params[:user])
			if @user.save
				redirect_to root_url, notice: "Thank you for signing up!"
			else
				render "new"
			end
		end
		
6. Add a signup form, and add a link and path to the form. Protip: Add a response if you have any errors:
		
		<% if @user.errors.any? %>
			<div class="error_messages">
				<h2>Form is invalid<h2>
				<ul>
					<% @user.errors.full_messages.each do |message| %>
						<li><%= message %></li>
					<% end %>
				</ul>
			</div>
		<% end %>
				
### Login

1. `rails g controller sessions new` and give it `resources :sessions`
2. Add a login form and add a link to the form.
3. Add the create session method, notice the flash message and the `notice:`
		
		def create
			user = User.find_by_email(params[:email])
			if user && user.authenticate(params[:password])
				redirect_to root_url, notice: "Logged in!"
			else
				flash.now.alert = "Email or password is invalid"
				render "new"
			end
		end
4. Back on the layout page, add the flash message code:

		<% flash.each do |name, msg| %>
			<%= content_tag :div, msg, id: "flash_#{name}" %>
		<% end %>
		
### Signing Out

1. Inside Sessions_Controller, we need to be able to destroy sessions. Here's how:

		def destroy
			session[:user_id] = nil
			reset_session
			redirect_to root_url, notice: "Logged out."
		end		



### Changing the View according to Current User

1. To create a "global" variable to use, it has to go into the Application_Controller. 

		private
			def current_user
				@current_user ||= User.find(session[:use_id	)	
			end
			
			# We need a helper_method to make this accessible from the view
			helper_method :current_user
			
2. Go back to your `application.html.erb` or any view and make changes as you like. For ex:

		<% if current_user %>
			Logged in as <%= current_user.name %>.
			<%= link_to "Log out", session_path("current"), method: :delete %>
		<% else %>
			<%= link_to "Signup", new_user_path %> or
			<%= link_to "Login", new_session_path %>
		<% end %>
		

## Improving the User Experience

#### Friendly URLs

[example.com/sessions/new](example.com/sessions/new) isn't the cleanest looking URL. How can we change it so we simply see [example.com/login](example.com/login), and maintain friendly URLs?

1. Go in `routes.rb` and define three new routes:

		get 'signup' => 'users#new', as: 'signup'
		get 'login' => 'sessions#new', as: 'login'
		get 'logout' => 'sessions#destroy', as: 'logout'
		
	Now that our `new_sessions_path` changed to `login_path`, defined by  `as: 'login'` we have to go back into our views and modify the links!
	
2. Go back into your `application.html.erb`

		<% if current_user %>
			Logged in as <%= current_user.name %>.
			<%= link_to "Log out", signout_path("current"), method: :delete %>
		<% else %>
			<%= link_to "Signup", signup_path %> or
			<%= link_to "Login", login_path %>
		<% end %>
	
3. Victory!

#### Automatic Login after Signup

There's a problem with the signup flow. Whenever someone creates an account they have to click login and reenter their credentials. Let's create a better experience for our users and login for them!

1. Go into the Users_Controller and add:
		
		def create
		...
		if @user.save
			session[:user_id] = @user.id
			...
		...
		end
		
#### Admin Permissions

1. Inside Application_Controller:

		def authorize
			redirect_to login_url, notice: "Not authorized" if current_user == nil
		end

2. Back in our Users_Controller:

		before_filter :authorize, only: [:edit, :update]

The above will stop users who are not logged in from accessing other user's information. To go further, let's only allow users to edit their own information.

3. Again in the Users_Controller:

		def edit
		  if current_user.id == User.find(params[:id]).id
		    @user = User.find(params[:id])
		  else
		    redirect_to root_path, notice: "Sorry you don't have access to that!"
		  end
		end
	

