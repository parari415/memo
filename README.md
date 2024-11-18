### エラー起きたら
`bundle install`を試してみる


# 手順①pull
## 前回の変更部分(Yブランチ)を確認してmainにマージした+私の変更部分もマージしているのでローカルに反映させる  
### i.おそらくYブランチにいると思うのでmainに移動する  
`git checkout main`  
### ⅱ.mainに移動できているか確認  
`git status`  
### ⅲ.移動できていたら変更をローカルに反映させる
`git pull`


# 手順②jsの反映
## 前回作ってもらったメニューバーのjs部分(ボタンで開閉)が反映できていなかったと思うので変更
### ⅰ.もう一度Yブランチに移動
`git checkout Y`
### ⅱ.Yに移動できているか確認
`git status`
`git pull origin main`
### ⅲ.フォルダ・ファイルの変更(今後再変更の可能性有)
app/javascript/application.jsを以下に変更
```
// Configure your import map in config/importmap.rb. Read more: https://github.com/rails/importmap-rails
import "@hotwired/turbo-rails"
import "controllers"

document.getElementById('toggle-btn').addEventListener('click', function() {
    var sidebar = document.getElementById('sidebar');
    var content = document.getElementById('content');
    sidebar.classList.toggle('hide');
    content.classList.toggle('hide');
});
```
app/javascript/packs/application.js  
packsフォルダ削除(せっかく作ってもらったけどm(_ _)m)
### ⅳ.動作確認(ボタンで開閉することを確認)
`rails server`  
もしかしたら`rails db:migrate`しないとエラーでるかも
### ⅴ.変更をコミット・プッシュ
`git add .`
`git commit -m "メッセージなんでも"`
`git push origin Y`

# 手順③私の変更進捗確認(②変更後に問題おきていないか確認)
### ⅰ.コンソールで確認要データ入力
ターミナルで`rails db:migrate`  
`rails console`  
以下一気に貼り付けて大丈夫
```
recipe = Recipe.create(
  title: "ハンバーグ",
  time: 30,
  kcal: 500,
  category: "洋食"
)
```
```
ingredient1 = recipe.ingredients.create(name: "牛ひき肉 200g")
ingredient2 = recipe.ingredients.create(name: "玉ねぎ 1個")
ingredient3 = recipe.ingredients.create(name: "パン粉 大さじ2")
ingredient4 = recipe.ingredients.create(name: "卵 1個")
ingredient5 = recipe.ingredients.create(name: "塩・こしょう 適量")
procedure1 = recipe.procedures.create(step: "玉ねぎをみじん切りにして炒める。")
procedure2 = recipe.procedures.create(step: "ボウルに牛ひき肉、玉ねぎ、パン粉、卵、塩・こしょうを入れて混ぜる。")
procedure3 = recipe.procedures.create(step: "混ぜた具材をハンバーグの形に整える。")
procedure4 = recipe.procedures.create(step: "フライパンで両面を焼き、中まで火を通す。")
```
`exit`
### ⅱ.起動と確認(先週書いてもらったcssの影響受けて配置崩れている可能性があるが一旦無視でOK)
`rails server`  
urlを変更  
urlの後ろに/recipes/1を追加  
例http://127.0.0.1:3000/recipes/1  
urlの後ろに/newを追加  
show(recipe/1)とnewの2ページ報告書のようなものが表示できればOK

# 手順④マージ
別途詳細
# 手順⑤プル(手順①と同作業)

# 手順⑥ユーザーモデル作成とログイン機能
`git checkout -b　session`でブランチ移動
### ⅰ.モデルとコントローラー作成
```
rails generate model User uid:string pass:string
rails generate controller Sessions
rails generate controller Users
rails db:migrate
```
### ⅱ.config/routes.rbに以下を追加
```
get "login", to: "sessions#new"
post "login", to: "sessions#create"
delete "logout", to: "sessions#destroy"

resources :users, only: [:new, :create]
```
### ⅲ.app/controllers/sessions_controller.rbを変更
```
class SessionsController < ApplicationController
  def new
    # ログインフォームの表示
  end

  def create
    user = User.find_by(uid: params[:uid])
    if user && BCrypt::Password.new(user.pass) == params[:pass]
      session[:login_uid] = user.uid
      redirect_to root_path, notice: "ログインしました"
    else
      flash[:alert] = "ログインに失敗しました"
      render :new, status: 422
    end
  end

  def destroy
    session.delete(:login_uid)
    redirect_to root_path, notice: "ログアウトしました"
  end
end
```
### ⅳ.app/controllers/users_controller.rbを変更
```
class UsersController < ApplicationController
  def new
    @user = User.new
  end

  def create
    password = BCrypt::Password.create(params[:user][:pass])
    @user = User.new(uid: params[:user][:uid], pass: password)
    if @user.save
      redirect_to login_path, notice: "登録が完了しました。ログインしてください。"
    else
      flash[:alert] = "登録に失敗しました"
      render :new, status: 422
    end
  end
end
```
### ⅴ.app/views/sessions/new.html.erbを作成
```
<h1>ログイン</h1>
<%= form_with url: login_path, local: true do |f| %>
  <div>
    <%= f.label :uid, "ユーザーID" %>
    <%= f.text_field :uid %>
  </div>
  <div>
    <%= f.label :pass, "パスワード" %>
    <%= f.password_field :pass %>
  </div>
  <%= f.submit "ログイン" %>
<% end %>
```
### ⅵ.app/views/users/new.html.erbを作成
```
<h1>新規登録</h1>
<%= form_with model: @user, local: true do |f| %>
  <div>
    <%= f.label :uid, "ユーザーID" %>
    <%= f.text_field :uid %>
  </div>
  <div>
    <%= f.label :pass, "パスワード" %>
    <%= f.password_field :pass %>
  </div>
  <%= f.submit "登録" %>
<% end %>
```
### ⅶ.動作確認
urlの後ろにusers/newを追加  
ユーザーの新規登録をしてみる⇨ログイン画面に遷移すると思うので登録したIDとパスでログイン  
topページに遷移すればOK
urlの後ろに/loginを追加  
登録していないIDとパスでログイン  
ログインページに再リロードすればOK
### ⅷ.コミット
`git add .`  
`git commit -m "session and userモデル"`  
`git push origin session`  
マージはまだしなくてOK
