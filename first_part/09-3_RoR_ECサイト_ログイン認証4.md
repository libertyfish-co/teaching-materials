## 9.3 Ruby on Rails：ECサイトの開発 ログイン認証とユーザー管理4

### 9.3.1 例題(つづき)

#### テストの追加

##### rspecのインストール

rspecがインストールされていない場合はインストールしましょう

`Gemfile`

```rb
・
・
・
group :development, :test do
  ・・・
  gem 'rspec-rails' # 追加
end
```

```sh
$ bundle install
$ rails g rspec:install
Running via Spring preloader in process 35280
      create  .rspec
      create  spec
      create  spec/spec_helper.rb
      create  spec/rails_helper.rb
```

Deviseのテストを行うにはRequest specを利用します。

##### ログアウトした状態でのテスト

ログアウトした状態でのテストは特に何も必要ありません。
いつも通りrequest specを書けば良いです。

`spec/requests/mypage_spec.rb`(新規作成)

```rb
require 'rails_helper'

RSpec.describe 'Mypage', type: :request do
  describe 'GET /mypage' do
    context 'ログインしていない場合' do
      it 'ログイン画面へリダイレクトすること' do
        get mypage_path
        expect(response).to redirect_to(:new_user_session)
      end
    end
  end
end
```


##### ログインした状態でのテスト

ログインしたい状態のテストを行うにはログインした状態を作ってからrequestを投げる必要があります。

ログイン状態を作るための設定を行います。

`spec/rails_helper.rb`

```rb
・
・
・
RSpec.configure do |config|
  ・
  ・
  ・
  # Devise
  config.include Warden::Test::Helpers
  config.before :suite do
    Warden.test_mode!
  end
end
```

specファイルを修正します。

`spec/request/mypage_spec.rb`

```rb
require 'rails_helper'

RSpec.describe 'Mypage', type: :request do
  describe 'GET /mypage' do
    context 'ログインしていない場合' do
      it 'ログイン画面へリダイレクトすること' do
        get mypage_path
        expect(response).to redirect_to(:new_user_session)
      end
    end
    
    context 'ログインしている場合' do
      let(:user) { User.create(email: 'test@example.com', password: 'password') }
      
      before do
        login_as user
      end
      
      it 'マイページが表示されること' do
        get mypage_path
        expect(response.body).to include 'ここはマイページです'
      end
    end
  end
end  
```

これでログインしている時のマイページのテストとログインしていない時のマイページのテストができました。

```sh
$ bundle exec rspec -fd spec/requests/mypage_spec.rb

Mypage
  GET /mypage
    ログインしていない場合
      ログイン画面へリダイレクトすること
    ログインしている場合
      マイページが表示されること

Finished in 0.32214 seconds (files took 2.68 seconds to load)
2 examples, 0 failures
```

このように出力されていればテストに成功しています。


### 9.3.2 問題

それでは、今作成しているECサイトの商品一覧に認証を追加してみましょう。また、新たにユーザのCRUDを作成し認証を追加しましょう。
