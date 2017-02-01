## Authentication

* [OAuth2](auth.md#authentication-with-oauth2)
* [Basic Auth](auth.md#authentication-with-basic-http)

--

### Authentication with OAuth2

There is [finagle-oauth2](https://github.com/finagle/finagle-oauth2) server-side provider that is
supported in Finch via the `finch-oauth2` package:

*Authorize*
```tut:silent
import io.finch._
import io.finch.oauth2._
import com.twitter.finagle.oauth2._
import com.twitter.util.Future
import java.util.Date

class MockDataHandler extends DataHandler[Int] {

  def validateClient(clientId: String, clientSecret: String, grantType: String) =
    Future.value(false)

  def findUser(username: String, password: String): Future[Option[Int]] =
    Future.value(None)

  def createAccessToken(authInfo: AuthInfo[Int]) =
    Future.value(AccessToken("", Some(""), Some(""), Some(0L), new Date()))

  def findAuthInfoByCode(code: String): Future[Option[AuthInfo[Int]]] =
    Future.value(None)

  def findAuthInfoByRefreshToken(refreshToken: String): Future[Option[AuthInfo[Int]]] =
    Future.value(None)

  def findClientUser(clientId: String, clientSecret: String, scope: Option[String]): Future[Option[Int]] =
    Future.value(None)

  def findAccessToken(token: String): Future[Option[AccessToken]] =
    Future.value(None)

  def findAuthInfoByAccessToken(accessToken: AccessToken): Future[Option[AuthInfo[Int]]] =
    Future.value(None)
  
  def getStoredAccessToken(authInfo: AuthInfo[Int]): Future[Option[AccessToken]] =
    Future.value(None)

  def refreshAccessToken(authInfo: AuthInfo[Int], refreshToken: String): Future[AccessToken] =
    Future.value(AccessToken("", Some(""), Some(""), Some(0L), new Date()))
}

val dataHandler: DataHandler[Int] = new MockDataHandler()
val auth: Endpoint[AuthInfo[Int]] = authorize(dataHandler)
val e: Endpoint[Int] = get("user" :: auth) { ai: AuthInfo[Int] => Ok(ai.user) }
```

*Issue Access Token*
```scala
import io.finch._
import io.finch.oauth2._
import com.twitter.finagle.oauth2._
import org.mockito.Mockito._

val token: Endpoint[GrandHandlerResult] = issueAccessToken(dataHandler)
```

Note that both `token` and `authorize` may throw `com.twitter.finagle.oauth2.OAuthError`, which is
already _handled_ by a returned endpoint but needs to be serialized. This means you might want to
include its serialization logic into an instance of `EncodeResponse[Exception]`.

### Authentication with Basic HTTP

[Basic HTTP Auth](http://en.wikipedia.org/wiki/Basic_access_authentication) is implemented as the
`BasicAuth` combinator available in `finch-core`.

```tut:silent
import io.finch._

val basicAuth: BasicAuth = BasicAuth("realm") { (user, password) =>
  Future { user == "user" && password == "password" }
}
val e: Endpoint[String] = basicAuth(Endpoint.liftOutput(Ok("secret place")))
```

--
Read Next: [JSON](json.md)
