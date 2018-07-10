

## 20180710 Today I Learned

### Chatrooms!

Required Gems : pusher, figaro, devise

1. Use four modes : chat_rooms, chats, admission, users!

   - chat_room: title, master_id, max_count, admission_count
   - chats: user (reference - more intuitve syntax), chat_room (reference), message
   - users: use devise to create
   - admission: (M:N connection model) chat_room, user

   ```ruby
   # Chatrooms
   t.string :title
   t.string :master_id
   t.integer :max_count
   t.integer :admissions_count, default: 0
   # Chats
   t.references :user
   t.references :chat_room
   t.text :message
   # Admissions
   t.references :chat_room
   t.references :user
   #User
   has_many :admissions
   has_many :chat_rooms, through: :admissions
   has_many :chats
   ```

2. Model relations

   - `belongs_to :chat_room, counter_cache: true` : counter_cache enables automated counting of the number of chatrooms
   - `has_many :users, through: admissions` : M:N relationship connection for chat_room!

   ```ruby
   # Chat_room
   has_many :admissions
   has_many :users, through: :admissions
   has_many :chats
   # Chat
   belongs_to :user
   belongs_to :chat_room
   # Admission
   belongs_to :user
   belongs_to :chat_room, counter_cache: true
   # User
   has_many :admissions
   has_many :chat_rooms, through: :admissions
   has_many :chats
   ```

3. Chatroom admission

   - `after_commit :<method_name>, on: :create` - after a model is created, execute a method

   ```ruby
   # Chat_room.rb - create a model function that puts user in the chatroom
   def user_admit_room(user)
   	Admission.create(user_id: user.id, chat_room_id: self.id)
   end
   
   # Chat_room controller
   def create
       @chat_room = ChatRoom.new(chat_room_params)
       @chat_room.master_id = current_user.email # Set the master_id
       respond_to do |format|
         if @chat_room.save
           @chat_room.user_admit_room(current_user) # Pass in the current user as master
           format.html { redirect_to @chat_room, notice: 'Chat room was successfully created.' }
           format.json { render :show, status: :created, location: @chat_room }
         else
           format.html { render :new }
           format.json { render json: @chat_room.errors, status: :unprocessable_entity }
         end
   	end
   end
   
   def user_admit_room # Make another function on the controller that uses model function to admit users to the room
       # 현재 유저가 있는 방
       @chat_room.user_admit_room(currnet_user)
   end
   ```

4. Changing the input form

   ```html
   <!-- _form.html.erb -->
   <div class="form-group">
   	<%= f.label :title %>
   	<%= f.text_field :title %>
   </div>
   <div class="form-group">
   	<%= f.label :max_count %>
   	<%= f.number_field :max_count %>
   </div>
   ```

5. Pusher - log in and create a project with Jquery frontend and rails backend

   > Pusher is similar to AJAX in that it works asynchronously with the client browser. There is a trigger in the server which sends the event and data to pusher, and pusher subscribe to this trigger with channel even and data and send it back to user. Similar to AJAX request and response but the server reaches the client through the Pusher application

   - `figaro install` from rails terminal

   - Add to `/config/application.yml`:

     ```ruby
     development:
         pusher_app_id: <ID>
         pusher_key: <KEY>
         pusher_secret: <SECRET>
         pusher_cluster: <CLUSTER>
     # Complete this according to your application specifics, do not release this!
     ```

   - Create `/config/initializers/pusher.rb` for validation:

     ```ruby
     require 'pusher'
     
     Pusher.app_id = ENV["pusher_app_id"]
     Pusher.key = ENV["pusher_key"]
     Pusher.secret = ENV["pusher_secret"]
     Pusher.cluster = ENV["pusher_cluster"]
     Pusher.logger = Rails.logger
     Pusher.encrpyted = true
     ```

   - Allow pusher in `layout/application.html.erb` with `<script src="https://js.pusher.com/4.1/pusher.min.js"></script>`

6. Use Trigger and Subscribe to set functions similar to AJAX

   - Speciftying triggers on the model

   ```ruby
   # Admission.rb
   class Admission < ApplicationRecord
       # This checks that paring of user_id and chat_room_id must be unique
       validates_uniqueness_of :user_id, :scope => "chat_room_id"
       belongs_to :user
       belongs_to :chat_room, counter_cache: true
       # if there is a new admission, automatically call for the user_joined_chat_room_notification
       after_commit :user_joined_chat_room_notification, on: :create
       # 유저가 챗방에 들어왔다는 사실을 알려줌
       def user_joined_chat_room_notification
           # Trigger Pusher to channel 'chat_room', with method 'join' and pass the necessary data as json
           Pusher.trigger('chat_room', 'join', {chat_room_id: self.chat_room_id, email: self.user.email}.as_json)
       end
   end
   ```

   ```ruby
   # Chat_room.rb
   class ChatRoom < ApplicationRecord
       has_many :admissions
       has_many :users, through: :admissions
       has_many :chats
       
       after_commit :create_chat_room_notification, on: :create
       
       def create_chat_room_notification
         Pusher.trigger('chat_room', 'create', self.as_json)
                     # (Channel_name, event_name, data) 
       end
       
       # 유저를 방에 넣기 방장을 방에 넣기!
       def user_admit_room(user)
         Admission.create(user_id: user.id, chat_room_id: self.id)
       end
       
   end
   ```

   - Subscribe through Javascript on the view pages

   ```html
   <!-- index.html.erb -->
   <% if user_signed_in? %>
     <%= current_user.email %> / <%= link_to 'Log out', destroy_user_session_path, method: :delete %>
   <% else %>
     <%= link_to 'Log in', new_user_session_path %>
   <% end %>
   
   <h1>Chat Rooms</h1>
   <table>
     <thead>
       <tr>
         <th>Room title</th>
         <th>Num Member</th>
         <th>Master ID</th>
         <th colspan="3"></th>
       </tr>
     </thead>
   	
     <tbody class="chat_room_list">
       <!-- load all the existing room -->
       <% @chat_rooms.reverse_each do |chat_room| %>
         <tr>
           <td><%= chat_room.title %></td>
           <!-- surround the current room size with span to access it through javascript -->
           <td><span class="current<%= chat_room.id %>"><%= chat_room.admissions.size %></span> 
           / <%= chat_room.max_count %></td>
           <td><%= chat_room.master_id %> </td>
           <td><%= link_to 'Show', chat_room %></td>
          </tr>
       <% end %>
     </tbody>
   </table>
   <br>
   <%= link_to 'New Chat Room', new_chat_room_path %>
   
   <script>
   $(document).on('ready',function() {
     // jquery method to view the created room right away
     function room_created(data) {
       $('.chat_room_list').prepend(`
         <tr>
           <td>${data.title}</td>
   	<!-- initialize the room with 0 person, this will be incremented right away --> 
           <td><span class="current${data.id}">0</span> / ${data.max_count} </td>
           <td>${data.master_id}</td>
           <td><a href="/chat_rooms/${data.id}">Show</a></td>
         </tr>`);
     };
     // jquery method to increase the number of members in chatroom when a user joins the chatroom
     function user_joined(data) {
       var current = $(`.current${data.chat_room_id}`);
       console.log(current);
       current.text(parseInt(current.text())+1);
     }
     // Create and authenticate the pusher object to use
     var pusher = new Pusher('<%= ENV["pusher_key"]%>', {
         cluster: "<%= ENV["pusher_cluster"]%>",
         encrypted: true
       });
     // Create a channel to recieve the trigger on.
     var channel = pusher.subscribe('chat_room');
     // Two methods that are conditioned on different function, they call the jquery function above
     channel.bind('create', function(data) {
       console.log(data);
       room_created(data);
     });
     channel.bind('join', function(data) {
       console.log(data);
       user_joined(data);
     });
   });
   </script>
   ```

   ```html
   <!-- show.html.erb -->
   <%= current_user.email %>
   
   <!-- Printing out the list of current members -->
   <h3>Current Chatroom Members</h3>
   <div class="joined_user_list">
   <% @chat_room.users.each do |user| %>
       <p><%= user.email %></p>
   <% end %>
   </div>
   <hr/>
   
   <% if @chat_room.users.exclude?(current_user) %>
       <!-- the condition "remote:true" automatically becomes AJAX, need to create user_admit_room.js.erb for AJAX response -->
       <%= link_to 'Join', join_chat_room_path(@chat_room), method: 'post', remote: true, class: "join_room" %>
   <% end %>
   <%= link_to 'Edit', edit_chat_room_path(@chat_room) %> |
   <%= link_to 'Back', chat_rooms_path %>
   
   <script>
   $(document).on('ready', function() {
       // Similar to the index, we create a channel and subscribe methods to update the member list in the chatroom when a user joins
       function user_joined(data) {
           $('.joined_user_list').append(`<p>${data.email}</p>`)  
       };
       var pusher = new Pusher('<%= ENV["pusher_key"]%>', {
         cluster: "<%= ENV["pusher_cluster"]%>",
         encrypted: true
       });
       var channel = pusher.subscribe('chat_room');
   
       channel.bind('join', function(data) {
           console.log(data);
           user_joined(data);
       });
       
   });
   </script>
   ```

#### Appendix - Code et cetera

```ruby
# Routes.rb
Rails.application.routes.draw do
  root 'chat_rooms#index'
  resources :chat_rooms do
    member do
      # Post method since we are putting the email on this member list
      # create route for join url to a user_admit_room function
      post '/join' => 'chat_rooms#user_admit_room', as: 'join'
    end
  end
  devise_for :users
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

```javascript
// user_admit_room.js
alert("Just Joined!")
$('.join_room').remove(); //Remove the .join_room class element
```

```ruby
# Chat_rooms_controller.rb
class ChatRoomsController < ApplicationController
  # add we need to find the chat_room to admit the user to!
  before_action :set_chat_room, only: [:show, :edit, :update, :destroy, :user_admit_room]
  # Need to authenticate to join rooms and view rooms
  before_action :authenticate_user!, except: [:index]
  
  # GET /chat_rooms
  # GET /chat_rooms.json
  def index
    @chat_rooms = ChatRoom.all
  end

  # GET /chat_rooms/1
  # GET /chat_rooms/1.json
  def show
  end

  # GET /chat_rooms/new
  def new
    @chat_room = ChatRoom.new
  end

  # GET /chat_rooms/1/edit
  def edit
  end
  
  def user_admit_room
    # 현재 유저가 있는 방
    @chat_room.user_admit_room(current_user)
  end

  # POST /chat_rooms
  # POST /chat_rooms.json
  def create
    @chat_room = ChatRoom.new(chat_room_params)
    @chat_room.master_id = current_user.email
    respond_to do |format|
      if @chat_room.save
        # Allow the master to join the room right away
        @chat_room.user_admit_room(current_user)
        format.html { redirect_to @chat_room, notice: 'Chat room was successfully created.' }
        format.json { render :show, status: :created, location: @chat_room }
      else
        format.html { render :new }
        format.json { render json: @chat_room.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /chat_rooms/1
  # PATCH/PUT /chat_rooms/1.json
  def update
    respond_to do |format|
      if @chat_room.update(chat_room_params)
        format.html { redirect_to @chat_room, notice: 'Chat room was successfully updated.' }
        format.json { render :show, status: :ok, location: @chat_room }
      else
        format.html { render :edit }
        format.json { render json: @chat_room.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /chat_rooms/1
  # DELETE /chat_rooms/1.json
  def destroy
    @chat_room.destroy
    respond_to do |format|
      format.html { redirect_to chat_rooms_url, notice: 'Chat room was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_chat_room
      @chat_room = ChatRoom.find(params[:id])
    end

    # Accept title and max_count when creating chatroom
    def chat_room_params
      params.fetch(:chat_room, {}).permit(:title, :max_count)
    end
    
end

```



