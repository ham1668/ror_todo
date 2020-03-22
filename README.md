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

rails db:migrate

After db migraion is completed, run the server and checkout this new endpoint:
localhost:3000/todo_lists/

User is presented with a basic page that shows Todo Lists...and user can add a new Todo list too!

