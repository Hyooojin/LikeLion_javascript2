# rails CRUD & jQuery 

## 1. CRUD 설정

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


## 2. comment 창 

* comment를 달 수 있도록 한다. 
```ruby
  resources :posts do
    member do
      post '/create_comment' => 'posts#create_comment', as: 'create_comment_to'
    end
  end
```
rake routes 확인 

```ruby
# posts_controller.rb
  def create_comment
    puts "create_commnet"
  end
```
```html
# show.erb
<%=form_tag create_comment_to_post_path do%>
  <%=text_field_tag :body%>
  <%=submit_tag "댓글달기"%>
<% end %>
```

* **Template is missing**
  * comment창을 확인하면, create_comment를 볼 수 있으면 된다. 


```ruby
ActiveRecord::SchemaMigration Load (0.2ms)  SELECT "schema_migrations".* FROM "schema_migrations"
Processing by PostsController#show as HTML
  Parameters: {"id"=>"1"}
  Post Load (0.3ms)  SELECT  "posts".* FROM "posts" WHERE "posts"."id" = ? LIMIT 1  [["id", 1]]
  Rendered posts/show.html.erb within layouts/application (4.3ms)
Completed 200 OK in 421ms (Views: 383.1ms | ActiveRecord: 0.8ms)


Started POST "/posts/1/create_comment" for 203.246.196.65 at 2017-11-28 01:16:41 +0000
Cannot render console from 203.246.196.65! Allowed networks: 127.0.0.1, ::1, 127.0.0.0/127.255.255.255
Processing by PostsController#create_comment as HTML
  Parameters: {"utf8"=>"✓", "authenticity_token"=>"mgzTBaGOnM+vzqVAVQPOJncx91DrnfSWhqjQ79qd3jYrgvIJcJRUaTL0W+Q7nQw+r3eatQxZFkBC6DLcy2L4oA==", "body"=>"aaaa", "commit"=>"댓글달 기", "id"=>"1"}
create_commnet
Completed 500 Internal Server Error in 9ms (ActiveRecord: 0.0ms)
```
* jQuery사용하기 위한 id 설정

```ruby
<%=form_tag create_comment_to_post_path, id: "comment" do%>
  <%=text_field_tag :body%>
  <%=submit_tag "댓글달기"%>
<% end %>
```
* comment 창에 jQuery 활용

```javascript
<script>

  $(function() {
    var form = $('#comment');
    // console.log(form);
    form.on('click', function() { 
      console.log("haha");
    });
  })

  
</script>
```
1. 코멘트 창을 클릭할 때마다 haha가 늘어나는 것을 확인할 수 있다. 
2. form event
	* click -> submit
	* e (=event)
	* e.preventDefault()
		* event의 결과는 필요없다. 
		* 버튼을 누르는 것까지만 하고, url로 날라가는 단계는 생략할 수 있다.
		* 더 이상 templete missing이 일어나지 않는다. 

* e.preventDefault
```javascript
<script>

  $(function() {
    var form = $('#comment');
    // console.log(form);
    form.on('submit', function(e) {
      e.preventDefault();
      // console.log("haha");
      var contents = $('#body').val();
      console.log(contents)
    });
  })
</script>
```
## 3. ajax

### Q1. 댓글 달기 + ajax로 구현하기
1. input태그에 값(댓글내용)을 입력한다. 
	(0) submit 버튼을 클릭한다.(submit 이벤트 발생)
	(1) input태그에 있는 값을 가져온다. 
	(2) 값이 유효한지 확인한다. (빈칸인지 아닌지)
	(3) 값이 없으면 값을 넣으라는 안내메시지를 뿌린다. 
2. ajax로 처리한다. 
	(0) 현재 글을 
3. 서버에서 처리가 완료되면 화면에 댓글을 출력한다. 

* show.erb => ajax => 서버랑 통신
```javascript
$.ajax({
        url: "<%=create_comment_to_post_path%>",
        method: "POST"
      })
```
1. 이렇게만 하면 missing templete에러가 발생한다. 
2. 500에러
3. 원래는 서버랑 통신을 하지 않고 있어서, error가 나지 않았지만, ajax를 쓰면서 서버랑 통신을 연결하면서 error가 발생하게 된다.
```ruby
500 (Internal Server Error)
```
* error_message확인
```ruby
Post Load (0.1ms)  SELECT  "posts".* FROM "posts" WHERE "posts"."id" = ? LIMIT 1  [["id", 1]]
  Rendered posts/show.html.erb within layouts/application (1.5ms)
Completed 200 OK in 22ms (Views: 20.5ms | ActiveRecord: 0.1ms)


Started GET "/posts/1" for 203.246.196.65 at 2017-11-28 01:43:34 +0000
Cannot render console from 203.246.196.65! Allowed networks: 127.0.0.1, ::1, 127.0.0.0/127.255.255.255
Processing by PostsController#show as HTML
  Parameters: {"id"=>"1"}
  Post Load (0.2ms)  SELECT  "posts".* FROM "posts" WHERE "posts"."id" = ? LIMIT 1  [["id", 1]]
  Rendered posts/show.html.erb within layouts/application (2.2ms)
Completed 200 OK in 24ms (Views: 23.0ms | ActiveRecord: 0.2ms)


Started POST "/posts/1/create_comment" for 203.246.196.65 at 2017-11-28 01:43:38 +0000
Cannot render console from 203.246.196.65! Allowed networks: 127.0.0.1, ::1, 127.0.0.0/127.255.255.255
Processing by PostsController#create_comment as */*
  Parameters: {"id"=>"1"}
create_commnet
Completed 500 Internal Server Error in 22ms (ActiveRecord: 0.0ms)

ActionView::MissingTemplate
```

#### 데이터 넘기기

* show.erb

```ruby
      $.ajax({
        url: "<%=create_comment_to_post_path%>",
        method: "POST",
        data: {body: contents}
      })
```

* posts_controller.rb

```ruby
  def create_comment
    puts params[:body]
  end
```
1. templete missing에러 잡기


* view/posts/create_comment.js.erb

```javascript
# create_comment.js.erb
alert("댓글이 등록됨");
```



* posts_controller.erb
```ruby
before_action :set_post, only: [:show, :edit, :update, :destroy, :create_comment]

  def create_comment
    # puts params[:body]
    @c = @post.commnets.create(body: params[:body]) # 와일드카드를 쓰고 있으므로 id도 같이 넘어온다. 
  end
```
* javascript에서 페이지 넘기는 방법

```ruby
  def create_comment
    # puts params[:body]
    unless user_signed_in?
      respond_to do |format|
        format.js {render 'please_login.js.erb'}
      end
    end
    @c = @post.comments.create(body: params[:body]) # 와일드카드를 쓰고 있으므로 id도 같이 넘어온다. 
  end
```
<br>


* please_login.js.erb

```html
if(confirm("로그인이 필요합니다. \n 로그인 페이지로 이동하시겠습니까?"))
location.href = "<%=new_user_session_path%>"
```

1. @post를 쓸 수 있도록 한다. 
2. redirect_to를 쓸 수 없다. 
3. user가 로그인을 안했을 경우, respond_to do |format|을 이용해서 다른 페이지로 넘긴다. 
4. please_login.js.erb 를 만들어준다. 

* show.erb
```html
<table class="table", id="comment_table">
  <thead>
      <tr>
        <th>댓글</th>
      </tr>
  </thead>
  <tbody>
    <% @post.comments.each do |p|%>
      <tr>
          <td><%= p.body %></td>
      </tr>
    <% end %>
  </tbody>  
</table>

```


```javascript
alert("댓글이 등록됨");
$('#body').val("");
$('#comment_table tbody').append(
`<tr>
    <td><%= @c.body%></td>
</tr>`);
```
append | prepend

* 기존에 입력되었던 댓글들 추가

```html
  <tbody>
    <% @post.comments.each do |p|%>
      <tr>
          <td><%= p.body %></td>
      </tr>
    <% end %>
  </tbody>  
```
<% @post.comments.each do |p|%> |  <% @post.comments.reverse.each do |p|%>

## 4. 수도코드 
### Q2. 댓글 구현하기(ajax를  통해서 )
1. form태그 안에 input 태그 만들기
2. submit 이벤트가 발생했을 경우에
3. form태그 동작하지 않게 하기!
4. input태그 안에 있는 값 가져오기
	(1) 빈킨인 경우 알림주기
5. jQuery ajax를 이용해서 원하는 url로 데이터 보내기
	(1) 로그인하지 않은 경우 알림주기
6. 서버에서 댓글 등록하기
7. 댓글이 등록되었다고 알림주기
8. 페이지 refresh 없이 댓글 이어주기
