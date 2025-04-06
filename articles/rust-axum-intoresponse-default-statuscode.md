---
title: "AxumでハンドラーがResult::Errを返してもステータスコードは200 OKになりうる"
emoji: "🕌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [Rust, Axum]
published: true
---

# はじめに
結論に自分でも呑み込めていない部分があるけどとりあえず放出。
Axumを使っていて少し意外だったことを紹介するよ。意外と言っても僕のAPI設計に対する無理解から来ていたことで

# 前提
以下はTodoアプリのコードの一部で、Routerにパスと対応するリクエストを処理するハンドラーを登録して、リクエストが来たらハンドラーの処理を呼び出す。(.with_stateでDBを注入しているが今回の主題ではない。ハンドラー内でDBを使えるようにするためのもの。)

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
// DBからTodoを取得してJsonで返すハンドラー
async fn todos_read(State(db): State<DB>) -> impl IntoResponse {
    let todos = db.read().unwrap();
    let todos = todos.values().cloned().collect::<Vec<Todo>>();
    Json(todos)
}
...
```
Axumのハンドラーとは`todos_index`,`todos_read`のようにリクエストを受け取るコントローラー層の関数（など）である。(正確な定義は[公式ドキュメント](https://docs.rs/axum/latest/axum/handler/index.html)にある。)

ハンドラーとして登録できる関数は`Result`を返すこともできるが、今回の記事では、ハンドラーが`Result::Err`を返しても、200 OKになりうることを紹介する。Errならいい感じに404 Not Foundとか500 Internal Server Errorを返してくれるのかと思っていたが、必ずしもエラー系のステータスコードが返るわけではなかった。

# 結論
先に結論を言うと、Errを返すとしてもどんなエラーなのか(500なのか、404なのか)は要件次第であり、それに伴い、どんなレスポンスを返すかも要件次第であるため、Errだからって一概にステータスコードを決められないよね、という話(だと思う)。

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
たらい回しにされているが、String → Cow → Body → Responseを呼び出していることがわかる。
最終的に呼び出された`Response::new()`ではステータスコードをデフォルトで200 OKにして返すようになっており、Stringへのinto_response()は200 OKを返すことがわかる。
長くなったが、ハンドラーがResult::Errを返すとErrの中身に対する`into_response()`の結果が返る。今回中身はStringなので200 OKが返ってきた、ということ。

```rust: StringのIntoResponse実装
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


# 結論
Errでもどんなエラーなのか(500なのか、404なのか)、どんなレスポンスを返すかは要件次第。それゆえErrだからといって一概にステータスコードを決められるわけではない。Errの中身(Errの`into_response()`)を見て決定する、というAxumの方針はしょうがないのだと思う。
常にステータスコードを指定させる設計にするといいのだろうか？

<!-- # おまけ
明示的にステータスコードを指定してみた。
```rust
async fn result_test(Path(r): Path<u8>) -> Result<(StatusCode,String), (StatusCode, String)> {
    if r == 1 {
        Ok((StatusCode::OK,"OK".to_string()))
    } else {
        Err((StatusCode::NOT_FOUND, "Error".to_string()))
    }
}
``` -->

