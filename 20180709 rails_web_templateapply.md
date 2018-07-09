## 20180709 Today I Learned

### Applying Webpage Template

###### Although this process might seem tedious, restructuring improves the loading speed

Implementation steps: 

1. Install bootstrap according to the template version (usually 3 or 4) through Gemfile

2. Copy and paste all required css files into `vendor/assets/stylesheets` and record them on `home.scss` with `@import '<CSS NAME>'`

3. Copy the viewport tag and ` <%= stylesheet_link_tag    params[:controller], media: 'all' %>`to layout/application.html.js

4. To debug with `rake assets:precompile`, add 

   ```ruby
   Rails.application.config.assets.precompile += %w( home.js 
                                                     home.scss)
   ```

   to `config/initializers/assets.rb`, then you will be able to see the error when precompiling

5. Splitting the layout - templates are usually composed of big <div> sections. Create a shared folder under the `views/layouts` and copy and paste the <div class="navbar"> element into a `_<navbar>.hml.erb` file. 

6.  Rendering the split parts - `<%= render 'shared/<NAVBAR>' %>`like so in the application.html file (keep it in the same order as the template)

7.  render - render finds the segment through the controller function's name. But you can specify the content part using `content_for`

   - Load the js file in the template : add `<%= yield 'javascript_content' %>` in the application.html.erb
   - Add `<%= javascript_include_tag params[:controller] %>` in the index.html.erb

8. JS files : in `app/javascripts/channels/home.js` add in all the js file like so - `//= require <JS name>` and copy all JS file to javascripts folder

   1. CSS files: copy all the CSS files in the template to `vendor/assets/styleshees` and import it from `app/assets/stylesheets/<CONTROLLER>.SCSS`

9. Images: to implement images, paste the templates image contents to `/app/assets/images` and go through the layout htmls, changing all images path one by one into a ruby syntax.

   ```html
   src="<%= asset_path 'users/avatar-3.jpg' %>" 
   ```

   The asset_path route is to the images folder

   (http://guides.rubyonrails.org/asset_pipeline.html)

10. Fonts: Copy the fonts file into `public/fonts` directory (for font-awesome in rails, you have to implement it via gemfile and application.scss)

11. `rake assets:clobber` & `rake assets:precompile` : stores major portion of the asset files into computer cache. 

    