# 20180706 Today I Learned

### Creating a Search Bar

1. Create a search bar element first

   ```html
   <input type="text" class = "form-control movie-title"><br/>
   <!-- Show the result of the search in element below -->
   <div class="recomm-movie row d-flex justify-content-start"></div>
   ```

2. Assign a AJAX request on the search bar

   ```javascript
   $(document).on('ready', function(){
     $(`.movie-title`).on('keyup', function() {
       var title = $(this).val();
       $.ajax({
         url: '/movies/search_movie',
         data: {
           q: title
         }
       })
     });
   });
   ```

3. Route the "movies/search_movie" in the route file

   ```ruby
   resources :movies do
       member do
         post '/comments' => 'movies#create_comment'
       end
       collection do
        delete '/comments/:comment_id' => 'movies#destroy_comment'
        patch '/comments/:comment_id' => 'movies#update_comment'
        get '/search_movie' => 'movies#search_movie' #This is the link for the AJAX request
       end
   ```

4. Create function for the route 'search_movie'

   ```ruby
   def search_movie
       respond_to do |format|
         # Check for the empty search bar to clear the search result section
         if params[:q].strip.empty?
           format.js {render 'no_content'}
         else
           # LIKE sql function to find the similar name, and % after the param allows for any suffixes
           @movies = Movie.where("title LIKE ?", "#{params[:q]}%")
           format.js {render 'search_movie'}
         end
       end
     end
   ```

5. Create JS views for each result

   - no-content.js.erb : empties the search result field

   ```javascript
   $('.recomm-movie').html(`
       
   `);
   ```

   - search_movie.js.erb : execute the search

   ```
   $('.recomm-movie').html(`
       <% @movies.each do |movie| %>
           <span class="badge badge-dark"><%=movie.title%></span>&nbsp;
       <% end %>
   `);
   ```

   

### Pagination for Rails Webpage

- Use Kaminari!