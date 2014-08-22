#Creating a Simple API with Ruby on Rails with AngularJS

Create a simple API with Angular and RoR using Matthias's Anagram Homework.

1. Inject AngularJS script in `HEAD` inside your `application.html.erb`

		<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.0-beta.17/angular.min.js"></script>
		
2. Add a `module` and `controller` to your own script. 

		angular.module('angulartest', []).controller('aCtrl', ['$scope', '$http', function($scope, $http) {}]);
		
3. Inside your HTML file add `ng-app` and `ng-controller` <small>(<i>views > layouts> application.html.erb</i>)</small>.
4. Define a method inside your `controller.rb`. This is your API waiting to be called. 
<br />ex. <small>(<i>controllers > anagrams_controller.rb)</i></small>
								
		def check_anagram
			s = params[:s]
			h = Hash.new
			s.chars.uniq.each { |x| h[x] = s.count(x).even? }
			render json: h.values.count(false) <= 1 ? 1 : 0
		end
		
5. Go into your `routes.rb` and create a route so the API can be called on. You can test this by checking in the browser `localhost:3000/api/your_string`
		
		get 'api/:s' => 'anagrams#check_anagram'
		
6. Back to the script, define a function with an `$http GET` request so the API can be called on from the DOM.

		$scope.is_anagram = function() {
          var data = $http.get('http://localhost:3000/api/' + $scope.teststring).success(function(data, status, headers, config) {
              $scope.data = data;
          });
        };
        
7. Call the function from the DOM using `ng-change` or another directive to run your `is_anagram` function.

		<input type="text" ng-model="teststring" ng-change="is_anagram()" ng-class="{ is_anagram: data == 1, not_anagram: data == 0 }" />  {{ data }}
		
		
		
		
## Why do this?

The power of AngularJS on Rails is that we can query the database asynchronously without reloading the page. This is huge improvement for creating good user experiences.