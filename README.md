# jQuery & ajax on Rails

# 1.jQuery
## **jQuery** ?
jQuery는 가볍고, DOM탐색이나 이벤트, 애니메이션, ajax등을 활용할 때 유용하게 사용할 수 있는 라이브러리이다.
<br>

**[javascript]**
```javascript
document.getElementById("divId"); document.getElementsByClassName("className"); document.getElementsByTagName("input");
```
<br>
**[jQuery]**

```javascript
$("#divId"); $(".className"); $("input");
```

출처: [왜 jQuery를 사용하는가?](출처: http://unikys.tistory.com/300 [All-round programmer])
<br>
jQeury사이트: [http://jquery.com/](http://jquery.com/)


# 준비
Rails에서 jQuery와 ajax를 이용해 CRUD를 구현한다.

## CRUD 기본 설정

**gemfile 설정**
```ruby
# gem file
gem 'devise'
gem 'faker'
gem 'kaminari'
gem 'bootstrap-sass'
```
**controller, model 설정** 

```ruby
$ rails g devise:install
$ rails g scaffold post title:string content:text
$ rake db:migrate

$ rails g devise user
$ rails g model comments post:references body:text
$ rake db:migrate
```
** controller, model 관계 설정 ** 
model관의 관계는 post랑 comment

```ruby
# post.rb
has_many :comments

# comment.rb
belongs_to :post
```

** route 설정 **

```ruby
# routes.rb

root 'posts#index'

```

** bootstrap 설정 **

```ruby
# application.scss (css -> scss)
@import 'bootstrap';
```

** 기본 view 설정 ** 

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

# 구현
본격적으로 jQuery와 ajax를 이용하여 댓글달기 기능을 구현한다.

# 1. 댓글달기 기능 

## 1. 댓글달기 기본 설정

### 1. 댓글을 달 수 있는 veiw 설정, form_tag로 comment를 달 수 있는 창 만들기.

** [View 설정: show.erb] ** 

```html
# show.erb
<%=form_tag create_comment_to_post_path do%>
  <%=text_field_tag :body%>
  <%=submit_tag "댓글달기"%>
<% end %>
```

** [컨트롤러 설정: posts_controller.rb] **
```ruby
# posts_controller.rb
  def create_comment
    puts "create_comment"
  end
```
webpage에서 console창 확인
<br>
comment를 달 수 있는 창이 생겼다. 

### 2. path를 설정해주기 위한 route.rb 작업 
<br>

```ruby
  resources :posts do
    member do
      post '/create_comment' => 'posts#create_comment', as: 'create_comment_to'
    end
  end
```
rake routes 확인해 보면, creat_comment_to_post_path로 prefix가 만들어진것을 확인할 수 있다.<br> 
<br>
**Template is missing** => comment창을 확인하면, create_comment는 볼 수 있다. 
<br>
<br>
### 3. Template error를 jQuery와 ajax로 차근차근 고쳐나가기
<br>
<br>
**<error>**
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
<br>

## 2. jQuery 사용

### 1. jQuery 사용하기 위한 id 설정
<br>

** [jQuery를 위한 설정: show.erb] ** 
```html
<%=form_tag create_comment_to_post_path, id: "comment" do%>
  <%=text_field_tag :body%>
  <%=submit_tag "댓글달기"%>
<% end %>
```
### 2. 인터넷 콘솔창에 jQuery 이용한 로그 찍기  
알맞는 기능을 구현되고 있는지, 인터넷창에서 콘솔창으로 로그를 확인할 수 있다.
<br>
<br>
** [jQuery를 활용한 자바스크립트 작성 : show.erb] ** 

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

* e.preventDefault <br>
  e.preventDefault를 하지 않으면, action에 의해 다음 page로 넘어가는데, 다음 page로 넘어가지 않게 한다.
  <br>
  <br>
### 3. content확인 : show.erb 

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
<br>
<br>
# 3. ajax
ajax는 page의 과부하를 주지않고, 작성할 수 있게 해준다. 
<br>

## 1. ajax 기본

ajax를 처음 접한 사람이라면, 수도코드를 작성하고 코드를 짜나가는 것을 추천한다고 한다. 
<br>

### Q1. 댓글 달기 + ajax로 구현하기
1. input태그에 값(댓글내용)을 입력한다. 
  (0) submit 버튼을 클릭한다.(submit 이벤트 발생)
  (1) input태그에 있는 값을 가져온다. 
  (2) 값이 유효한지 확인한다. (빈칸인지 아닌지)
  (3) 값이 없으면 값을 넣으라는 안내메시지를 뿌린다. 
2. ajax로 처리한다. 
  (0) 현재 글을 
3. 서버에서 처리가 완료되면 화면에 댓글을 출력한다. 

### 1. 서버와의 통신
ajax => 서버랑 통신
<br>
** [show.erb] ** 
```javascript
$.ajax({
        url: "<%=create_comment_to_post_path%>",
        method: "POST"
      })
```
1. 이렇게만 하면 missing templete에러가 발생한다. 
2. 500에러
3. 원래는 서버랑 통신을 하지 않고 있어서, error가 나지 않았지만, ajax를 사용하여 서버랑 통신을 연결하면서 error가 발생하게 된다.
  <br>
  <br>
  **<error>**
```ruby
500 (Internal Server Error)
```

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

### 2. 데이터 넘기기
** show.erb ** 
```ruby
      $.ajax({
        url: "<%=create_comment_to_post_path%>",
        method: "POST",
        data: {body: contents}
      })
```

**[posts_controller.rb]**
```ruby
  def create_comment
    puts params[:body]
  end
```

### 3. templete missing
create_comment.js.erb 를 만들어준다.
** [view/posts/create_comment.js.erb] **

```javascript
# create_comment.js.erb
alert("댓글이 등록됨");
```
<br>
** [posts_controller.erb] **
```ruby
before_action :set_post, only: [:show, :edit, :update, :destroy, :create_comment]

  def create_comment
    # puts params[:body]
    @c = @post.commnets.create(body: params[:body]) # 와일드카드를 쓰고 있으므로 id도 같이 넘어온다. 
  end
```
### 4. javascript에서 페이지 넘기기

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
please_login.js.erb를 만들어준다.

** [please_login.js.erb]**

```html
if(confirm("로그인이 필요합니다. \n 로그인 페이지로 이동하시겠습니까?"))
location.href = "<%=new_user_session_path%>"
```

1. @post를 쓸 수 있도록 한다. 
2. redirect_to를 쓸 수 없다. 
3. user가 로그인을 안했을 경우, respond_to do |format|을 이용해서 다른 페이지로 넘긴다. 
4. please_login.js.erb 를 만들어준다. 

### 5. 인터넷 창에서 데이터 확인
** [view 깔끔하게: show.erb]**
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

**[댓글 추가: create_comment.js.erb]**
```javascript
alert("댓글이 등록됨");
$('#body').val("");
$('#comment_table tbody').append(
`<tr>
    <td><%= @c.body%></td>
</tr>`);
```
append | prepend

** [기존에 입력되었던 댓글들 추가: show.erb]**

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

## 2. ajax를 위한 수도코드
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


## 3. 좋아요 기능 구현

```javascript
$('css selector').on('eventName', function() {
  
});

$(document).on('eventName', 'css selector', function(){
  
});

```
### Q3. 좋아요 버튼 + ajax구현
1. 좋아요 버튼을 누른다. 
2. 버튼을 누른경우
  (1) 기존에 좋아요를 이미 누른 경우
  (2) 기존에 좋아요를 누르지 않은 경우
3. 이미 누른 경우
  (1) 좋아요 삭제
  (2) 

4. ​

### 1. 좋아요 버튼! 

** [좋아요 버튼 만들기: show.erb]**

```html
<%=link_to 'Like', like_to_post_path, class: "btn btn-info", id: "like_button" %>
```
<br>
** [routes.rb] **
```ruby
 post '/like_post' => 'posts#like_post', as: 'like_to'
```
<br>
### 2. 좋아요 모델 만들기
**[Like 모델 만들기] **

```ruby
$ rails g model Like user:references post:references
```
<br>
** [show.erb] **
ajax로 e.prevent 랑 console창 확인하면서 구현

```javascript
 $(function() {
    
    $('#like_button').on('click', function(e) {
      e.preventDefault();
      console.log("Like Button Clicked");
    })
```
### 3. 모델 관계 설정
<br>

```ruby
# references로 like.rb에는 belong_to가 생성되어 있다. 

# post.rb
has_many :likes

# user.rb
has_many :likes
```



```javascript

```

```ruby
  def like_post
    puts "Like Post Success"
  end
```

### ajax작성

```
ORM 객체 == DB Row
Like.create => DH Row ++ ;
like.destroy => DB Row -- ;
@post.destroy
frozen =.



```