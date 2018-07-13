## 20180713 Today I Learned

### Google Social login (https://github.com/zquestz/omniauth-google-oauth2)

> Google authentication API allows web application to request login data from google. Google keeps track of our site and distinguish it via client_id and client_secret_key.

1. Register your project at Google API (https://console.developers.google.com/apis/)

2. Install figaro and save the client ID and client secret key for the Google API at application.yml (must include `gem 'figaro'` on the Gemfile and `figaro install` on console)

3. In devise.rb, include omniauth config for google :

   ```javascript
   config.omniauth :google_oauth2, ENV['GOOGLE_CLIENT_ID'], ENV['GOOGLE_CLIENT_SECRET'], {
       scope: 'email',
       prompt: 'select_account'
    }
   ```

4. Create migrate data with `rails g migrate add_columns_to_users` -

   ```ruby
   class AddColumnsToUsers < ActiveRecord::Migration[5.0]
     def change
       #add_column :DB명, :컬럼명,        :타입
       add_column :users, :provider,     :string
       add_colum :users, :name,          :string
       add_column :users, :uid,          :string # uid keeps record of the authentication
     end
   end
   ```

5. Add `:omniauthable, omniauth_providers: [:google_oauth2]` to the devise of model/user.rb - user is now authenticable with omniauth

6. Modify the route at route.rb to allow user devise to be accessible by omniauth: 

   ```ruby
   # This means that the controller will handle the omniauth_callbacks with controller under '/user/omniauth_callbacks_controller'
   devise_for :users, controllers: { omniauth_callbacks: 'users/omniauth_callbacks' }
   ```

7. `rails g devise:controllers users` : generate controller for user authentication, this will create user folder under controllers with 6 different controllers. (Note that users is plural!)

8. Update omniauth_callbacks_controller :

   ```ruby
   class User::OmniauthCallbacksController < Devise::OmniauthCallbacksController
     def google_oauth2
         # You need to implement the method below in your model (e.g. app/models/user.rb)
         @user = User.from_omniauth(request.env['omniauth.auth'])
   
         if @user.persisted?
           flash[:notice] = I18n.t 'devise.omniauth_callbacks.success', kind: 'Google'
           sign_in_and_redirect @user, event: :authentication
         else
           session['devise.google_data'] = request.env['omniauth.auth'].except(:extra) # Removing extra as it can overflow some session stores
           redirect_to new_user_registration_url, alert: @user.errors.full_messages.join("\n")
         end
     end
   end
   ```

9. Add a user model function at user.rb :

   ```ruby
   def self.from_omniauth(access_token)
       data = access_token.info
       user = User.where(email: data['email']).first
   
       # Uncomment the section below if you want users to be created if they don't exist
       unless user
           user = User.create(name: data['name'],
              email: data['email'],
              password: Devise.friendly_token[0,20],
              provider: access_okten.provider,
              uid: acess_token.uid
           )
       end
       user
     end
   ```



### Kakao Social Login

1. Create your kakao application in www.developers.kakao.com

2. Go to 일반, and add a platform with the site domain and redirect path.

   ```ruby
   devise_scope :user do
       get '/users/auth/kakao/', to: 'users/omniauth_callbacks#kakao'
       get '/users/auth/kakao/callback', to: 'users/omniauth#kakao_auth'
       #/users/auth/kakao/callback - this becomes your redirect path for your application
       # This setting automatically generate 'users_auth_kakao_path'
   end
   ```

3. Make sure to add your javascript API key from KAKAO to application.yml - add the REST_API key

4. Create view for devise gem with `rails g devise:views` (this shows all the code for the devise gem)

5. `views/devise/session/new.html.erb` add `<%= link_to "Sign in with Kakao", users_auth_kakao_path =%>` at the bottom

   - The link is according to mapping in `views/devise/shared_link.html.erb` 

     ```html
     <%- if devise_mapping.omniauthable? %>
       <%- resource_class.omniauth_providers.each do |provider| %>
         <%= link_to "Sign in with #{OmniAuth::Utils.camelize(provider)}", omniauth_authorize_path(resource_name, provider) %><br />
       <% end -%>
     <% end -%>
     ```

6. `omniauth_controller`: add kakao authenticating function

   ```ruby
   def kakao
       redirect_to "http://kauth.kakao.com/oauth/authorize?client_id=#{ENV['KAKAO_REST_API_KEY']}&redirect_uri=https://movie-maeror.c9users.io//users/auth/kakao/callback&response_type=code"
   end
   
   # Goes to kakao_auth after the user is authenticated
   def kakao_auth
       code = params[:code]
       base_url = "https://kauth.kakao.com/oauth/token"
       base_response = RestClient.post(base_url, {grant_type: 'authorization_code',
                                                  client_id: ENV['KAKAO_REST_API_KEY'],
                                                  redirect_url: 'https://movie-maeror.c9users.io/users/auth/kakao/callback&response_type=code',
                                                  code: code})
       res = JSON.parse(base_response)
       access_token = res["access_token"]
       info_url = "https://kapi.kakao.com/v2/user/me"
       info_response = RestClient.get(info_url, params: {secure_resource: false}, 
                                       Authorization: "Bearer #{access_token}")
       @user = User.from_omniauth_kakao(JSON.parse(info_response))
       if @user.persisted?
         flash[:notice] = "카카오 로그인에 성공했습니다."
         sign_in_and_redirect @user, event: :authentication
       else
         flash[:notice] = "카카오 로그인에 실패했습니다. 다시 시도해주세요."
         redirect_to new_user_session_path
       end
     end
   ```

7. Add a method to save the email to user data from Kakako_auth

   ```ruby
   # Add to Model user.rb
   def self.from_omniauth_kakao(token)
       user = User.where(provider: "kakao", uid: token["id"]).first
       if user
         user
       else
         if token["kakao_account"]["email"].present?
           user_email = token["kakao_account"]["email"]
         else
           user_email = "#{token["id"]}@kakao.com"
         end
         User.create(email: email,
                     password: Devise.friedly_token[0,20],
                     name: token["properties"]["nickname"],
                     uid: token["id"],
                     provider: "kakao"
                     )
       end
       user
   end
   ```

   