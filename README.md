# README

## How to make an app ?
* Let's go !
```
rails new -d postgresql photo_app
```

```
cd photo_app
```

- To setup homepage, generate a controller with an action : 
```
rails g controller welcome index
```

- Set the root routes to this in the _config/routes.rb_ file
```
root'welcome#index'
```

- Update this page in the _app/views/welcome_ folder in a file called **index.html.erb**

* Preparation the app for deployment to heroku : 
1. moving **sqlite3 gem to group development**
2. adding the gems :
```ruby 
group :production do
    gem 'pg', '>= 0.18', '< 2.0'
end
```
3. Run **bundle install**
```$ bundle i --without production```

4. Create a github repo
5. Create a heroku app by using 
```heroku create```

6. Deploy app to production : 
```git push heroku master```

Install devise :
```ruby
rails g devise:install
```

Create **USER** table :
```rails g devise User```

Extract the migration file
Link : [https://github.com/heartcombo/devise/wiki/How-To:-Add-:confirmable-to-Users]

Go to the **db>migrate>devise_create_users**
Uncommented ##confirmable 

Add **:confirmable** in models > user.rb

Doing 
```rails db:create db:migrate```

Add in **application_controller**
```before_action :authenticate_user!```

=> Obliger le visiteur a être loggé pour arriver sur la home page du site. 

Pour ignorer le log sur le site : 

Se rendre dans le **welcome_controller** et add : 

```skip_before_action :authenticate_user!, only [:index]** before the method.```

=> plus besoin d'être connecté pour accéder à la page d'accueil. 

* Générer une app bootstrap

```rails generate bootstrap:install static```
```rails generate bootstrap:layout application```

* Générer une view devise

```rails g devise:views:locale en```
```rails g devise:views:bootstrap_templates```

Delete 5 links to the favicon :)

In _app > assets > stylesheets > application.css_ adding :

```*= require devise_bootstrap_views```

Install email 

```heroku addons:create sendgrid:starter```

In heroku website : 

```Account setting > Billing```

Then in terminal :

```heroku addons:create sendgrid:starter```

In heroku website, _click on the project_ 
Then _click on Sendgrid logo_. 

In Sendgrid website, click on **Settings**. 
Then, click on **API keys**.

Create an Api Keys with name, authorisation. 

Recover **API Keys** created. 

* Configuration heroku with **API Keys**

```heroku config:set SENDGRID_USERNAME=apikey```

```heroku config:set SENDGRID_PASSWORD=apipassword```

* Create an **.env** file for putting the **apikey** and **apipassword**

```SENDGRID_USERNAME=''```
```SENDGRID_PASSWORD=''```

* Sendgrid configuration in App

In folder _config_
In file _environment.rb_, write this :

```
ActionMailer::Base.smtp_settings = {
    :user_name => ENV['SENDGRID_USERNAME'],
    :password => ENV['SENDGRID_PASSWORD'],
    :domain => 'monsite.fr',
    :address => 'smtp.sendgrid.net',
    :port => 587,
    :authentication => :plain,
    :enable_starttls_auto => true
  }
```

In folder _config_ > _environments_ > _development.rb_ :

```config.action_mailer.delivery_method = :letter_opener```

```config.action_mailer.perform_deliveries = true```

```config.action_mailer.default_url_options = { :host => '' }```

In folder _config_ > _environments_ > _production.rb_ :

```config.action_mailer.delivery_method = :smtp```

```config.action_mailer.default_url_options = { :host => 'ced-photo-app.herokuapp.com', :protocol => 'https' }```

For db:migrate on heroku : 
```heroku run rails db:migrate```

For change the mailer sender, **config > initializers > devise.rb** :
```config.mailer_sender = 'donot-reply@exemple.com'```
