== README
Create a new app:
```ruby
rails new app
bundle install
```
Install Devise:
Add gem 'devise' to Gemfile
```ruby
bundle install
```
```ruby
rails generate devise:install
rails generate devise user
```
Install CanCan:
Add gem 'cancan' to Gemfile
```ruby
rails generate cancan:ability
```
```ruby
rails generate model role name:string
```
```ruby
rails generate migration UsersHaveAndBelongToManyRoles
```
Edit the migration 
```ruby
class UsersHaveAndBelongToManyRoles &lt; ActiveRecord::Migration 
  def self up 
    create_table :roles_users , :id =&gt; false do | t | 
      t. references :role , :user 
    end 
  end 

  def self down 
    drop_table :roles_users 
  end 
end 
```
```ruby
rake db:migrate
```
Edit User model 
```ruby
class User &lt; ActiveRecord::Base 
  has_and_belongs_to_many :roles 
  def role? ( role ) 
    return !! self roles find_by_name ( role. to_s camelize ) 
  end 
```
Edit Role model 
```ruby
class Role &lt; ActiveRecord::Base 
  has_and_belongs_to_many :users 
end 
```
Edit Ability model 
```ruby
class Ability
  include CanCan::Ability 

  def initialize ( user ) 
    user || = User. new # guest user 

    if user. role :super_admin 
      can :manage , :all 
    elsif user. role :product_admin 
      can :manage , [ Product, Asset, Issue ] 
    elsif user. role :product_team 
      can :read , [ Product, Asset ] 
      # manage products, assets he owns 
      can :manage , Product do | product | 
        product. try ( :owner ) == user
      end 
      can :manage , Asset do | asset | 
        asset. assetable try ( :owner ) == user
      end 
    end 
  end 
end 
```
```
mkdir app / controllers / users
vi app / controllers / users / registrations_controller. rb 
```
Edit RegistrationsController 
```ruby
class Users::RegistrationsController &lt; Devise::RegistrationsController 
  before_filter :check_permissions , :only =&gt; [ : new , :create , :cancel ] 
  skip_before_filter :require_no_authentication 
  def check_permissions
    authorize! :create , resource
  end 
end 
```
Edit config/routes.rb and replace devise_for :users with: 
```ruby
devise_for :users , :controllers =&gt; { :registrations =&gt; "users/registrations" } 
```
Edit ApplicationController 
```ruby
class ApplicationController &lt; ActionController::Base 
  ...
  rescue_from CanCan::AccessDenied do | exception | 
    flash [ :error ] = exception. message 
    redirect_to root_url
  end 
  ...
end 
```
Add WelcomeController 
```ruby
rails generate controller welcome index
```
Navigating to / users / sign_up will now redirect you to welcome #index
