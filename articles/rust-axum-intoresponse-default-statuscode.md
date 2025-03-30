---
title: "AxumでハンドラーがResult::Errを返してもステータスコードは200 OKになりうる"
emoji: "🕌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [Rust, Axum]
published: false
---

# はじめに
Axumを使っていて少し意外だったことを紹介するよ。

# 前提
以下はTodoアプリのコードの一部で、Routerにパスと対応するリクエストを処理するハンドラーを登録して、リクエストが来たらハンドラーの処理を呼び出すというもの。(.with_stateでDBを注入しているが今回の主題ではない。)

Axumのハンドラーとは`todos_index`,`todos_read`, `todos_create`, `todos_update`, `todos_delete`のようにアプリケーションロジックを記述する関数などである。(正確なことは[公式ドキュメント](https://docs.rs/axum/latest/axum/handler/index.html)を見てほしい。関数だけじゃなくてStringを返すこともできる。)

同様にハンドラーとして登録できる関数は`Result`を返すこともできるのだが、今回の記事では、ハンドラーが`Result::Err`を返しても、200 OKになりうることを紹介する。Errならいい感じに404 Not Foundとか500 Internal Server Errorを返してくれるのかと思っていたが、そんな都合のいいことはなかった。

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let db = memory_db::DB::default();
    let app = Router::new()
        .route("/", get(todos_index))
        .route("/todos", get(todos_read).post(todos_create))
        .route("/todos/:id", patch(todos_update).delete(todos_delete))
        .with_state(db);

    let listener = tokio::net::TcpListener::bind("localhost:3000").await?;
        
    axum::serve(listener, app).await?;
    Ok(())
}
// Htmlを返すハンドラー
async fn todos_index() -> Html<&'static str> {
    Html(include_str!("../index.html"))
}
...
```
# 結論
先に結論をいうと、Errを返すとしてもどんなエラーなのか(500なのか、404なのか)は要件次第であり、それに伴い、どんなレスポンスを返すかも要件次第であるため、Errだからって一概にステータスコードを決められないよね、という話。

## コードを追ってみる

実際に`Result::Err`を返すハンドラーを作って試してみよう。`result_test`はパスパラメータに1が来たらOKを返すが、それ以外はErrを返すハンドラーである。

```rust
async fn result_test(Path(r): Path<u8>) -> Result<String, String> {
    if r == 1 {
        Ok("OK".to_string())
    } else {
        Err("Error".to_string())
    }
}
```

### Result::Okの場合
curlコマンドでリクエストを投げてみる。
```bash
curl localhost:3000/result/1 -v
```
![Result OK](/images/rust-axum-intoresponse-default-statuscode/Result_String_OK.png)
OK200 OKが返ってくる。

### Result::Errの場合
```
curl localhost:3000/result/2 -v
```
![Result Err](/images/rust-axum-intoresponse-default-statuscode/Result_String_Error.png)
やっぱり200 OKが返ってくる。

## 何が起きているのか

関数に対するHandlerトレイトの実装は以下のようになっている。

```rust
impl<F, Fut, Res, S> Handler<((),), S> for F
where
    F: FnOnce() -> Fut + Clone + Send + 'static,
    Fut: Future<Output = Res> + Send,
    Res: IntoResponse,
{
    type Future = Pin<Box<dyn Future<Output = Response> + Send>>;

    fn call(self, _req: Request, _state: S) -> Self::Future {
        Box::pin(async move { self().await.into_response() })
    }
}
```
注目してほしいのは`Res: IntoResponse`の部分である。これは、ハンドラーの戻り値がIntoResponseトレイトを実装している必要があることを示している。

では、Resultに対するIntoResponseトレイトはどのように実装されているのか見てみよう。
```rust
impl<T, E> IntoResponse for Result<T, E>
where
    T: IntoResponse,
    E: IntoResponse,
{
    fn into_response(self) -> Response {
        match self {
            Ok(value) => value.into_response(),
            Err(err) => err.into_response(),
        }
    }
}
```
OkでもErrでも`into_response()`を呼び出していることがわかる。つまり、Resultの中身がIntoResponseトレイトを実装していれば、OkでもErrでも`into_response()`をそのまま返すということだ。
なので、ステータスコードはErrの中身の`into_response()`次第、ということになる。

Stringに対するIntoResponseトレイトも見てみよう。
```rust
impl IntoResponse for String {
    fn into_response(self) -> Response {
        Cow::<'static, str>::Owned(self).into_response()
    }
}
impl IntoResponse for Cow<'static, str> {
    fn into_response(self) -> Response {
        let mut res = Body::from(self).into_response();
        res.headers_mut().insert(
            header::CONTENT_TYPE,
            HeaderValue::from_static(mime::TEXT_PLAIN_UTF_8.as_ref()),
        );
        res
    }
}
impl IntoResponse for Body {
    fn into_response(self) -> Response {
        Response::new(self)
    }
}
#[inline]
pub fn new(body: T) -> Response<T> {
    Response {
        head: Parts::new(),
        body,
    }
}
impl Parts {
    /// Creates a new default instance of `Parts`
    fn new() -> Parts {
        Parts {
            status: StatusCode::default(),
            version: Version::default(),
            headers: HeaderMap::default(),
            extensions: Extensions::default(),
            _priv: (),
        }
    }
}
impl Default for StatusCode {
    #[inline]
    fn default() -> StatusCode {
        StatusCode::OK // 200 OKを返している
    }
}
```
たらい回しにされているが、Stringの`into_response()`はCowの`into_response()`を呼び出し、CowのはBodyのを、Bodyのは`Response::new()`を呼び出していることがわかる。
最終的に呼び出された`Response::new()`ではデフォルトの200 OKをステータスコードとして返すようになっている。

長くなったが、ResultのErrを返すと中身の`into_response()`を返す。今回中身はStringなので200 OKを返した、ということ。

# 結論
Errでもどんなエラーなのか(500なのか、404なのか)は要件次第であり、それに伴い、どんなレスポンスを返すかも要件次第である。それゆえErrだからといって一概にステータスコードを決められるわけではない。Errの中身(Errの`into_response()`)を見て決定する、というAxumの方針は適切そうだ。
