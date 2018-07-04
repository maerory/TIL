## Today I Learned (20180704)

#### How Browser interprets HTML (https://d2.naver.com/helloworld/59361)

Components of a Browser

1. User Interface - URL, Before/Next Button, Bookmark menu

2. Browser Engine - Controls the action between user interface and rendering engine

3. Rendering Engine - Presents the called contents, parses HTML and CSS

   Process of the rendering engine

   - HTML parsing for creating DOM tree
   - Creates/places/draws the render tree

4. Connection - Used for Network calls like HTTP request. This is an independent interface

5. UI Backend - Draws the combo box or window objects

6. JS interpreter

7. File storage



#### More Web JavaScript

1. Use of $ sign in selecting elements in the web, we can apply action to all the elements in the web using this method

   - `$("css selector")` : returns all matching elements, put # in front of a title. (EX. ".btn", ".#title" - '.' refers to the class, '#' refers to the ID)

   - You can add events to the element using the condition function or the on function to specify the condition

     ```javascript
     $('.btn').mouseover(function() {
         alert("Hello World") //applies to all btn elements
     }); 
     $('.btn').on('mouseover', function() {
         alert("Hello World")
     });
     ```

     ```javascript
     // You can put multiple condition using 'on' function
     $('.btn').on('mouseenter mouseout', function() {
       if ($(this).hasClass('btn-danger')) { //'this' refers to the selected element only
         $(this).removeClass('btn-danger').addClass('btn-primary');
       } else {
         $(this).removeClass('btn-primary').addClass('btn-danger');
       }
     });
     
     $('.btn').on('mouseenter mouseout', function() {
         // toggleClass adds or removes in the opposite way
         $(this).toggleClass('btn-danger').toggleClass('btn-primary');
     });
     ```

   - Difference between `(this)` and `$(this)` : `$(this)` is the jquery object, you can apply jquery functions not DOM methods. `(this)` is just DOM object and you cannot apply jquery function to it.

2. Changing the attribute of the web element

   ```javascript
   $('.btn').on('mouseover', function() {
     $('img').attr('style', 'width: 100px;'); //$('img') selects the image that the btn is belonged to 
   });
   ```

   ```javascript
   $('.btn').on('mouseover', function() {
       $('.card-title').text("Don't Touch Me!"); // 'text' functions selects all the text inside the element 
   });
   ```

   There is also `.css` function that changes the css elements

3. Finding close-by elements using 'find' / 'sibiling' / 'parent'

   - Find searches from the elements thats belonged to the current element

   ```javascript
   $(this).parent().find('.card-title').text("Don't Touch");
   ```

#### Creating a simple translator using Hangul.js Library

1. In ordrer to use the library, you must create and save the file hangul.js in the same directory and put `<script src="hangul.js" type="text/javascript"></script>` in the code

2. `split(), join()` : these functions seperate the string into words and combine

3. We create function translated using the above helpers:

   ```javascript
   function translate(str) {
     return str.split('').map(function(el) { //Each split str is passed in as 'el' variable
         var d = Hangul.disassemble(el);
         if (d[3] && Hangul.isVowel(d[1]) && Hangul.isVowel(d[2])) { //isVowel checks whether the hangul is a vowel
           var tmp = d[2];
           d[2] = d[3];
           d[3] = tmp;
         }
         return Hangul.assemble(d);
     }).join('');
   };
   ```

   To utilize this function, put `<script src="translate.js"></script>` in the body. Now we utilize that translated function in a script tag. After creating a button with `class=translate` we can access that element with `$('.translate')`.

   ```javascript
   <script type="text/javascript">
       $('.translate').on('click', function() {
         var input = $('#input').val()
         var result = translate(input);
         $('h3').text(result);
         console.log(result);
       });
     </script>
   ```

   

#### Introduction to AJAX

- Usually the server request and responds with HTML, but with AJAX you request in javascript and get the response in javascript
- You need to know the route(url) and the method(GET, POST...) in order for the request
- The advantage of AJAX is you can request to server without changing the view
- ![1530683513305](C:\Users\student\AppData\Local\Temp\1530683513305.png)

```javascript
$.ajax({
    url: //to send the request to
    method: //the method of the request
    data: {
        k: v //key value pair to send, can be accessed with params[k] => v
    }
});
```

#### Creating Likes Button with AJAX in Rails

- When creating a likes data model, there might be conflicts with the existing relationships between the posts (ex. users and posts have 1:n), thus we have to choose one to maintain and supplement the other relationship in some other way (ex. just saving the raw id number and finding it later)

- Likes button logic:

  1. When pressed, send request to server - (request that current user likes the current post)

     ```javascript
     <script>
     $(document).on('ready', function () {
         $('.like').on('click', function() {
             console.log("Like!");
             $.ajax({ //ajax request to the url, so you have to route
                //thus, you also have to set the route and attach a method to that route
                url: '/likes/<%= @movie.id %>' 
             });
         });
     })
     </script>
     ```

  2. Server action: Create a like method in the controller and creates a like object that combines the user_id and post_id

     ```ruby
     def like_movie
         # Select the first element of found like (since it will return an array)
         @like = Like.where(user_id: current_user.id, movie_id: params[:movie_id]).first
         if @like.nil?
     		#현재 유저와 params에 담긴 movie 간의 좋아요 관계를 설정
         	# Create function combines both new and save
             @like = Like.create(user_id: current_user.id, movie_id: params[:movie_id])
         else
             # 현 유저가 이미 좋아요를 눌렀을 경우, 해당 Like 인스턴스 삭제
             # 새로 눌렀을 때만 좋아요 관계 설정
             @like.destroy
         end
     end
     ```

  3. Server response: change the 'like' to 'unlike' and `btn-info -> btn-warning text-white` 

     ```javascript
     // LIKE_MOVIE.JS.ERB (rails automatically calls for this file when there is an ajax call with that same name)
     alert("LIKEY!");
     // The Frozen method checks whether the @like instance is alive!
     if(<%= @like.frozen? %>) {
         $('.like').text("좋아요").toggleClass("btn-info btn-danger");
     } else {
         $('.like').text("좋아요 취소").toggleClass("btn-danger btn-info");
     }
     ```

     