# User Registration and Authentication

## Summary
In this challenge we're going to explore user registration and authentication.  To begin, our application will be limited to allowing users to register for our site and afterward login and logout.  Once our application has this functionality, we'll look at authorization—allowing logged in users access to content that guests cannot access.


### Working with Sessions
HTTP is a [stateless protocol][], meaning that each request-response cycle is independent of all other requests and responses.  Servers don't remember which clients made which requests or to whom they've sent which responses.  But, we know that if we log into a website, the application knows that we're logged in.  In other words, our logged-in state is preserved across multiple requests.  How does this happen?

In this challenge, we'll learn to track whether or not a user is logged in using [sessions][] in Sinatra, which store data using HTTP cookies.  [Cookies][HTTP cookies] are used to retain state across web requests.  Cookies are passed back-and-forth between client and server in HTTP headers.  The server sets a cookie and sends it to the client as part of the response.  Then with its next request, the client sends the cookie back to the server.  In this manner, we're able to manage state across multiple request-response cycles.


### Protecting User Data with BCrypt
When users register with our application, they trust us with their data:  names, e-mail addresses, passwords, etc.  We must do everything we can to protect our users in case our database is compromised.  To that end, we should never store a user's plain-text password in our database.

But that does raise a problem.  Users register for our site with an e-mail address and a password.  We can't save that password.  But, when users return to the site and log in, we have to check that they enter the correct password.  How do we do that?

Instead of storing the password itself, we can store a transformed version of it.  One option is to run the password through a [hashing algorithm](https://en.wikipedia.org/wiki/Cryptographic_hash_function) and then store the hashed version in our database.  One such hashing algorithm is [bcrypt](https://en.wikipedia.org/wiki/Bcrypt), for which there is a [Ruby gem](https://github.com/codahale/bcrypt-ruby).  Figure 1 shows an example of using the gem to create a hashed version of a string.

```ruby
require 'bcrypt'

hashed_password = BCrypt::Password.create("password")
# => "$2a$10$KncjtgHorSeQcQyzOQjKVOWS6/nhjGo8vuvbb4H54QyuC41c10DQ6"
```  
*Figure 1*.  Using the BCrypt gem to hash a password.


Saving the hashed password in the database means that we can avoid storing plain-text passwords.  But if all we save is the hashed password, when a user logs in, how do we check the password submitted in the log-in form?  

We ask if the plain-text password is the same as the string originally used to created the hashed password.  How do we determine that?  The hashed password is made up of two parts:  the first part is the *salt* and the second part is the *checksum* (see Figure 2).  When we compare a hashed password to a plain-text string, we take the string and the salt from the hashed password, we given them both to the hashing algorithm, and we see whether the algorithm produces the same hashed password.  All of this is provided for us in the [`BCrypt::Password`][bcrypt password] class, and we can use the [`#==`][bcrypt password equality] method to ask a `BCrypt::Password` if it was made from a given string (see Figure 3).

```ruby
hashed_password
# => "$2a$10$KncjtgHorSeQcQyzOQjKVOWS6/nhjGo8vuvbb4H54QyuC41c10DQ6"
hashed_password.salt
# => "$2a$10$KncjtgHorSeQcQyzOQjKVO"
hashed_password.checksum
# => "WS6/nhjGo8vuvbb4H54QyuC41c10DQ6"
```
*Figure 2*.  Accessing a `BCrypt::Password` object's salt and checksum values.

```ruby
hashed_password == "my secret"
# => false
hashed_password == "password"
# => true
```  
*Figure 3*.  Determining whether a hashed password was made from a particular string.


## Releases
### Release 0:  Implement User Registration
The first feature that we'll build out is user registration.  When users visit our site, they should be able to create new accounts by submitting some basic information:

- full name
- e-mail address
- password

When users register with our site, we'll need to persist the information they provided in the database (i.e., save the new user).  What data should we store?  How should we store it?  Remember, we should not store a user's plain-text password.  What constraints should we have in our database? What validations on our models?  What would happen if two users registered with the same e-mail address?  Or, if a user did not supply an e-mail address?

*Note:*  When users later return to our site and attempt to login, they will submit an email address and password.


### Release 1:  Implement User Login
When users register for our site, we'll want them to be able to later return to our site and login using the details that they provided when registering—specifically, the e-mail address and password.  How will our application determine whether or not the user supplied a correct e-mail and password combination?

```ruby
class User < ActiveRecord::Base
  # e.g., User.authenticate('jesse@devbootcamp.com', 'apples123')
  def self.authenticate(email, password)
    # if email and password correspond to a valid user, return that user
    # otherwise, return nil
  end
end
```
*Figure 4*.  Shell code for an authenticate method.

We'll add an `.authenticate` method to our `User` model to make this determination.  Figure 4 provides some shell code and pseudocode for our method.

Now, let's actually implement logging in by allowing users to submit their login credentials as a post request to a `/login` URL.  If the user sends a valid email and password combination, we'll log the user in.  We'll "remember" that the user is logged in by storing data in the `session` hash.

If the `session` hash and/or how to use it is unclear, read the [using sessions section][using sessions] of the Sinatra documentation.  If it is still unclear after reading the documentation, ask for help from a staff member.


### Release 2:  Implement Logging Out
If users can login, we'll also want to allow them to logout.  Implement a controller method that will log a user out when they visit it.  This will most likely involve deleting some content from the `session` hash.


### Release 3:  Implement Authorization
To this point, we've been dealing with *authentication*:  answering the question, "Who are you?"  Now we're going to handle *authorization*:  answering the question, "What do you have permission to do?"

In applications there are different use cases for authorization.  For example, an application might have a group of administrators with special privileges.  Perhaps they can edit content, delete posts, etc., while other users cannot.  In our application, we'll authorize all logged in users to view the content of our site.  If users have not logged in, they will not be authorized to view the content.

What is our content?  We're practicing authorization, so let's just create a "secret" page.  Users should be authorized to see the secret page only if they're logged in.  If users try to access the secret page when they're not logged in, they should be redirected to the login page.

One way to restrict access to authorized users is a [before filter][].  This not the only way to accomplish this and not necessarily the best way in this case.  But it's one tool to implement this kind of pre-route logic.  For an application this simple, it's ok to put the authorization logic in the route handler itself.


## Conclusion
Think about the apps we use everyday:  Twitter, Instagram, GitHub, etc.  User registration, authentication, and authorization are key aspects in these applications.  These are skills that we must have.  We've gotten an introduction to these concepts in this challenge.  Moving forward at Dev Bootcamp we'll receive more practice with them as we'll continue building user authentication into our applications.

[bcrypt password]: https://github.com/codahale/bcrypt-ruby/blob/master/lib/bcrypt/password.rb
[bcrypt password equality]: https://github.com/codahale/bcrypt-ruby/blob/master/lib/bcrypt/password.rb#L65
[sessions]: http://www.sinatrarb.com/faq.html#sessions
[HTTP cookies]: http://en.wikipedia.org/wiki/HTTP_cookie
[using sessions]: http://www.sinatrarb.com/intro#Using%20Sessions
[before filter]: http://www.sinatrarb.com/intro#Filters
[stateless protocol]: https://en.wikipedia.org/wiki/Stateless_protocol
