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


* Build homepage

In **app > views > welcome > index.html.erb**, add :
```styling for home page```

* Add css jumbotron 
In **app > assets > stylesheets > custom.css.scss**
```.features```
```.jumbotron```
```.no-left-padding```
```.listing```

In **app > assets > images > stylesheets**, download image : 
```css
.jumbotron {
      background-image: asset-url('meeting.png');
}
```
In **app/views/layouts** folder, the **application.html.erb** page 
change the following to ```col-lg-12``` from ```col-lg-9```


# Stripe and Payment introduction 

Accepter les paiements ou intégrer un moyen de paiement. 
Pour s'inscrire sur le site, les users doivent payer pour accéder au contenu du site. 

Pour le moment, les users peuvent simplement s'inscrire à leur compte en entrant leur password et leur confirmation de mot via un lien d'activation dans leur e-mail.


Adding the Stripe gem to your application's **Gemfile** :

```gem 'stripe'```

Then, run ```bundle install --without production```

Après, créer un fichier ```stripe.rb``` dans le dossier **config/initializers** écrire :

```Rails.configuration.stripe = {
  :publishable_key => ENV['STRIPE_TEST_PUBLISHABLE_KEY'],
  
  :secret_key => ENV['STRIPE_TEST_SECRET_KEY']
}

Stripe.api_key = Rails.configuration.stripe[:secret_key]
```

Récupérer les clés API dans le dashboard de l'admin de Stripe. 

### Installer la gem devise
Se rendre ```https://github.com/heartcombo/devise```
Mettre la gem ```devise``` dans le ```Gemfile```

Inscrire les clés dans le fichier ```.env```

Configurer sur Heroku.

Dans le terminal :

```heroku config:set STRIPE_TEST_SECRET_KEY=```
```heroku config:set STRIPE_TEST_PUBLISHABLE_KEY=```


Parcourir cet onglet :
```https://stripe.com/docs/legacy-checkout/rails```

Le formulaire d'inscription du site doit être capable de prendre en compte la feature de paiement de Stripe. 

1. JavaScript pour gérer la soumission du formulaire à Stripe
2. Récupérer les attributs imbriqués dans les tokens lors de l'inscription.

## PAYMENT MODEL

Il va avoir un email pour l'utilisateur ID, un token renvoyé de Stripe que je dois enregistrer, qui est une chaîne. 

```rails generate model Payment email:string token:string user_id:integer```

Je migre cette nouvelle table

```rails db:migrate```


Je lie le model du paiement avec le model de l'user. 

Dans le fichier ```app > models > user.rb```

- ```has_one :payment```

Le client aura un seul unique ID de paiment pour s'inscrire sur le site. 

Dans le même fichier : 

- ```accepts_nested_attributes_for :payment```

Dans le fichier ```app > models > payment.rb```
- ```belongs_to :user```

Le model **payment** fonctionne avec les attributs des informations de carte de crédit (pour que le JavaScript envoie les infos à Stripe), puis les supprime. 

Dans le même fichier :

- ```attr_accessor :card_number, :card_cvv, :card_expires_month, :card_expires_year```

Rédiger des méthodes pour faire fonctionner le paiement : 

```ruby
def self.month_options
  Date::MONTHNAMES.compact.each_with_index.map { |name, i| ["#{i+1} - #{name}", i+1]}
end

def self.year_options
  (Date.today.year..(Date.today.year+10)).to_a
end

def process_payment
  customer = Stripe::Customer.create email: email, card: token
  
Stripe::Charge.create customer: customer.id,
                        amount: 1000,
                        description: 'Premium',
                        currency: 'eur'
                        end
end
```

## Update form for Credit Card Payments 
### Update sign-up form to accept credit card information

Ajouter les champs de traitement des cartes de crédit dans le formulaire d'inscription.

Dans le fichier ```views > devise > registrations > new.html.erb``` Line 24 : 

```ruby
<%= fields_for( :payment) do |p| %>
    <div class="row col-md-12">
      <div class="form-group col-md-4 no-left-padding">
        <%= p.label :card_number, "Card Number", data: { stripe: 'label' } %>
        <%= p.text_field :card_number, class: "form-control", required: true, data { stripe: 'number' } %>
      </div>
      <div class="form-group col-md-2">
        <%= p.label :card_cvv, "Card CVV", data: { stripe: 'label' } %>
        <%= p.text_field :card_cvv, class: "form-control", required: true, data { stripe: 'cvv' } %>
      </div>
      <div class="form-group col-md-6">
        <div class="col-md-12">
          <%= p.label :card_expires, "Card Expires", data: { stripe: 'label'} %>
        </div>
        <div class="col-md-3">
          <%= p.select :card_expires_month, options_for_select(Payment.month_options), { include_blank: 'Month' }, "data-stripe" => "exp-month", class: "form-control", required: true %>
        </div>
        <div class="col-md-3">
          <%= p.select :card_expires_year, options_for_select(Payment.year_options.push), { include_blank: 'Year' }, class: "form-control", data: { stripe: "exp-year" }, required: true %>
        </div>
      </div>
    </div>
<% end %>
```

