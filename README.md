# README

This README would normally document whatever steps are necessary to get the
application up and running.

Things you may want to cover:

* Ruby version
  rails 5.2.5
	ruby 2.6.6
	yarn 1.22.5
* System dependencies

* Configuration
	Creamos un scaffolds de kinds y uno de categorias haciendo referencia asi mismo:

	rails g scaffold Kind title

	rails g scaffold Category title is_public:boolean category:references

	hacemos las migraciones correspondientes 
	
	rails db:migrate
* Database creation


* Database initialization

* How to run the test suite

* Services (job queues, cache servers, search engines, etc.)

* Deployment instructions


 Inicialmente, nuestro sistema debe contener al menos 20 registros predefinidos. (punto 6)

agregamos la gema FAKER al gemfile gem 'faker' luego hacemos un require en el archivo seed.rb y agregamos los 20 registros

require 'faker'

# 20.times do |i|
#   title = Faker::Commerce.department + (i + 1).to_s
#   Kind.create!(title: title)
# end

# 5.times do |i|
#   title = Faker::Hipster.word + (i + 1).to_s
#   is_public = [true, false].sample
#   Seed = Category.create!(title: title, is_public: is_public)
#   10.times do |j|
#     title = Faker::Hipster.word + (j + 1).to_s
#     is_public = [true, false].sample
#     category_id = Seed.id
#     Category.create!(title: title, is_public: is_public, category_id: category_id)
#   end
# end
 
 nuestro seed.rb con todas la tablas referenciadas con los modelos 


categories = Category.all
kinds = Kind.all

10.times do |i|
  title = Faker::DcComics.title  + (i + 1).to_s
  url = Faker::Internet.url
  Bookmark.create!(title: title, url: url)
end

Bookmark.all.each do |b|
  3.times do 
    BookmarkCategory.create!(bookmark: b, category: categories.sample)
    BookmarkKind.create!(bookmark: b, kind: kinds.sample)
  end
end



Debemos poder acceder a las categorias padres e hijo desde el modelo category definimos la asociacion en el modelo category.rb

belongs_to :parent_category, class_name: "Category", optional: true, foreign_key: 'category_id'

  has_many :children_categories, class_name: "Category", foreign_key: "category_id"

	con esto podemos hacer la autorefrecia del category_id


	**

	Creamos un controlador home con método index y agregamos una barra de navegación por lo tanto bootstrap para nuestro proyecto 

	agremos la gema '' a nuestro gem file

	gem 'bootstrap', '~> 4.0.0'
	gem 'jquery-rails'

	luego en el manifiesto application.js

//= require jquery3
//= require popper
//= require bootstrap
//= require rails-ujs
//= require activestorage
//= require turbolinks
//= require_tree .

y luego el @import ''bootstrap; en el application.css (le cambiamos la extención .scss de ser necesario )

 *= require_tree .
 *= require_self
 */
@import "bootstrap";


creamos una carpeta shared para las vistas parciales

creamos una vista parcial _navbar.html.erb en shared y modificamos los botones a nuestra necesidad

	rails g controller home index
	hacemos nuestra ruta raiz

	root to: "home#index"

	Recuerda en la vista index y show de categories cambiar el <td> por category.parent_category

	**Haciendo validaciones:

	Validamos que category.rb tenga title y que kind.rb sea unico

	category.rb 
	validates :title, presence: true

	kind.rb
	validates :title, uniqueness: true

	vamos a la pagina con esto ya podemos crear padres y crear hijos

	Haciendo el modelo bookmark

	rails g model Bookmark title url 
	
	Aqui debemos pensar que necesitamos un modelo que nos una a los 2 una tabla intermedia  entre category y bookmark
	con su migración a final

	rails g migration CreateBookmarkCategories bookmark:references category:references 

	y agregmos las referencias como llaves foraneas para que se cree la relacion entre las tablas y pasen por esas llaves

	class CreateBookmarkCategories < ActiveRecord::Migration[5.2]
  def change
    create_table :bookmark_categories do |t|
      t.references :bookmark, foreign_key: true
      t.references :category, foreign_key: true

      t.timestamps
    end
  end
end

hacemos las realciones entre los modelos bookmark.rb, category.rb con nuestra tabla intermedia bookmark_category quedando asi cada modelo 

	en bookmark.rb

  has_many :bookmark_categories
  has_many :categories, through: :bookmark_categories
  

	en category.rb 

	belongs_to :parent_category, class_name: "Category", optional: true, foreign_key: 'category_id'
  has_many :children_categories, class_name: "Category", foreign_key: "category_id"
  
  has_many :bookmark_categories
  has_many :bookmarks, through: :bookmark_categories
  

  validates :title, presence: true

  def to_s
    title
  end

	ahora crearemos el controlador para bookmark y poder gestionar el CRUD con los 7 metodos

rails g controller bookmarks index show new create edit update destroy

(nos ayudaremos con el settings del controlador de categories_controller y lo modificamos para bookmarks_controller)

Haciendo la tabla intermedia entre bookmark y kind

rails g migration CreateBookmarkKinds bookmark:references kind:references 

al igual que la pasada la refrenciamos con bookmark.rb y kind.rb quedando el modelo asi:  

class Bookmark < ApplicationRecord
  has_many :bookmark_categories
  has_many :categories, through: :bookmark_categories
  
  has_many :bookmark_kinds
  has_many :kinds, through: :bookmark_kinds
  
end

class Kind < ApplicationRecord
  has_many :bookmarks, through: :bookmark_kinds
  has_many :bookmark_kinds

  validates :title, uniqueness: true

  def to_s
    title
  end
end

* ...
