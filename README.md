# CRUD with Controllers in Rails

CRUD stands for **Create, Read, Update, Delete**—the four basic operations for managing data in a web application. Rails makes it easy to implement these operations using controllers, models, and views.

---

## What is CRUD?

- **Create:** Add new records to the database (e.g., a new article or user)
- **Read:** Retrieve and display records (e.g., show a list of articles or a single article)
- **Update:** Modify existing records (e.g., edit an article)
- **Delete:** Remove records from the database (e.g., delete an article)

---

## How Rails Controllers Implement CRUD

In Rails, a typical resource (like `Article`) will have a controller with seven RESTful actions:

| Action   | HTTP Verb | Path                | Purpose                |
|----------|-----------|---------------------|------------------------|
| index    | GET       | /articles           | List all articles      |
| show     | GET       | /articles/:id       | Show a single article  |
| new      | GET       | /articles/new       | Form for new article   |
| create   | POST      | /articles           | Create a new article   |
| edit     | GET       | /articles/:id/edit  | Form to edit article   |
| update   | PATCH/PUT | /articles/:id       | Update an article      |
| destroy  | DELETE    | /articles/:id       | Delete an article      |

Rails can generate all of these for you with:

```sh
bin/rails generate scaffold Article title:string body:text
```

---

## Example: ArticlesController (Manual Implementation)

Below is a typical controller for managing articles, with explanations for each action:

```ruby
class ArticlesController < ApplicationController
  # GET /articles
  def index
    @articles = Article.all
    # Renders app/views/articles/index.html.erb
  end

  # GET /articles/:id
  def show
    @article = Article.find(params[:id])
    # Renders app/views/articles/show.html.erb
  end

  # GET /articles/new
  def new
    @article = Article.new
    # Renders app/views/articles/new.html.erb
  end

  # POST /articles
  def create
    @article = Article.new(article_params)
    if @article.save
      redirect_to @article, notice: "Article was successfully created."
    else
      render :new, status: :unprocessable_entity
    end
  end

  # GET /articles/:id/edit
  def edit
    @article = Article.find(params[:id])
    # Renders app/views/articles/edit.html.erb
  end

  # PATCH/PUT /articles/:id
  def update
    @article = Article.find(params[:id])
    if @article.update(article_params)
      redirect_to @article, notice: "Article was successfully updated."
    else
      render :edit, status: :unprocessable_entity
    end
  end

  # DELETE /articles/:id
  def destroy
    @article = Article.find(params[:id])
    @article.destroy
    redirect_to articles_path, notice: "Article was successfully deleted."
  end

  private

  def article_params
    params.require(:article).permit(:title, :body)
  end
end
```

---

## Example: Views for CRUD

**index.html.erb**
```erb
<h1>Articles</h1>
<%= link_to 'New Article', new_article_path %>
<ul>
  <% @articles.each do |article| %>
    <li>
      <%= link_to article.title, article_path(article) %>
      (<%= link_to 'Edit', edit_article_path(article) %> |
      <%= link_to 'Delete', article_path(article), method: :delete, data: { confirm: 'Are you sure?' } %>)
    </li>
  <% end %>
</ul>
```

**show.html.erb**
```erb
<h1><%= @article.title %></h1>
<p><%= @article.body %></p>
<%= link_to 'Edit', edit_article_path(@article) %> |
<%= link_to 'Back', articles_path %>
```

**new.html.erb & edit.html.erb**
```erb
<%= render 'form', article: @article %>
<%= link_to 'Back', articles_path %>
```

**_form.html.erb (Partial)**
```erb
<%= form_with(model: article) do |form| %>
  <% if article.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(article.errors.count, "error") %> prohibited this article from being saved:</h2>
      <ul>
        <% article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </div>
  <div>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </div>
  <div>
    <%= form.submit %>
  </div>
<% end %>
```

---


## Instance Variables in Views

Instance variables (those beginning with `@`) are the primary way controllers pass data to views in Rails. Any instance variable set in a controller action is accessible in the corresponding view template.

### How It Works

- In your controller action, set an instance variable:
  ```ruby
  def show
    @article = Article.find(params[:id])
  end
  ```
- In your view (`show.html.erb`), you can use `@article` directly:
  ```erb
  <h1><%= @article.title %></h1>
  <p><%= @article.body %></p>
  ```

### Example: Passing Collections

For listing resources, set an array or collection in the controller:
```ruby
def index
  @articles = Article.all
end
```
And use it in the view:
```erb
<ul>
  <% @articles.each do |article| %>
    <li><%= article.title %></li>
  <% end %>
</ul>
```

### Example: Passing Multiple Variables

You can set as many instance variables as you need:
```ruby
def dashboard
  @recent_articles = Article.order(created_at: :desc).limit(5)
  @user = current_user
end
```
And use them in the view:
```erb
<h2>Welcome, <%= @user.name %>!</h2>
<ul>
  <% @recent_articles.each do |article| %>
    <li><%= article.title %></li>
  <% end %>
</ul>
```

### Best Practices

- Only set instance variables you actually need in the view.
- Use descriptive names for clarity (`@article`, `@articles`, `@user`).
- Avoid setting instance variables in filters/callbacks unless necessary.
- Prefer using instance variables over global or class variables for passing data to views.

---

---

## More Tips

- Use `resources :articles` in `config/routes.rb` to generate all standard CRUD routes.
- Use strong parameters (`article_params`) to prevent mass-assignment vulnerabilities.
- Use partials (like `_form.html.erb`) to DRY up your views.
- Use flash messages (e.g., `notice:`) to give users feedback after actions.

---

This is how CRUD is implemented in Rails controllers, routes, and views!
