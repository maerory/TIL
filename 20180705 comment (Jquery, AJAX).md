## 20180705 Today I Learned

#### Javascript String Interpolation

1. `string text ${expression in JS} string ` : surround with apostrophe!
2. Difference between id and class : id is unique while class is not!

### Creating Comment Section (JS, JQuery, AJAX)

- Develop each logical step to complete an action
- Create 
  1. Jquery - create jquery function to set the event and event-listener
  2.  AJAX request - AJAX request allows us to update without reloading the page
  3. Routing - create a route for the AJAX request URL
  4. Controller Function - we also need a controller function for the route
  5. js.erb file - .js.erb file contains the response in javascript language that is executed after AJAX request

#### 1. Inserting Comment Section (Implemented with AJAX)

Remember to put the script commands inside `$(document).on('ready', function() {})`

1. When form is submitted, fetch the input inside
2. AJAX request to the server with url: '/create/comment' with movie_id
3. Save in server and create js.erb for server's response
4. js.erb - put the comment in the comment box

#### 2. Deleting Comments

1. Press the Delete button (Element) to initiate (Event Listener)
2. Disappears from the screen (Event Handler)
3. Deleted from the DB (AJAX)

#### 3. Editing Comments

1. Press the Edit Button
2. The comment box turns to text-edit-box, and the Edit button changes to Confirm box
3. After pressing the confirm button, the original comment changes to the contents of the textbox, edit button returns to normal
4. If there is an element with "update-comment" class, stop other buttons' event-handler
5. AJAX server request with the new comment text, server finds the comment and updates it

#### Code

###### Show.html.erb

```html
<form class="text-right comment">
    <!--We add our comment-contents class to access this element-->
    <input class="form-control comment-contents"><br/> 
    <input type="submit" value="댓글쓰기" class="btn btn-success">
</form><br/>
<h4>Comments</h4>
<ul class="list-group comment-list">
<!--Iterate through movie array to print the contents and buttons-->
<% @movie.comments.reverse_each do |comment| %>
    <!--Iterate through movie array to print the contents and buttons-->
    <!-- Pass on comment.id on ruby tag to specify class for each element -->
    <li class="comment-<%= comment.id %> list-group-item d-flex justify-content-between">
        <span class="comment-detail-<%= comment.id %>"><%= comment.contents %></span>
        <div>
            <!-- attribute 'data-id' allows us to use the .data method later -->
            <button data-id="<%= comment.id %>" class="btn btn-warning text-white edit-comment">수정</button>
            <button data-id="<%= comment.id %>" class="btn btn-danger destroy-comment">삭제</button>    
        </div>
    </li>
<% end %>
</ul>
```

```javascript
//This is the <Script> Tag inside "show.html.erb"
$('.comment').on('submit', function(e) {
    //e.preventDefault stops the page reload after jquery method is executed
    e.preventDefault();
    var comm = $('.comment-contents').val() //val fetches the contents inside input tags, save this inside a JS variable and send it via AJAX request
    console.log(comm)

    $.ajax({ //Ajax request takes url, method, and data
        url: "/movies/<%= @movie.id %>/comments",
        method: "POST",
        data: {
            contents: comm
        }
    });
});
// This way of event listening allows for listening on any part of document with that specific class, id,or attrs
$(document).on('click', '.destroy-comment', function(e) {
    console.log("Deleted");
    var comment_id = $(this).attr('data-id');
    console.log(comment_id);
    $.ajax({
        url: "/movies/comments/" + comment_id,
        method: "DELETE"
    });
});

$(document).on('click', '.edit-comment', function() {
    //If another update-comment element is active, end this jquery
    //Compare it to the length since HTML collection object is not FALSE despite being empty
    if (document.getElementsByClassName("btn update-comment").length != 0) {
        alert("한 번에 하나만 수정 가능")
        return;
    }
    // .data function to fetch attribute that starts with data
    var comment_id = $(this).data("id");
    // Note that apostrphe tag is used in JS string interpolation
    var edit_comment = $(`.comment-detail-${comment_id}`);
    // Trim the contents before since it might break the text
    var contents = edit_comment.text().trim();
    // .html method allows for inputing any HTML language to that element, this transforms the edit_comment into a text input box
    edit_comment.html(`<input type="text" value="${contents}"" class="form-control edit-comment-${comment_id}">`);
    // Change the button to a confirm button by removing and adding classes. Note that update-comment class is important because we have to use this class to check whether a user is editing a comment right now
    $(this).text("확인").removeClass("edit-comment btn-warning").addClass("update-comment btn-dark");
});

$(document).on('click', '.update-comment', function() {
    var comment_id = $(this).data("id");
    var comment_form = $(`.edit-comment-${comment_id}`);
    $.ajax({
        url: "/movies/comments/" + comment_id,
        method: "PATCH",
        data: {
            contents: comment_form.val()
        }
    });
});
```

###### movies_controller.rb

````ruby
## These methods are routed to the url of AJAX requests
def create_comment
    # Two different ways of creating the comment
    @comment = Comment.create(user_id: current_user.id, movie_id: @movie.id, contents: params[:contents])
    #@movie.comments.new(user_id: current_user.id, movie_id:@movie.id, contents: params[:contents]).save
  end

  def destroy_comment
    # You can destroy and save the object at the same time
    @comment = Comment.find(params[:comment_id]).destroy
  end

  def update_comment
    # Find the element first and update for single selection query
    @comment = Comment.find(params[:comment_id])
    @comment.update(contents: params[:contents])
  end
````

###### routes.rb

````ruby
resources :movies do
    # member routing: prefix of the url is "/model_name/:model_id"
    member do
        post '/comments' => 'movies#create_comment' #/movies/:movie_id/comments
    end
    # collection routing: prefix of the url is "/model_name"
    collection do
        delete '/comments/:comment_id' => 'movies#destroy_comment' # /movies/comments/:comment_id
        patch '/comments/:comment_id' => 'movies#update_comment'
    end
````

###### .JS.ERB VIEWS

````javascript
// create_comments.js.erb
alert('댓글 등록')
// Add the latest comment to the top
$('.comment-list').prepend(`
<li class="comment-<%= @comment.id %> list-group-item d-flex justify-content-between">
    <span class="comment-detail-<%= @comment.id %>"><%= @comment.contents %></span>
    <div>
        <button data-id="<%= @comment.id %>" class="btn btn-warning text-white edit-comment">수정</button>
        <button data-id="<%= @comment.id %>" class="btn btn-danger destroy-comment">삭제</button>    
    </div>
</li>`);
// Empty the comment box
$('.comment-contents').val('');
    
// update_comment.js.erb
alert("수정 완료")
// Find the comment element
var edit_comment = $('.comment-detail-<%= @comment.id %>');
// Set the element with our updated contents
edit_comment.html('<%= @comment.contents %>');
// Change the button
$('.update-comment').text("수정").removeClass("update-comment btn-dark").addClass("edit-comment btn-warning");

// destroy-contents
alert("댓글 삭제")
// Remove function remove the element
$(".comment-<%= @comment.id %>").remove();
````

