# README

This README would normally document whatever steps are necessary to get the
application up and running.

Things you may want to cover:

* Ruby version

* System dependencies

* Configuration
**rails new -d postgresql photo_app**
cd into the app: 
**cd photo_app**

- To setup homepage, generate a controller with an action : 
**rails g controller welcome index**
- Set the root routes to this in the _config/routes.rb_ file
**root'welcome#index'**
- Update this page in the _app/views/welcome_ folder in a file called **index.html.erb**

* Preparation the app for deployment to heroku : 
1. moving sqlite3 gem to group development
2. adding the gems :
```ruby 
group :production do
    gem 'pg', '>= 0.18', '< 2.0'
end
```
3. Run **bundle install**
$ bundle i --without production
4. Create a github repo
5. Create a heroku app by using **heroku create**
6. Deploy app to production : **git push heroku master**


Install devise :
```ruby
rails g devise:install
```

Create **USER** table :
```ruby
rails g devise User
```

Extract the migration file
(doc)[https://github.com/heartcombo/devise/wiki/How-To:-Add-:confirmable-to-Users]

Go to the **db>migrate>devise_create_users**
Uncommented ##confirmable 

Add **:confirmable** in models > user.rb

Doing **rails db:create db:migrate**

Add **before_action :authenticate_user!** in **application_controller**

Obliger le visiteur a être loggé pour arriver sur la home page du site. 

Pour ignorer le log sur le site : 

Se rendre dans le **welcome_controller** et add : 
- **skip_before_action :authenticate_user!, only [:index]** before the method.

=> plus besoin d'être connecté pour accéder à la page d'accueil. 

* Générer un app bootstrap
**rails generate bootstrap:install static**

**rails generate bootstrap:layout application**

* Générer une view devise

**rails g devise:views:locale en**
**rails g devise:views:bootstrap_templates**

Delete 5 links to the favicon :)

In _app > assets > stylesheets > application.css_ adding :
- ***= require devise_bootstrap_views**