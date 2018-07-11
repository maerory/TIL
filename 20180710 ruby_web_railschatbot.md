

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

   > Pusher is similar to AJAX in that it works asynchronously with the client browser. There is a trigger in the server which sends the event and data to pusher, and pusher subscribe to this trigger with channel even and data and send it back to user. Similar to AJAX request and response but the server reaches the client through the Pusher application.
   >
   > However Pusher is different from AJAX in that PUSHER applies the event to everyone instead of just you!

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

   - Speciftying triggers on the model (when user comes in and goes out of the room)
   - note that the channel in the 'show' and 'index' is different! This is because only the people in the chatroom must see the message!

   ```ruby
   # Admission.rb
   class Admission < ApplicationRecord
       # This checks that paring of user_id and chat_room_id must be unique
       validates_uniqueness_of :user_id, :scope => "chat_room_id"
       belongs_to :user
       belongs_to :chat_room, counter_cache: true
       # if there is a new admission, automatically call for the user_joined_chat_room_notification
       after_commit :user_joined_chat_room_notification, on: :create
       after_commit :user_exited_chat_room_notification, on: :destroy
       # 유저가 챗방에 들어왔다는 사실을 알려줌
       def user_joined_chat_room_notification
           # Trigger Pusher to channel 'chat_room', with method 'join' and pass the necessary data as json
           Pusher.trigger('chat_room', 'join', {chat_room_id: self.chat_room_id, email: self.user.email}.as_json)
       end
       
       def user_exited_chat_room_notification
           Pusher.trigger("chat_room_#{chat_room_id}", 'exit', self.as_json.merge({email: self.user.email}))
           Pusher.trigger("chat_room", 'exit',  self.as_json)
       end
       
   end
   ```

   ```ruby
   # Chat_room.rb
   class ChatRoom < ApplicationRecord
       has_many :admissions
       has_many :users, through: :admissions
       has_many :chats
       
       # Trigger pusher after some effect in the model
       after_commit :create_chat_room_notification, on: :create
       after_commit :edit_chat_room_notification, on: :update
       after_commit :delete_chat_room_notification, on: :destroy
       
       def create_chat_room_notification
         Pusher.trigger("chat_room", 'create', self.as_json)
                     # (Channel_name, event_name, data) 
       end
       
       def edit_chat_room_notification
         Pusher.trigger("chat_room", 'update', self.as_json)
       end
       
       def delete_chat_room_notification
         Pusher.trigger("chat_room", "delete", self.as_json)
       end
       
       # 유저를 방에 넣기 방장을 방에 넣기!
       def user_admit_room(user)
         Admission.create(user_id: user.id, chat_room_id: self.id)
       end
       
       def user_exit_room(user)
         Admission.where(user_id: user.id, chat_room_id: self.id)[0].destroy
       end
       
   end
   ```

   - Subscribe through Javascript on the view pages

   ```html
   <% if user_signed_in? %>
     <%= current_user.email %> / <%= link_to 'Log out', destroy_user_session_path, method: :delete %>
   <% else %>
     <%= link_to 'Log in', new_user_session_path %>
   <% end %>
   
   <h1>Chat Rooms</h1>
   
   <table>
     <thead>
       <tr>
         <th>방제</th>
         <th>인원</th>
         <th>방장</th>
         <th colspan="3"></th>
       </tr>
     </thead>
   
     <tbody class="chat_room_list">
       <% @chat_rooms.reverse_each do |chat_room| %>
         <tr class="room<%= chat_room.id%>">
           <td><span class="title<%= chat_room.id%>"><%= chat_room.title %></span></td>
           <td><span class="current<%= chat_room.id %>"><%= chat_room.admissions.size %></span> 
           / <span class="max_count<%= chat_room.id%>"><%= chat_room.max_count %></span></td>
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
     // 방이 만들어졌을 때, 방에 대한 데이터를 받아서 방 목록에 추가
     function room_created(data) {
       $('.chat_room_list').prepend(`
         <tr class="room${data.id}">
           <td><span class="title${data.id}">${data.title}</span></td>
           <td><span class="current${data.id}">0</span> 
           / <span class="max_count${data.id}">${data.max_count}</span></td>
           <td>${data.master_id}</td>
           <td><a href="/chat_rooms/${data.id}">Show</a></td>
         </tr>`);
     };
     
     function delete_room(data) {
       $(`.room${data.id}`).remove();
     };
     
     function edit_room(data) {
       var title = $(`.title${data.id}`);
       var max_count = $(`.max_count${data.id}`);
       title.text(data.title);
       max_count.text(data.max_count);
     }
     
     function user_joined(data) {
       var current = $(`.current${data.chat_room_id}`);
       current.text(parseInt(current.text())+1);
     };
     function user_exited(data) {
       var current = $(`.current${data.chat_room_id}`);
       current.text(parseInt(current.text())-1);
     };
     
     // Create and authenticate to pusher
     var pusher = new Pusher('<%= ENV["pusher_key"]%>', {
         cluster: "<%= ENV["pusher_cluster"]%>",
         encrypted: true
       });
   
     // Create a channel specified to that chat_room
     var channel = pusher.subscribe('chat_room');
     
     // bind these functions to the channel so that pusher can trigger this for all users who is listening to the channel
     channel.bind('create', function(data) {
       console.log(data);
       room_created(data);
     });
     channel.bind('join', function(data) {
       console.log(data);
       user_joined(data);
     });
     channel.bind('update', function(data) {
       console.log(data);
       edit_room(data);
     });
     channel.bind('exit', function(data) {
       console.log(data);
       user_exited(data);
     });
     channel.bind('delete', function(data){
       console.log(data);
       delete_room(data);
     });
   });
   </script>
   ```

   ```html
   <p> 
   Now logged in as -  <%= current_user.email %> / 
   <%= link_to 'Log out', destroy_user_session_path, method: :delete %>
   </p>
   
   <h1><%= @chat_room.title %></h1>
   <!-- Printing out the list of current members -->
   <h3>Current Chatroom Members</h3>
   <div class="joined_user_list">
   <% @chat_room.users.each do |user| %>
       <p class="user-<%= user.id %>"><%= user.email %></p>
   <% end %>
   </div>
   <hr/>
   
   <!-- Load chatting messages -->
   <% if @chat_room.users.include?(current_user) %>
       <h4>Chat</h4>
       <div class="chat_list">
       <% @chat_room.chats.each do |chat| %>
           <p><%= chat.user.email %>: <%= chat.message %> <small>(<%= chat.created_at %>)</small></p>
       <% end %>
       </div>
       <% end %>
       <%= form_tag("/chat_rooms/#{@chat_room.id}/chat", remote: true) do %>
           <%= text_field_tag :message %>
   <% end %>
   <hr/>
   
   <!-- Chatroom menu bar -->
   <% if @chat_room.users.exclude?(current_user) %>
       <!-- the condition "remote:true" automatically becomes AJAX -->
       <span class="join_room">
       <%= link_to 'Join', join_chat_room_path(@chat_room), method: 'post', remote: true %> |
       </span>
   <% else %>
       <% if @chat_room.master_id.eql?(current_user.email) %>
       <%= link_to 'Delete', chat_room_path(@chat_room), method: 'DELETE', data: {confirm: "Delete this room?"} %> |
       <% else %>
       <%= link_to 'Exit', exit_chat_room_path(@chat_room), method: 'DELETE', remote: true, data: {confirm: "Exit this room?"} %> |
       <% end %>
   <% end %>
   <%= link_to 'Edit', edit_chat_room_path(@chat_room) %> |
   <%= link_to 'Back', chat_rooms_path %>
   
   <script>
   $(document).on('ready', function() {
       function user_joined(data) {
           $('.joined_user_list').append(`<p class="user-${data.user_id}">${data.email}</p>`)
           $('.chat_list').append(`<p>${data.email} entered this room</p>`);
       };
       function user_chat(data) {
           $('.chat_list').append(`<p>${data.email}: ${data.message} <small>(${data.created_at})</small></p>`)
       };
       function user_exit(data) {
           $(`.user-${data.user_id}`).remove();
           $('.chat_list').append(`<p>${data.email} exited this room</p>`);
       };
       function delete_room(data) {
           alert("Room has been deleted!");
           location.href = "root_path"
       }
       
       var pusher = new Pusher('<%= ENV["pusher_key"]%>', {
         cluster: "<%= ENV["pusher_cluster"]%>",
         encrypted: true
       });
   	// This is different from the index channel because it only applies to user who are currently in the room
       var channel = pusher.subscribe('chat_room_<%= @chat_room.id %>');
   
       channel.bind('join', function(data) {
           console.log(data);
           user_joined(data);
       });
       channel.bind('chat', function(data) {
           console.log(data);
           user_chat(data);
       });
       channel.bind('exit', function(data) {
           console.log(data);
           user_exit(data);
       });
   });
   </script>
   ```

#### Appendix - Code et cetera

```ruby
Rails.application.routes.draw do
  root 'chat_rooms#index'
  resources :chat_rooms do
    member do
      # Post method since we are putting the email on this member list
      post '/join' => 'chat_rooms#user_admit_room', as: 'join'
      post '/chat' => 'chat_rooms#chat'
      delete '/exit' => 'chat_rooms#user_exit_room'
    end
  end
  devise_for :users
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

```javascript
// user_admit_room.js.erb
alert("Just Joined!")
$('.join_room').remove(); //Remove the .join_room class element
```

```javascript
// user_exit_room.js.erb
alert("Room Exited");
location.reload(); //This reloads the page of the client!
```

```javascript
// chat.js.erb
console.log("채팅함~")
$('#message').val(''); // empties the message type box!
```

```ruby
# Chat_rooms_controller.rb
class ChatRoomsController < ApplicationController
  # add we need to find the chat_room to admit the user to!
  before_action :set_chat_room, only: [:show, :edit, :update, :destroy, :chat, :user_admit_room, :user_exit_room]
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
    # 현재 유저가 있는 방에서 Join을 눌렀을 때 동작하는 액션
    # 이미 조인한 유저라면 참가한 방입니다 alert
    if @chat_room.admissions_count < @chat_room.max_count
      if current_user.joined_room?(@chat_room)
          # Or another way is - chat_room.users.include?(current_user)
          # where function always returns an array object
        render js: "alert('you are already a member!');"
      else
      # 아닐 경우에는 참가
        @chat_room.user_admit_room(current_user)
      end
    end
  end
  
  # calls the chat_room model function 'user_exit_room' with param of current user
  def user_exit_room
    @chat_room.user_exit_room(current_user)
  end
  
  # create a chat message! this will trigger AFTER_COMMIT on the CHAT MODEL
  def chat
    @chat_room.chats.create(user_id: current_user.id, message: params[:message])
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
    # first allow the user to exit, then destroy all the admission connection between users and room and destroy the room
    @chat_room.user_exit_room(current_user)
    @chat_room.admissions.destroy_all
    @chat_room.destroy
    
    redirect_to root_path
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



