20180709 Today I Learned

Applying Webpage Template (http://guides.rubyonrails.org/asset_pipeline.html)

Although this process might seem tedious, restructuring improves the loading speed

Implementation steps: 

1. Install bootstrap according to the template version (usually 3 or 4) through Gemfile (register other gemfiles as well)
2. Copy and paste all required css files into vendor/assets/stylesheets and record them on home.scss with @import '<CSS NAME>' (Delete original imports) 
   - (vendor files are like libraries, they won't change. On the other hand, all the modifiable files go under app/assets/stylesheet) (Also change the file extension to application.scss)
   - Change the coffeescripts to js extensions
3. JS files : in app/javascripts/channels/home.js add in all the js file like so - //= require <JS name> and copy all JS file to javascripts folder
   - CSS files: copy all the CSS files in the template to vendor/assets/styleshees and import it from app/assets/stylesheets/<CONTROLLER>.SCSS
4. Copy the viewport tag and <%= stylesheet_link_tag    params[:controller], media: 'all' %>to layout/application.html.js
5. To store data in cache with rake assets:precompile, add 
       Rails.application.config.assets.precompile += %w( home.js 
                                                         home.scss)
       # Or other js, scss files that you will use
   to config/initializers/assets.rb, then you will be able to see the error when precompiling
   - rake assets:clobber & rake assets:precompile : stores major portion of the asset files into computer cache. 
   - You can check any errors in js or scss files ahead with precompiling
6. Splitting the layout - templates are usually composed of big <div> sections. Create  views/layouts/shared and copy and paste the <div class="navbar"> element into a _<navbar>.hml.erb file (footers and other navbars or menu should also be splitted)
   - Rendering the split parts - <%= render 'shared/<NAVBAR>' %>like so in the application.html file (keep it in the same order as the template)
7. render - render finds the segment through the controller function's name. But you can specify the content part using content_for
   - Load the js file in the template : add <%= yield 'javascript_content' %> in the application.html.erb
   - Add <%= javascript_include_tag params[:controller] %> in the index.html.erb, this allows for implementing different js files for different controllers
8. Images: to implement images, paste the templates image contents to /app/assets/images and go through the layout htmls, changing all images path one by one into a ruby syntax.

    src="<%= asset_path 'users/avatar-3.jpg' %>" 

- The asset_path route is to the images folder (http://guides.rubyonrails.org/asset_pipeline.html)

1. Fonts: Copy the fonts file into public/fonts directory (for font-awesome in rails, you have to implement it via gemfile and application.scss)


