# ROR Todo tutorial

Following along the directions from https://medium.com/@deallen7/how-to-build-a-todo-app-in-rails-e6571fcccac3 to learn Ruby on Rails

Project is created using the command: rails new todo

Starting up server is done with the following command: rails server
NOTE: You can point a browser to localhost:3000 to check it out

Scaffolding is created when running the following commands from within the ./todo directory:

rails generate scaffold todo_list title:string description:text
or
rails g scaffold todo_list title:string description:text

NOTE: Some rails commands are shortened to single character
ANOTHER NOTE: Scaffolding is like a shortcut...it will automatically generate model, view, controller for use with a single table.
YET ANOTHER NOTE: Ruby on Rails follows conventions. One of these is to pluralize names of database tables. For example, I might use scaffold to describe an employee...so the resulting database table would make sense to be named employees. That's the gist from this page: http://underpop.online.fr/r/ruby/rails/tutorial/ruby-on-rails-2-6.html

Since the scaffold command setup a db migration, I need to actually run the db migration command:

rake db:migrate

After db migraion is completed, run the server and checkout this new endpoint:
localhost:3000/todo_lists/

User is presented with a basic page that shows Todo Lists...and user can add a new Todo list too!

In the config/routes.rb file, I change the root route:

root "todo_lists#index"

Now, the localhost:3000/ url will go to a Todo Lists index page which can be found in the app/view/todo_lists directory

I generate todo_item model using this command:

rails generate model todo_item content:string todo_list:references
or
rails g model todo_item content:string todo_list:references

This is building a model that will store todo items which will be related to todo lists.

Afterwards, I run the db:migrate command:

rake db:migrate

These migration files can be found in the db/migrate directory.
The model files can be found in the app/models directory.

Now...to describe the association of todo_item and todo_list...
In the model for todo_list.rb

has_many :todo_items

Let's setup routes! In the config/routes.rb...

  resources :todo_lists do
  	resources :todo_items
  end

To inspect routes after making this configuration change:

rake routes

With the model created for todo_items, it is time to create the controller with this command:

rails generate controller todo_items
or
rails g controller todo_items

You can find the controller in the app/controller directory. It will need a bunch of methods added to it...

******
  before_action :set_todo_list
  def create
    @todo_item = @todo_list.todo_items.create(todo_item_params)
    redirect_to @todo_list
  end

  private

  def set_todo_list
    @todo_list = TodoList.find(params[:todo_list_id])
  end

  def todo_item_params
    params[:todo_item].permit(:content)
  end
******

We've added an action for creating new todo's. Now we need a form that will let us collect this data. We need a new view! The tutorial calls for creation of two "partials" in the views/todo_items folder:

_todo_item.html.erb
_form.html.erb

According to google, partials in RoR allow you to easily organize and reuse view code in Rails application. Partial filenames typically start with _ character and end in the same .html.erb extension as your views.

Adding to the _form.html.erb file:

<%= form_for([@todo_list, @todo_list.todo_items.build]) do |f| %>
 <%= f.text_field :content, placeholder: "New Todo" %>
 <%= f.submit %>
<% end %>

Adding to the _todo_item.html.erb file:
<p><%= todo_item.content %></p>

To get these to show up in the todo_lists page, I need to modify the views/todo_lists files:

show.html.erb - After description and before the edit blocks

<div id="todo_items_wrapper">
 <%= render @todo_list.todo_items %>
 <div id="form">
  <%= render "todo_items/form" %>
 </div>
</div>

Start up rails server...

Should be able to visit your todo list page, show the items...and begin adding todo items!
