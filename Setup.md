# Setup Inicial
## DEVISE
- Agregar e inicializar Devise
  * bundle add devise
  * rails g devise:install
  * rails g devise user
  * rails g migration addDetailsToUser name phone
  * rails g devise:controllers users

- Modificar `app/controllers/application_controller.rb`
```ruby
class ApplicationController < ActionController::Base
  # Only allow modern browsers supporting webp images, web push, badges, import maps, CSS nesting, and CSS :has.
  allow_browser versions: :modern

  before_action :configure_permitted_parameters, if: :devise_controller?
  
  def authorize_request_for_admin
    unless current_user.email == 'kari@gmail.com'
      redirect_to photos_path, notice: "No estas autorizado para modificar las fotos"
    end
  end

  protected
  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name, :photo])
    devise_parameter_sanitizer.permit(:account_update, keys: [:name, :photo])
  end
end
```
- Modificamos archivos app/views/registrations/new.html.erb y edit.html.erb
```html
<h2>Sign up</h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= render "devise/shared/error_messages", resource: resource %>

  <div class="field">
    <%= f.label :name %><br />
    <%= f.text_field :name, autofocus: true, autocomplete: "Nombre:", required: true %>
  </div>
  
  <div class="field">
    <%= f.label :avatar %><br />
    <%= f.url_field :avatar, autofocus: true, autocomplete: "Avatar:", required: true %>
  </div>
  
  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email", required: true %>
  </div>

  <div class="field">
    <%= f.label :password %>
    <% if @minimum_password_length %>
    <em>(<%= @minimum_password_length %> characters minimum)</em>
    <% end %><br />
    <%= f.password_field :password, autocomplete: "new-password" %>
  </div>

  <div class="field">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation, autocomplete: "new-password" %>
  </div>

  <div class="actions">
    <%= f.submit "Sign up", class: 'btn btn-success' %>
  </div>
<% end %>

<%= render "devise/shared/links" %>
```
- Descomentamos la siguiente línea en config/initializers/devise.rb
  * config.navigational_formats = ['*/*', :html, :turbo_stream]

## ActiveStorage
- Inicializamos activestorage
  * bundle add activestorage
  * rails active_storage:install

- Creamos un modelo con archivo adjunto
```ruby
class Photo < ApplicationRecord
  has_one_attached :image

  has_many :comments, dependent: :destroy
  has_many :reactions, dependent: :destroy
end
```
- Se modifica el formulario del modelo
```html.erb
<%= form_with(model: photo) do |form| %>
  <% if photo.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(photo.errors.count, "error") %> prohibited this photo from being saved:</h2>

      <ul>
        <% photo.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
  ...
  <div>
    <%= form.label :image, style: "display: block" %>
    <%= form.file_field :image, required: true %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```
- Se Modifica el controlador correspondiente
```ruby
class PhotosController < ApplicationController
  before_action :set_photo, only: %i[ show edit update destroy ]
  before_action :authenticate_user!, except: [:index, :show]
  before_action only: [:new, :create, :edit, :update, :destroy] do
    authorize_request_for_admin
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_photo
      @photo = Photo.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def photo_params
      params.require(:photo).permit(:name, :content, :image)
    end
end
```
## Figaro
- Instala y configura *figaro*
  * bundle add figaro
  * bundle exec figaro install
- Crear archivo en `config/application.yml`, y agregarlo a `.gitignore`
- Se modifican archivos `config/enviroments/development.rb` y `production.rb`
  * config.active_storage.service = :amazon
- Modificar configuraciones de `config/storage.yml`
```yml
amazon:
  service: S3
  access_key_id: <%= ENV['aws_access_key_id'] %>
  secret_access_key: <%= ENV['aws_secret_access_key'] %>
  region: us-west-1
  bucket: nombre-del-bucket
```
- Instala gema de S3
  * bundle add aws-sdk-s3


