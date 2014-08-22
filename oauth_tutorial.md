# OAuth

OAuth is an authentication protocol that allows you to approve one application interacting with another on your behalf without giving away your password.

###How does OAuth work?

1. **The user shows intent** – A request goes from your app to the service provider for permission. 
2. **Your App gets permission**– Your app sends a request to the service provider, asking for a request token and secret. The product secret will be penned on each request so the service provider can know where the request is coming from.
3. **The user is redirected to the service provider** - Your app redirects to the service provider for authorization. They use the request token.
4. **The user gives permission** – When granted, the service provider will grant access to the request token to be used. When your app requests access, it will be accepted (so long as it's signed using the same secret).
5. **Your app obtains an access token!** – From the request token and authorization, the product obtains an access token and secret.
6. The product can now access the protected resource! 

![image](https://opensocial.atlassian.net/wiki/download/attachments/526872/Oauth_3legged.png?version=1&modificationDate=1289933713495&api=v2)

###Things to remember:

* The `Redirect URI` or `Callback URL` must point back to your URL: `localhost:3000/github`. This will have to change when deploying your app!

	![image](https://s3.amazonaws.com/uploads.hipchat.com/39979/962485/T0LotXto0sTg1vH/screenshot.png)
* For the most part OAuth services have a similar creation flow, but they may have slightly different security features. For the Github OAuth Authentication, the Application Name is also a key code.
* `Client ID` is okay to tell people, but the `Client Secret` SHOULD NOT be shared. It should be in your *server*.
* OAuth requires a client_id and client_secret everytime.

Let's take a look at two different links for OAuth authentication:

	<a href="https://api.instagram.com/oauth/authorize/?client_id=<%= ENV["IG_CLIENT_ID"] %>&redirect_uri=http://localhost:3000/instagram&response_type=code">Log into Instagram</a>
	
	<a href="https://github.com/login/oauth/authorize?scope=user:email&client_id=<%= ENV["GH_CLIENT_ID"] %>">Log into Github</a>


### So how do we use the damn thing?

How to use OAuth authentication and create a user with a 3rd-party API, using HTTParty to help keep API requests in the background.

1. Add gem `HTTParty`. Run a `bundle install`.
2. Authorize an app on an OAuth supported service (Google, Facebook, Twitter, Soundcloud, Github, etc.). Look at "Things to Remember" to fill out the specific forms like Redirect URI.
3. Stash away the Client ID and Client Secret. You might want to add them in your `.bash_profile` for your development environment, or get a gem/plugin that hides your Client Secret for when you throw your code into production. 

	![image](https://s3.amazonaws.com/uploads.hipchat.com/39979/962485/uI4ML6GbGeRbmRR/screenshot.png)

4. For this specific example we're going to throw those in our `.bash_profile`. It might look something like this:

	<small><i>`.bash_profile`</i></small>
	
		export GH_CLIENT_ID="your_copied_code"
		export GH_CLIENT_SECRET="your_copied_code"
		
	Now we have access to these environmental variables on our development machine!
5. First thing first, we need to setup a request to grab a user's access token. Let's create a link with the correct URL:

		<a href="https://github.com/login/oauth/authorize?scope=user:email&client_id=<%= ENV["GH_CLIENT_ID"] %>">Log into Github</a>
		
6. This is going to lead to a dead URL though, because we haven't set our routes to do anything! To have a view do anything, we need to setup an action in our controller and route it to that controller's action. Let's create an action to define the outcome we want!

	Create a <small><i>`github_controller.rb`</i></small> and let's open that up. We can define two actions in here:

		def github
			# Find the access token
			res = HTTParty.post('https://github.com/login/oauth/access_token',
				{body:{client_id: ENV["GH_CLIENT_ID"],
				 client_secret: ENV["GH_CLIENT_SECRET"],
				 code: params[:code]},
				 headers: {"Accept" => "application/json"}})
			puts res.parsed_response.inspect
			redirect_to repos_path(acc_token: res.parsed_response["access_token"])
		end
		
		def repos
			# Show info about this user
			headers = {"User-Agent" => "APPLICATION NAME",
				"Authorization" => "token #{params[:acc_token]}"}
			url = "https://api.github.com/user"	# ?access_token=#{URI::escape(params[:acc_token])}
			@user_info = HTTParty.get(url, headers: headers)
			# Based on the URLs returned, let's find out about the repos
			@repos = HTTParty.get(@user_info["repos_url"], headers: headers)
		end

	Be aware of the "APPLICATION NAME" in `repos`. This should be replaced by your Github Application Name from your settings. 

7. Now, we need to create a route to point to this action. Remember, the link we click on will be sent to the routes, and the route will determine what happens by pointing to a specific action in the controller!

	Inside <small><i>`routes.rb`</i></small>:
	
		get '/github' => 'github#github'
	  	get '/github/repos' => 'github#repos', as: :repos

8. Success!
	  	
![image](http://media3.giphy.com/media/A9RhsvsdWz2aA/200w.gif)


****
###Shortcomings of OAuth

If our app was a super-shady Evil Co. it could pop up a window that looked like Github but was really phishing for your username and password.  As a user, always be sure to verify that the URL you’re directed to is actually the service provider (Github, in this case). As a company, let's take measures to deserve our users' trust.

Big thank you to Rob Sobers for his enlightening post that I pulled much of the content from. [http://blog.varonis.com/introduction-to-oauth/](http://blog.varonis.com/introduction-to-oauth/)