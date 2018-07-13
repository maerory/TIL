## 20180712 Today I Learned

### 1. Using daum map API! (http://apis.map.daum.net/)

1. Register and create an application at KAKAO developers
2. Add the project main url to the application, find the API keys for different languages
3. Follow the DAUM MAP API document to implement maps in your application

### 2. SummerNote! (https://github.com/summernote/summernote-rails)

1. Add gem and JS files to your workspace. (We can create js instead of coffee, translate at (www.js2.coffee))

   ```javascript
   // summernote-init.js
   $(document).on('ready', function() {
     $('[data-provider="summernote"]').each(function() {
       $(this).summernote({
         height: 300
       });
     });
   });
   ```

2. Add the data-provider option in your form

   ```html
   <!-- _form.html.erb -->
   <div class="form-group">
       <%= f.label :description %>
       <%= f.text_area :description, 'data-provider': :summernote %>
   </div>
   ```

   

### 3. Image uploading with Summernote

1. Create both the image model and the uploader (`rails g uploder summernoteUploader`) for summernote

2. Mount uploader at image.rb with `mount_uploader :image_path, SummernoteUploader`

3. Modify summernote-init.js to accept the uploaded image to the link

   ```javascript
   //summernote-init.js
   $(document).on('ready', function() {
     function sendFile(file, toSummernote) {
       var data = new FormData;
       data.append('upload[image]', file);
       $.ajax({
         data: data,
         type: 'POST',
         url: '/uploads',
         cache: false,
         contentType: false,
         processData: false,
         success: function(data) {
           console.log('file uploading...');
           // change to data.image_path.url because the movie model saves to 'image_path'
           return toSummernote.summernote("insertImage", data.image_path.url);
         }
       });
     }
     
     $('[data-provider="summernote"]').each(function() {
       $(this).summernote({
         height: 300,
         callbacks: {
           onImageUpload: function(files) {
             return sendFile(files[0], $(this));
           }
         }
       });
     });
   });
   ```

4. Create upload_image function for movie_controller

   ```ruby
   # Part of movies_controller.rb
   def upload_image
       @image = Image.create(image_path: params[:upload][:image])
       render json: @image
   end
   ```

5. Change the <p> to `<p><%= simple_format(@movie.description) %></p>` to allow browser to read the image link



### 4. Using Rails_Admin (https://github.com/sferik/rails_admin)

> Rails admin is a built-in rails application similar to rais_db that helps admin manage the site.

