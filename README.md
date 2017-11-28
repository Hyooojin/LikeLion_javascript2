# rails CRUD & jQuery 


```ruby
$ rails g devise:install
$ rails g scaffold post title:string content:text
$ rake db:migrate

$ rails g devise user
$ rails g model comments post:references body:text
$ rake db:migrate
```

* model관의 관계는 post랑 comment

```ruby
# post.rb
has_many :comments

# comment.rb
belongs_to :post
```

* route 설정

```ruby
# routes.rb

root 'posts#index'

```

* gemfile 설정

```ruby
# gem file
gem 'devise'
gem 'faker'
gem 'kaminari'
gem 'bootstrap-sass'
```

* bootstrap 설정

```ruby
# application.scss (css -> scss)
@import 'bootstrap';
```

```ruby
# views/application.erb

  <% if user_signed_in? %>
    <%=link_to "SIGN OUT", destroy_user_session_path, method: :delete, data: {confirm: "로그아웃 하시겠습니까?"}%>
  <% else %>
    <%=link_to "SIGN IN", new_user_session_path, data: {confirm: "로그인 하시겠습니까?"}%>
  <% end %>
  <%=link_to "HOME", root_path %>

<div class="container">
  <%= yield %>
</div>
```





