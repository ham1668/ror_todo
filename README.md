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

Time to add DELETE functionality to todo items...Update the _todo_item.html.erb file to this:

<p><%= todo_item.content %></p>
<%= link_to todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>

The lists form will now have endpoints for each todo_list item displayed. Let's change this so it will show "Delete" instead.

That endpoint path is revealed because of this:
  link_to todo_list_todo_item_path(@todo_list, todo_item.id)

Change to this:
  link_to "Delete", link_to todo_list_todo_item_path(@todo_list, todo_item.id)

The _todo_item.html.erb should look like this:

<p><%= todo_item.content %></p>
<%= link_to "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>

At this point, the todo item form is showing the "Delete" hyperlink and it looks good now. Clicking the "Delete" button will pop up that confirmation message...and if you confirm, it will result in error.

The action destroy could not be found for TodoItemsController...so we have to go there and fix that.

Go to the todo_items_controller.rb file in app/controllers and add the destroy method:

def destroy
 @todo_item = @todo_list.todo_items.find(params[:id])
 if @todo_item.destroy
  flash[:success] = "Todo List item was deleted."
 else
  flash[:error] = "Todo List item could not be deleted."
 end
 redirect_to @todo_list 
end

Now we can delete todo items!

The next thing to implement will be a way to mark a todo item as completed. I need to add a new column to the database:

rails generate migration add_completed_at_to_todo_items completed_at:datetime
or
rails g migration add_completed_at_to_todo_items completed_at:datetime

This creates a new migration in the db/migration directory.
It knows that this is an extra column to be added to the todo_items table.

HOW?

Seems like rails convention is to create a migration name that is literally what is to be done.

In this case, the "add_completed_at_to_todo_items" migration name literally means..."add" a column named "completed_to" to the "todo_items" table. The singular column is described in the command...so that's what is used in the migration.

More columns can be added too! The name can be slimmed down to just "add_columns_to_todo_items" for example...and specify the various columns afterwards...OR...you can open the migration file before it has been applied, and add the columns there. This "rails generate migration" command is just to generate the migration template file for you...and if it is easier to just modify that resulting file...at least it is there for you.

Run the rake db:migration to get this column added. The original migration that created the "todo_items" table cannot be modified at this point because it has already been run.

With the new field added to the todo_items table, now it is time to add a route. In the app/config/routes.rb file, modify the :todo_lists resources block:

  resources :todo_lists do
  	resources :todo_items do
  	  member do
  	  	patch :complete
  	  end
  	end
  end

A link needs to be added to the end of the _todo_item.html.erb partial. It would look like this:

<%= link_to "Mark as Complete", complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch %>

So we've handled the data model to have this completed_at column. The view now has a "Mark as Completed" link for each todo item. The last bit is to modify the todo_items_controller.rb.

The tutorial calls out a private method, a before action, and a complete method.

The private method looks like this:

def set_todo_item
  @todo_item = @todo_list.todo_items.find(params[:id])
end

The before_action bit looks like this:

before_action :set_todo_item, except: [:create]

The complete method part looks like this:

def complete
  @todo_item.update_attribute(:completed_at, Time.now)
  redirect_to @todo_list, notice: "Todo item completed"
end

At this point, the author has us test the "Mark as Complete" link. Not much happens besides the text at the top of the page proclaiming "Todo item completed".

So now, we're going to work on something else happening on the view. The _todo_item.html.erb file will be modified.

A conditional is added so that if the todo item is completed...that it will be displayed with strikethrough.

<div class="row clearfix">
 <% if todo_item.completed? %>
  <div class="complete">
   <%= link_to "Mark as Complete", complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch %>
  </div>
  <div class="todo_item">
   <p style="opacity: 0.4;"><strike><%= todo_item.content %></strike></p>
  </div>
  <div class="trash">
   <%= link_to "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>
  </div>
 <% else %>
  <div class="complete">
   <%= link_to "Mark as Complete", complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch %>
  </div>
  <div class="todo_item">
   <p><%= todo_item.content %></p>
  </div>
  <div class="trash">
   <%= link_to "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>
  </div>
 <% end %>
</div>

Next, the todo_item.rb model will be modified to define this ".completed?" method mentioned in the if statement above.

  def completed?
  	!completed_at.blank?
  end

In the model, we're saying that .completed? will return boolean of the completed_at field is not blank. If it is not blank, True is returned. Otherwise, False is returned.

Now when I refresh my list of todo items, I can see that "Watch a video" is struck through!

At this point, the tutorial focuses on styling the look and feel...

The app/assets/stylesheets/application.css file is renamed to application.css.scss.

Lots is copied from https://github.com/mackenziechild/Todo-App/blob/master/app/assets/stylesheets/application.css.scss

