---
layout: post
title: "The Anatomy of the JMeter Load Test"
date: 2015-10-23 13:52:33 -0500
comments: true
categories: 
---

Yesterday, my boss told me to write load tests for an endpoint in our Rails application. At first, I was all like:

<center>{% img https://media.giphy.com/media/v6bWA3jIqyamQ/giphy.gif 300 %}</center>

But once I delved into the world of Apache JMeter, I realized that my knowledge of web requests would make understanding load testing a piece of cake.

<center>{% img https://media.giphy.com/media/TPOjURhuAenII/giphy.gif 300 %}</center>

## Setup

In order to gain a better understanding of JMeter, I wanted to write load tests for the simplest type of Rails app – a Rails scaffold. Let's set up our trusty Cat App:

```
rails new cat-app
cd cat-app
rails g scaffold cat name
rails s -p 8080
```

For the most part, this post assumes that you've been able to set up JMeter with your Rails app. In a nutshell, I added the location of the JMeter `bin` directory to my `PATH`, and then launched JMeter with the local host as `-Jhostname`:

```
export PATH=~/Development/apache-jmeter-2.13/bin:$PATH
JVM_ARGS="-Xms512m -Xmx2048m" && jmeter -Jhostname=127.0.0.1
```

## Testing Components

To start testing the app, I added a Thread Group called Basic App Calls to the Test Plan (by right-clicking on Test Plan in the sidebar). Then, I added a Loop Controller called Cats to the Test Plan, and set the Loop Count to 10. All of the HTTP Requests that I tested were contained under the Cats Loop Controller.

I also added an HTTP Cookie Manager and a Summary Report to the Basic App Calls Thread Group.

## The Simplest Case

Let's start with our `cats#index` test. We shouldn't need to pass any parameters into this request – all we have to do is visit `/cats`.

Rails set up this controller test:
```
test "should get index" do
  get :index
  assert_response :success
  assert_not_nil assigns(:cats)
end
```

So in order to test that this call works correctly, we just have to set up the correct HTTP Request Sampler under the Cats Loop Controller.

Under Server Name we'll use our `localhost`, which gets entered as `${__P(hostname,)}`. We're running on port 8080, which we also need to specify.

Under HTTP Request, let's set the method to GET and the path to `/cats`. The final settings look like this:

<center>{% img http://i.imgur.com/k9KSbsm.png 600 %}</center>

If we run our tests, we should see the request successfully hit the server and return a 200 status code. Easy!

## Rails Authenticity Tokens

Now that we've created a simple load test, we'll want to actually create a cat to test with. First off, we can set the settings, which are self-explanatory enough: Server Name `${__P(hostname,)}`, Port Number `8080`, HTTP Request `POST`, and Path `/cats`.

We know that we'll need to post some data with our request – luckily, the HTTP Request section includes a list of parameters. Let's add `cat[name]` as the Name, and a good name for a cat as the Value. I chose `Meow`.

However, if we run the tests, the summary report shows an 100% error percentage for Create Cat. If we look at the error in Terminal, we can see that the `POST` request returned a `422 Unprocessable Entity` status code and a `Can't verify CSRF token authenticity` error. This makes sense – we know that we can't post to a form in Rails without an authenticity token.

Luckily, with a little creativity, we can grab that token from another HTTP Request Sampler. Let's set up an HTTP Request for `cats#new`: Server Name `${__P(hostname,)}`, Port Number `8080`, HTTP Request `GET`, and Path `/cats/new`.

<center>{% img http://i.imgur.com/EiIUKIS.png 600 %}</center>

When I opened up `http://localhost:8080/cats/new`, I could see that the form had an input tag: `<input type="hidden" name="authenticity_token" value="TOKEN_HERE">`. I wanted to grab that token and pass it into my `POST` params, which I could do with a Regular Expression Extractor nested under my `new` HTTP request. The Regex Extractor looks like this:

<center>{% img http://i.imgur.com/s5xXg7N.png 600 %}</center>

Whatever is within the parentheses gets captured and stored in a variable `FORM_AUTH_TOKEN`. So now, we'll want to add another parameter to our `POST` request – Name: `authenticity_token`, Value: `${FORM_AUTH_TOKEN}`. Make sure to check the Encoded checkbox for this parameter.

<center>{% img http://i.imgur.com/wLJLYnl.png 600 %}</center>

Now, if you run the tests, they should all return 0% errors.

## Let's See the Cat!

In order to make a call to `cats#show` we'll need the specific ID of the cat that we wish to view. There are a few ways to grab this, but since I'm not familiar with JMeter, I decided to use a Regex Extractor like the one I used for the authenticity token. Since the `create` action redirects to the `show` page, the HTML response for `create` includes a link that has the cat's ID. So I set up my Regex Extractor as follows:

<center>{% img http://i.imgur.com/5eG76LL.png 600 %}</center>

Now that I saved the cat's ID into a `CAT_ID` variable, I can embed it in the path for my `cats#show` request. The rest is pretty simple:

<center>{% img http://i.imgur.com/qKBY9BT.png 600 %}</center>

## Destruction

Unfortunately, our testing took a turn for the evil – we had to destroy our cat! The call to `cats#destroy` is a `DELETE` request to `/cats/${CAT_ID}`. The only catch is that we need both our ID and our authenticity token, but they're all set up, so it's easy. Remember to encode the token:

<center>{% img http://i.imgur.com/4c8Xmo9.png 600 %}</center>

Now, if we run our tests, we get no errors on any of the tests and we can start tracking performance!

<center>{% img http://i.imgur.com/qpbABjc.png 600 %}</center>

Hooray!

<center>{% img https://media.giphy.com/media/Md4xQfuJeTtx6/giphy.gif 300 %}</center>