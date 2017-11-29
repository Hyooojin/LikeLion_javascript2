# jQuery & ajax on Rails

# jQuery?
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

출처: [왜 jQuery를 사용하는가?](http://unikys.tistory.com/300)

<br>
<br>

jQeury사이트: [http://jquery.com/](http://jquery.com/)


# [준비]
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
**controller, model 관계 설정** 
model관의 관계는 post랑 comment

```ruby
# post.rb
has_many :comments

# comment.rb
belongs_to :post
```

**route 설정**

```ruby
# routes.rb

root 'posts#index'

```

**bootstrap 설정**

```ruby
# application.scss (css -> scss)
@import 'bootstrap';
```

**기본 view 설정** 

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

# [구현]
본격적으로 jQuery와 ajax를 이용하여 댓글달기 기능을 구현한다. 

# 댓글달기 기능

댓글을 ajax로 구현하기

## 1. 댓글달기 기능 기본 설정

### 1. 댓글을 달 수 있는 veiw 설정, form_tag로 comment를 달 수 있는 창 만들기.

**[View 설정: show.erb]** 

```html
# show.erb
<%=form_tag create_comment_to_post_path do%>
  <%=text_field_tag :body%>
  <%=submit_tag "댓글달기"%>
<% end %>
```

**[컨트롤러 설정: posts_controller.rb]**
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
**[error]**

```
create_comment
Completed 500 Internal Server Error in 3ms (ActiveRecord: 0.0ms)
```
<br>

## 2. jQuery 사용

### 1. jQuery 사용하기 위한 id 설정
<br>

**[jQuery를 위한 설정: show.erb]** 

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
**[jQuery를 활용한 자바스크립트 작성 : show.erb]** 

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
1. [form#comment, context: document, selector: "#comment"]
2. 코멘트 창에 커서를 둘 때마다 "haha" 가 늘어나는 것을 확인할 수 있다. 
3. form event
  * click -> submit
  * e (=event)
  * e.preventDefault()
    * event의 결과는 필요없다. 
    * 버튼을 누르는 것까지만 하고, url로 날라가는 단계는 생략할 수 있다.
    * 더 이상 templete missing이 일어나지 않는다. 

4. e.preventDefault <br>
e.preventDefault를 하지 않으면, action에 의해 다음 page로 넘어가는데, 다음 page로 넘어가지 않게 한다.
<br>
sumit이 성공하면, 해당요청을 중지시키고, ajax방법을 써서 post방식으로 요청을 보낸다. 

  <br>
  <br>

### 3. action이 다음페이지로 넘어가지 않고, console창에서 내용(value)확인

**[e.preventDefault를 이용: show.erb]**

[form#comment, context: document, selector: "#comment"]

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


# ajax?

ajax는 page의 과부하를 주지않고, 작성할 수 있게 해준다. 
<br>

## 3. ajax 기본
ajax로 구현하기 위해서는 기본적으로 4가지가 필요하다. 
<br>
* **라우팅**
* **리모트 true**
* **해당 method javascript파일**
* **플래그 사용? (.frozen이 하나의 기준이 된다.)**
<br>

ajax를 처음 접한 사람이라면, 수도코드를 작성하고 코드를 짜는 것이 좋다. 
<br>

### Q1. 댓글 달기 + ajax로 구현하기
--------------------------
1. input태그에 값(댓글내용)을 입력한다. 
* (0) submit 버튼을 클릭한다.(submit 이벤트 발생)
* (1) input태그에 있는 값을 가져온다. 
* (2) 값이 유효한지 확인한다. (빈칸인지 아닌지)
* (3) 값이 없으면 값을 넣으라는 안내메시지를 뿌린다. 
2. 값이 있을 경우, ajax로 처리한다. 
* (1) 현재 글은 어디인지, 작성자는 누구인지 파악한다.
* (2) DB에 댓글을 저장한다.
3. 서버에서 처리가 완료되면 화면에 댓글을 출력한다. 

### 1. 서버와의 통신
ajax를 이용해서 서버랑 통신할 수 있게 한다.
<br>

**[ajax 추가: show.erb]** 

```javascript
$.ajax({
        url: "<%=create_comment_to_post_path%>",
        method: "POST"
      })
```

1. 이렇게만 하면 페이지는 넘어가지는 않지만, 콘솔창에는 500eroor, missing templete에러가 발생한다.<br>

**[error]**

```
Completed 500 Internal Server Error in 20ms (ActiveRecord: 0.0ms)

ActionView::MissingTemplate
```
2. 원래는 서버랑 통신을 하지 않고 있어서, error가 나지 않았지만, ajax를 사용하여 서버랑 통신을 연결하면서 error가 발생하게 된다.
  <br>


### 2. 데이터 넘기기

**[서버로 데이터 넘기기: show.erb]** 

```javascript
      $.ajax({
        url: "<%=create_comment_to_post_path%>",
        method: "POST",
        data: {body: contents}
      })
```

**[서버로 데이터가 넘어왔는지 확인: posts_controller.rb]**
```ruby
  def create_comment
    puts params[:body]
  end
```
파라미터가 두개가 넘어온다. <br>
Parameters: {"body"=>"aa", "id"=>"1"}<br>

### 3. ActionView::MissingTemplate

create_comment.js.erb 를 만들어준다.

**[새로 만들어주기: view/posts/create_comment.js.erb]**

```javascript
# create_comment.js.erb
alert("댓글이 등록됨");
```

1. 댓글이 등록되었다는 알람창이 뜬다.
2. Post Load가 이루어진다.

```
  Post Load (0.2ms)  SELECT  "posts".* FROM "posts" WHERE "posts"."id" = ? LIMIT 1  [["id", 1]]
  Rendered posts/show.html.erb within layouts/application (1.3ms)
Completed 200 OK in 38ms (Views: 36.6ms | ActiveRecord: 0.2ms)
```
<br>

### 4. 데이터를 DB의 해당 row에 저장

**[posts_controller.erb]**

```ruby
    @c = Post.find(params[:id]).comments.create(
      body: params[:body]
      )
      
# Comment.create(body: params[:body])
      
```

rails/db를 확인하면 post id와 body값이 db에 들어오는 것을 확인할 수 있다. 
단, Comment.create를 할 경우 post id는 저장되지 않으므로 주의!
<br>
<br>
private활용

```ruby
before_action :set_post, only: [:show, :edit, :update, :destroy, :create_comment]

  def create_comment
    @c = @post.commnets.create(body: params[:body])
  end
```

### 4. javascript에서 페이지 넘기기

로그인 한 유저만이 댓글을 쓰도록 하고 싶다. 따라서 로그인 안 한 유저는 로그인 페이지로 보낸다.
<br>
기본적으로 액션과 이름이 같은 javascript파일을 랜딩한다. 
```ruby
  def create_comment
    unless user_signed_in?
      respond_to do |format|
        format.js {render 'please_login.js.erb'}
      end
    end
    @c = @post.comments.create(body: params[:body]) 
  end
```

**[error]**

```
ActionView::MissingTemplate (Missing template posts/please_login.js.erb, application/please_login.js.erb
```
<br>

please_login.js.erb를 만들어준다.

**[missing template 해결: please_login.js.erb]**

```js
if(confirm("로그인이 필요합니다. \n 로그인 페이지로 이동하시겠습니까?"))
location.href = "<%=new_user_session_path%>"
```

1. @post를 쓸 수 있도록 한다. 
2. redirect_to를 쓸 수 없다. 
3. user가 로그인을 안했을 경우, respond_to do |format|을 이용해서 다른 페이지로 넘긴다. 
4. please_login.js.erb 를 만들어준다. 

### 5. 인터넷 창에서 데이터 확인

**[해당 페이지에서 데이터를 직접 확인: show.erb]**

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

이럴경우 계속 새로고침을 눌러줘야 댓글이 추가되는 것을 볼 수 있다. 

참고: [HTML 표 작성](https://pat.im/209)

**[새로고침 없이 댓글 추가: create_comment.js.erb]**

```js
alert("댓글이 등록되었습니다.")
$('#body').val("");
$('#comment_table tbody').append(
`<tr>
    <td><%= @c.body%></td>
</tr>`);
```
append | prepend <br>
* prepend를 사용하면 최근에 단 댓글을 위에 달리게 할 수 있다. 
<br>

**[기존에 입력되었던 댓글들 추가: show.erb]**

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


### 6. ajax를 위한 수도코드
### Q2. 댓글 구현하기(ajax를  통해서 )
1. form태그 안에 input 태그 만들기
2. submit 이벤트가 발생했을 경우에
3. form태그 동작하지 않게 하기!
4. input태그 안에 있는 값 가져오기
* (1) 빈킨인 경우 알림주기
5. jQuery ajax를 이용해서 원하는 url로 데이터 보내기
* (1) 로그인하지 않은 경우 알림주기
6. 서버에서 댓글 등록하기
7. 댓글이 등록되었다고 알림주기
8. 페이지 refresh 없이 댓글 이어주기


# 좋아요 기능

## 1. 좋아요 기능 기본 설정

### 1. 수도코드 작성

### Q3. 좋아요 버튼 + ajax구현
1. 좋아요 버튼을 누른다. 
2. 기존에 버튼을 누른경우
* (1) 좋아요 삭제
* (2) 버튼을 like로 변경
3. 처음 누른 경우
* (1) 좋아요 등록
* (2) 버튼을 dislike로 변경
<br>

**jQuery 여러 사용법**

```javascript
$('css selector').on('eventName', function() {
  
});

$(document).on('eventName', 'css selector', function(){

});
```

### 2. 좋아요 버튼! view 작성

**[좋아요 버튼 만들기: show.erb]**

```html
<%=link_to 'Like', like_to_post_path, class: "btn btn-info", id: "like_button" %>
```
<br>

**[like_to의 prefix 설정: routes.rb]**

```ruby
 post '/like_post' => 'posts#like_post', as: 'like_to'
```
<br>

**[컨트롤러 설정: posts_controller.rb]**

```ruby
  def like_post
    puts "Like Post Success"
  end
```
### 3. 좋아요 모델 만들기

**[Like 모델 만들기]**

```ruby
$ rails g model Like user:references post:references
```
<br>

### 4. 모델의 관계 설정

```ruby
# references로 like.rb에는 belong_to가 생성되어 있다. 

# post.rb
has_many :likes

# user.rb
has_many :likes
```
<br>


### 5. event 설정

**[좋아요 이벤트 발생: show.erb]**
<br>
ajax로 e.prevent 랑 console창 확인하면서 구현

```javascript
 $(function() {
    $('#like_button').on('click', function(e) {
      e.preventDefault();
      console.log("Like Button Clicked");
    })
```
<br>

### 6. ajax를 이용해 event 처리

**[이벤트 처리: show.erb]**

```js
 $.ajax({
        method: "POST",
        url: "<%=like_to_post_path%>"
      })
```
ActionView::MissingTemplate

### 7. missing template 처리

**[새로만들기: like_post.js.erb]**

```js
alert("좋아요를 눌렀습니다.")
```

### 8. 좋아요 누르기 상황 설정

**[상황 설정: posts_controller.rb]**

```ruby
  before_action :set_post, only: [:show, :edit, :update, :destroy, :create_comment, :like_post]
 
  def like_post
  def like_post
    puts "Like Post Sucess"
    unless user_signed_in?
      respond_to do |format|
        format.js {render 'please_login.js.erb'}
      end
    else
      if Like.where(user_id: current_user.id, post_id: @post.id).first.nil?
        @result = current_user.likes.create(post_id: @post.id)
        puts "좋아요"
      else
        @result = current_user.likes.find_by(post_id: @post.id).destroy
        puts "좋아요 취소"
      end
      puts "test"
      puts @result
    end
 
```
1. 상황설정에 따라 puts에 대한 값이 나오는지 확인
2. 상황설정 [1] : Like 모델
* user id값과 post id 값을 둘다 가지고 있다. row <br>
* user id값과 post id 값이 없는 row
* 로그인한 유저가 Like를 누르면 해당 post_id를 가지고, <br>
Like 모델의 row에 data가 추가된다. 
3. Like.where(user_id, post_id).first.nil? 
* DB에 아무런 정보가 없다 -> 해당 유저는 해당 포스트에 좋아요를 누르지 않았다.
4. else
* DB에 정보가 있다. 다시 한번 Like를 눌렀으므로, 취소한다. 
5. frozen을 활용하여 destroy를 해 준다. 
* frozen? True -> 얼어있다/ disliked 눌린 상태 -> Like 활성화

### 9. 좋아요 / 좋아요 취소

**[like_post.js.erb]**

```js

```

**[ajax작성]**
<br>

```
ORM 객체 == DB Row
Like.create => DH Row ++ ;
like.destroy => DB Row -- ;
@post.destroy
frozen =.
```


#### 0. ajax의 문제점

facebook을 ajax로 짠다면? 굉장한 server에 과부하를 주게된다. <br>
또한 server단을 단계별로 구분해 놓았다. <br>
좋아요를 눌르면 DB에 저장되는 것을 피하고 있다. 


### 삭제 버튼 구현

**[resources :posts]**
- /posts/:id
- /posts/:id/edit

member do
- /posts/:id/{내가 설정한 url(메소드)} 
이런식으로 url을 구성할 수 있다. 
id를 직접 넣지 않아도 된다. 

collection do
- /posts/{내가 설정한 url}
우리가 직접 설정해 주어야 한다. 




# 고치기


# Ajax2

# infinite scroll 기능 
ajax로 infinite scroll 구현하기

## 1. inifnite scroll 기본

### 1. pagination을 활용하기 위한 gem을 추가. 

**[pagination: gemfile 설정]**

```ruby
gem 'kaminari'

$ bundle install
```
<br>

### 2. view단에 쓸수 있도록..  

**[pagination 추가: index.html.erb]**

```ruby
<%= paginate @posts %>
```
<br>

<br>



## 2. jQuery 작성

### 1. 스크롤 이벤트를 console 창에서 확인

**[scroll 기능을 구현 :  index.html.erb ]**

```ruby
<script>
$(function() {
    $(document).on('scroll', fonction() {
      console.log("스크롤이 움직인다.");
    })
}) 
</script>
```

<br>

### 2. page 파라미터에 번호 매기기

**[scroll 기능을 구현 :  index.html.erb ]**

```ruby
<script>
$(function() {
  var page_scroll_index = 1;
    $(document).on('scroll', fonction() {
      console.log("움직인다다다다다");
    });
}); 
</script>
```


## 3. 한페이지에서 스크롤이 끝났을 경우, 계속 되도록, 알고리즘 

<br>

### 1. 페이지의 끝을 알 수 있어야 한다. 

브라우저 콘솔창에서 확인해 보기 

```html
$(window).height();
509
# 브라우저

$(document).height();
509

$(window).scrollTop();
0
```
* $(window).height();
* $(document).height();
* $(window).scrollTop();

> 다이나믹하게 변하는 것 2개와 고정된 것이  하나가 있다. **document**는 우리가 받아 온 전체 html 문서이고, **window**는 우리가 보고 있는 창을 말한다. 
> 즉, 우리가 실제로 볼 수 있는 창이 window.height이고 ,document.height는 처음부터 끝까지 높이라고 할 수 있다. 
<br>

### 2. 페이지의 끝을 알 수 있는 알고리즘 구현

**[index.html.erb]**

```js
$(window).scrollTop() >= $(document).height() - $(window).height();
```

```js
$(function() {
  var page_scroll_index = 2;
  $(document).on('scroll', function() {
    var end = $(window).scrollTop() >= $(documnet).height() - $ $(window).height();
    console.log(end);
  })
})
```

<br>

### 3. 


1. true가 나왔을때만!
2. if 조건문 안에 넣어준다. 
3. ajax로 보내준다. 

<table>

	<th></th>
</table>



```js
<script>
$(function() {
  var page_scroll_index = 1;
    $(document).on('scroll', function() {
      // console.log("움직인다다다다다");
      //var end =
      if($(window).scrollTop() >= $(document).height() - $(window).height()){
        // console.log(end);
        $.ajax({
          method: "GET",
          url: "<%= scroll_posts_path %>"
        });
      }
    });
}); 
</script>
```



```ruby
 def page_scroll
    puts "haha"
  end
```









page scoll

1. index에서 id를 찾는다. 
2. 아무 에러가 안나면 javascript파일을 확인, 500에러면 ruby코드





### 1.2.3.4.5.6.

1. ofset을 늘려줘야 하낟. 

2. 강제로 늘려주기

```js
$.ajax({
method: "GET",
url: "<%= scroll_posts_path %>",
data: {
page: page_scroll_index
```

3. pge라는 파라미터가 넘어온다. 
4. 계속 번호가 이어지게 하기위해서는 

5. page 처음은 1이니까 그에 상응하는 page로 맞춰준다. 

```js
page: page_scroll_index++

#고쳐주기
var page_scroll_index = 2;
```



### 새글을 작성하는데 순서를 바꾸기

**[posts_controller.rb]**

```ruby
# #index, #page_scroll
@posts = Post.order("created_at DESC ").page(params[:page])
```



### default 값을 늘리기

1pxel만 있어도 스크롤이 있다고 생각, 40개씩 보여지도록 한다. 

**해결 1. [post.rb]**

```ruby
paginates_per 40
```

kaminari에 있는 method이므로, kaminari의 옵션이다. 

<br>

**해결 2. [사이즈 강제로 늘리기]**



### response로 오는 코드 바꾸기

```js
alert("스크롤 무한스크롤 ㅋㅋㅋ");

     $('#myTable tbody').append(
     `
     <% @posts.each do |post| %>
     <tr>
         <td><%= post.id %></td>
         <td><%= post.title %></td>
         <td><%= truncate post.content, length: 10 %></td>
         <td><%= link_to 'Show', post %></td>
         <td><%= link_to 'Edit', edit_post_path(post) %></td>
         <td><%= link_to 'Destroy', post, method: :delete, data: { confirm: 'Are you sure?' } %></td>
     </tr>
     <% end %>
     `);   
```
method를 부르는 횟수를 줄이는 것이 좋다. !! <br>
40개의 테이블을 만들어 놓고 붙이는 형식으로 진행된다. 


# validation => model 자체에서 구현가능
# text_area_field에 글자수 제한하기
# asset pipeline => bootstrap thema 사용하기
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

출처: [왜 jQuery를 사용하는가?](http://unikys.tistory.com/300)

<br>
<br>

jQeury사이트: [http://jquery.com/](http://jquery.com/)


# 댓글에 validation 기능
validation(유효성 검사) <br>
우리가 원하는 길이인가? (최소, 최대) <br>
우리가 원하는 형태인가? <br>
<br>
Validation이라는 것은 front, server 둘다에서 작성해야 한다. <br> 
front end에서 길이를 확인<br>
back end에서는 유효성 검사

## 1. Front: 댓글에 validation 주기
### 1.  댓글에 validation 주기

**[show.erb]**

```js
     $('#comment').on('keypress', function() {
       console.log("hahaha");
     })
```
* keyboard event 
  [https://api.jquery.com/category/events/keyboard-events/](https://api.jquery.com/category/events/keyboard-events/)

keypress는 문자가 추가될 때마다, keydown(), keyup() <br>
키보드에서 손을 떼었을때 event가 발생하는 것으로 사용
<br>

`#comment`는 form에 달려있는 아이지 input에 달려 있는 아이가 아니다. 


### 2. keypress, keyup
```js
     $('#comment').on('keyup', function() {
       var com = $('#comment_body').val();
       console.log(com);
     })
```
keypress는 나중에 찍히고, keyup은 눌렀다가 떼는 순간 event가 발생한다. 

### 3. 글자하나씩 count하기 

```js
     $('#comment').on('keyup', function() {
       var text_length = $('#comment_body').val().length;
       console.log(text_length);
       
     })
```

### 4. 전체글자수 .. 현재 글자수 나타내기 

```html
<h3><span id="word_count">0</span>/50</h3>
```

### 5. 0을 현재 글자수로 바꾸기 

```js
    $('#word_count').text(text_length);    
```

### 6. word count가 늘어나는 것을 막기
this가 바인딩 되는 곳
```ruby
    var max_text_length = 50;
     $('#comment').on('keyup', function() {
       var text_length = $('#comment_body').val().length;
      $('#word_count').text(text_length);      
      // console.log(text_length);
      if(내가 입력한 텍스트의 길이가 최대 길이를 넘으면) {
        alert("최대 길이 넘음");
      }
       
     })
```
1. 길이가 넘으면 지워준다. 
2. 기존의 `comment_body`로 바꾼다. 
3. substring method

[W3school](https://www.w3schools.com/jsref/jsref_substr.asp)

```js
    var max_text_length = 50;
     $('#comment').on('keyup', function() {
       var text_length = $('#comment_body').val().length;
      
      $('#word_count').addClass('text-success').removeClass('text-danger');
      // console.log(text_length);
      if(text_length >= max_text_length) {
        alert("최대 길이 넘음");
        
        $('#word_count').addClass('text-danger').removeClass('text-sucess');
        $('#comment_body').val($('#comment_body').val().substr(0, max_text_length))
        text_length = $('#comment_body').val().length;
        
      }
       $('#word_count').text(text_length); 
     })
```


## 2. Back: 댓글에 validation 주기
### 1.  댓글에 validation 주기
코멘트를 입력하는것이 :body로 되어 있다. 
**[comment.rb]**
```ruby
  validates :body, length: {maximum: 40},
                    presence: true
```
프론트에서는 50자 제한
백에서는 40자 제한.

### 2. 백엔드에서 길이가 기억안나면 

```ruby
class Comment < ActiveRecord::Base
    
    def self.MAX_LENGTH
         40 
    end
    
    
  belongs_to :post
  validates :body, length: {maximum: self.MAX_LENGTH},
                    presence: true
                    

  
end
```

# text_area_field 글자수 제한
## 1.  
### 1. 




































# [준비]
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
**controller, model 관계 설정** 
model관의 관계는 post랑 comment

```ruby
# post.rb
has_many :comments

# comment.rb
belongs_to :post
```

**route 설정**

```ruby
# routes.rb

root 'posts#index'

```

**bootstrap 설정**

```ruby
# application.scss (css -> scss)
@import 'bootstrap';
```

**기본 view 설정** 

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






























# [구현]
본격적으로 jQuery와 ajax를 이용하여 댓글달기 기능을 구현한다. 

# 댓글달기 기능

## 1. 댓글달기 기능 기본 설정

### 1. 댓글을 달 수 있는 veiw 설정, form_tag로 comment를 달 수 있는 창 만들기.

**[View 설정: show.erb]** 

```html
# show.erb
<%=form_tag create_comment_to_post_path do%>
  <%=text_field_tag :body%>
  <%=submit_tag "댓글달기"%>
<% end %>
```

**[컨트롤러 설정: posts_controller.rb]**
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
**[error]**

```
create_comment
Completed 500 Internal Server Error in 3ms (ActiveRecord: 0.0ms)
```
<br>

## 2. jQuery 사용

### 1. jQuery 사용하기 위한 id 설정
<br>

**[jQuery를 위한 설정: show.erb]** 

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
**[jQuery를 활용한 자바스크립트 작성 : show.erb]** 

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
1. [form#comment, context: document, selector: "#comment"]
2. 코멘트 창에 커서를 둘 때마다 "haha" 가 늘어나는 것을 확인할 수 있다. 
3. form event
  * click -> submit
  * e (=event)
  * e.preventDefault()
    * event의 결과는 필요없다. 
    * 버튼을 누르는 것까지만 하고, url로 날라가는 단계는 생략할 수 있다.
    * 더 이상 templete missing이 일어나지 않는다. 

4. e.preventDefault <br>
  e.preventDefault를 하지 않으면, action에 의해 다음 page로 넘어가는데, 다음 page로 넘어가지 않게 한다.
  <br>
  <br>

### 3. action이 다음페이지로 넘어가지 않고, console창에서 내용(value)확인

**[e.preventDefault를 이용: show.erb]**

[form#comment, context: document, selector: "#comment"]

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


# ajax?

ajax는 page의 과부하를 주지않고, 작성할 수 있게 해준다. 
<br>

## 3. ajax 기본

ajax를 처음 접한 사람이라면, 수도코드를 작성하고 코드를 짜나가는 것을 추천한다고 한다. 
<br>

### Q1. 댓글 달기 + ajax로 구현하기
--------------------------
1. input태그에 값(댓글내용)을 입력한다. 
* (0) submit 버튼을 클릭한다.(submit 이벤트 발생)
* (1) input태그에 있는 값을 가져온다. 
* (2) 값이 유효한지 확인한다. (빈칸인지 아닌지)
* (3) 값이 없으면 값을 넣으라는 안내메시지를 뿌린다. 
2. ajax로 처리한다. 

3. 서버에서 처리가 완료되면 화면에 댓글을 출력한다. 

### 1. 서버와의 통신
ajax를 이용해서 서버랑 통신할 수 있게 한다.
<br>

**[ajax 추가: show.erb]** 

```javascript
$.ajax({
        url: "<%=create_comment_to_post_path%>",
        method: "POST"
      })
```

1. 이렇게만 하면 페이지는 넘어가지는 않지만, 콘솔창에는 500eroor, missing templete에러가 발생한다.<br>

**[error]**

```
Completed 500 Internal Server Error in 20ms (ActiveRecord: 0.0ms)

ActionView::MissingTemplate
```
2. 원래는 서버랑 통신을 하지 않고 있어서, error가 나지 않았지만, ajax를 사용하여 서버랑 통신을 연결하면서 error가 발생하게 된다.
  <br>


### 2. 데이터 넘기기

**[서버로 데이터 넘기기: show.erb]** 

```javascript
      $.ajax({
        url: "<%=create_comment_to_post_path%>",
        method: "POST",
        data: {body: contents}
      })
```

**[서버로 데이터가 넘어왔는지 확인: posts_controller.rb]**
```ruby
  def create_comment
    puts params[:body]
  end
```
파라미터가 두개가 넘어온다. <br>
Parameters: {"body"=>"aa", "id"=>"1"}<br>

### 3. ActionView::MissingTemplate

create_comment.js.erb 를 만들어준다.

**[새로 만들어주기: view/posts/create_comment.js.erb]**

```javascript
# create_comment.js.erb
alert("댓글이 등록됨");
```

1. 댓글이 등록되었다는 알람창이 뜬다.
2. Post Load가 이루어진다.

```
  Post Load (0.2ms)  SELECT  "posts".* FROM "posts" WHERE "posts"."id" = ? LIMIT 1  [["id", 1]]
  Rendered posts/show.html.erb within layouts/application (1.3ms)
Completed 200 OK in 38ms (Views: 36.6ms | ActiveRecord: 0.2ms)
```
<br>

### 4. 데이터를 DB의 해당 row에 저장

**[posts_controller.erb]**

```ruby
    @c = Post.find(params[:id]).comments.create(
      body: params[:body]
      )
      
# Comment.create(body: params[:body])
      
```

rails/db를 확인하면 post id와 body값이 db에 들어오는 것을 확인할 수 있다. 
단, Comment.create를 할 경우 post id는 저장되지 않으므로 주의!
<br>
<br>
private활용

```ruby
before_action :set_post, only: [:show, :edit, :update, :destroy, :create_comment]

  def create_comment
    @c = @post.commnets.create(body: params[:body])
  end
```

### 4. javascript에서 페이지 넘기기

로그인 한 유저만이 댓글을 쓰도록 하고 싶다. 따라서 로그인 안 한 유저는 로그인 페이지로 보낸다.

```ruby
  def create_comment
    unless user_signed_in?
      respond_to do |format|
        format.js {render 'please_login.js.erb'}
      end
    end
    @c = @post.comments.create(body: params[:body]) 
  end
```

**[error]**

```
ActionView::MissingTemplate (Missing template posts/please_login.js.erb, application/please_login.js.erb
```
<br>

please_login.js.erb를 만들어준다.

**[missing template 해결: please_login.js.erb]**

```js
if(confirm("로그인이 필요합니다. \n 로그인 페이지로 이동하시겠습니까?"))
location.href = "<%=new_user_session_path%>"
```

1. @post를 쓸 수 있도록 한다. 
2. redirect_to를 쓸 수 없다. 
3. user가 로그인을 안했을 경우, respond_to do |format|을 이용해서 다른 페이지로 넘긴다. 
4. please_login.js.erb 를 만들어준다. 

### 5. 인터넷 창에서 데이터 확인

**[해당 페이지에서 데이터를 직접 확인: show.erb]**

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

이럴경우 계속 새로고침을 눌러줘야 댓글이 추가되는 것을 볼 수 있다. 

참고: [HTML 표 작성](https://pat.im/209)

**[새로고침 없이 댓글 추가: create_comment.js.erb]**

```js
alert("댓글이 등록되었습니다.")
$('#body').val("");
$('#comment_table tbody').append(
`<tr>
    <td><%= @c.body%></td>
</tr>`);
```
append | prepend <br>
* prepend를 사용하면 최근에 단 댓글을 위에 달리게 할 수 있다. 
  <br>

**[기존에 입력되었던 댓글들 추가: show.erb]**

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


### 6. ajax를 위한 수도코드
### Q2. 댓글 구현하기(ajax를  통해서 )
1. form태그 안에 input 태그 만들기
2. submit 이벤트가 발생했을 경우에
3. form태그 동작하지 않게 하기!
4. input태그 안에 있는 값 가져오기
* (1) 빈킨인 경우 알림주기
5. jQuery ajax를 이용해서 원하는 url로 데이터 보내기
* (1) 로그인하지 않은 경우 알림주기
6. 서버에서 댓글 등록하기
7. 댓글이 등록되었다고 알림주기
8. 페이지 refresh 없이 댓글 이어주기


# 좋아요 기능

## 1. 좋아요 기능 기본 설정

### 1. 수도코드 작성

### Q3. 좋아요 버튼 + ajax구현
1. 좋아요 버튼을 누른다. 
2. 버튼을 누른경우
* (1) 기존에 좋아요를 이미 누른 경우
* (2) 기존에 좋아요를 누르지 않은 경우
3. 이미 누른 경우
* (1) 좋아요 삭제
* (2) 
  <br>

**jQuery 여러 사용법**

```javascript
$('css selector').on('eventName', function() {
  
});

$(document).on('eventName', 'css selector', function(){

});
```

### 2. 좋아요 버튼! view 작성

**[좋아요 버튼 만들기: show.erb]**

```html
<%=link_to 'Like', like_to_post_path, class: "btn btn-info", id: "like_button" %>
```
<br>

**[like_to의 prefix 설정: routes.rb]**

```ruby
 post '/like_post' => 'posts#like_post', as: 'like_to'
```
<br>

**[컨트롤러 설정: posts_controller.rb]**

```ruby
  def like_post
    puts "Like Post Success"
  end
```
### 3. 좋아요 모델 만들기

**[Like 모델 만들기]**

```ruby
$ rails g model Like user:references post:references
```
<br>

### 4. 모델의 관계 설정

```ruby
# references로 like.rb에는 belong_to가 생성되어 있다. 

# post.rb
has_many :likes

# user.rb
has_many :likes
```
<br>


### 5. event 설정

**[좋아요 이벤트 발생: show.erb]**
<br>
ajax로 e.prevent 랑 console창 확인하면서 구현

```javascript
 $(function() {
    $('#like_button').on('click', function(e) {
      e.preventDefault();
      console.log("Like Button Clicked");
    })
```
<br>

### 6. ajax를 이용해 event 처리

**[이벤트 처리: show.erb]**

```js
 $.ajax({
        method: "POST",
        url: "<%=like_to_post_path%>"
      })
```
ActionView::MissingTemplate

### 7. missing template 처리

**[새로만들기: like_post.js.erb]**

```js
alert("좋아요를 눌렀습니다.")
```

### 8. 좋아요 누르기 상황 설정

**[상황 설정: posts_controller.rb]**

```ruby
  before_action :set_post, only: [:show, :edit, :update, :destroy, :create_comment, :like_post]
 
  def like_post
    # puts "Like Post Success"
    unless user_signed_in?
      respond_to do |format|
        format.js {render 'please_login.js.erb'}
      end
    else
      if Like.where(user_id: current_user.id, post_id: @post.id).first.nil?
        # 좋아요 누르지 않은 상태에 대한 실행문
        # 좋아요를 만들어준다.
        @result = current_user.likes.create(post_id: @post.id)
        # puts "좋아요 누름"
      else 
        # 좋아요를 누른 상태에 대한 실행문
        # 기존의 좋아요를 삭제한다. 
        
        @result = current_user.likes.find_by(post_id: @post.id).destroy
        # puts "좋아요 취소"    
      end
      # puts ("test가 나오나 안나오나")
      @result = @result.frozen?
      puts @result
    end
 
```
1. 상황설정에 따라 puts에 대한 값이 나오는지 확인
2. 


**[ajax작성]**
<br>

```
ORM 객체 == DB Row
Like.create => DH Row ++ ;
like.destroy => DB Row -- ;
@post.destroy
frozen =.
```



