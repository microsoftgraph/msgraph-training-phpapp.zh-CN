<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将扩展上一练习中的应用程序，以支持使用 Azure AD 进行身份验证。 这是获取调用 Microsoft Graph 所需的 OAuth 访问令牌所必需的。 在此步骤中，您将 [oauth2-client](https://github.com/thephpleague/oauth2-client) 库集成到应用程序中。

1. 打开 PHP 应用程序的根中的 **.env** 文件，将以下代码添加到文件的末尾。

    :::code language="ini" source="../demo/graph-tutorial/example.env" range="51-57":::

1. 替换为 `YOUR_APP_ID_HERE` 应用程序注册门户中的应用程序 ID，并 `YOUR_APP_SECRET_HERE` 替换为生成的密码。

    > [!IMPORTANT]
    > 如果你使用的是源代码管理（如 git），那么现在应该将文件从源代码管理中排除，以避免意外泄露应用 ID 和 `.env` 密码。

1. 在名为 **./config** 的文件夹中创建新文件 `azure.php` 并添加以下代码。

    :::code language="php" source="../demo/graph-tutorial/config/azure.php":::

## <a name="implement-sign-in"></a>实现登录

1. 在名为 **./app/Http/Controllers** 目录中创建新文件 `AuthController.php` 并添加以下代码。

    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class AuthController extends Controller
    {
      public function signin()
      {
        // Initialize the OAuth client
        $oauthClient = new \League\OAuth2\Client\Provider\GenericProvider([
          'clientId'                => config('azure.appId'),
          'clientSecret'            => config('azure.appSecret'),
          'redirectUri'             => config('azure.redirectUri'),
          'urlAuthorize'            => config('azure.authority').config('azure.authorizeEndpoint'),
          'urlAccessToken'          => config('azure.authority').config('azure.tokenEndpoint'),
          'urlResourceOwnerDetails' => '',
          'scopes'                  => config('azure.scopes')
        ]);

        $authUrl = $oauthClient->getAuthorizationUrl();

        // Save client state so we can validate in callback
        session(['oauthState' => $oauthClient->getState()]);

        // Redirect to AAD signin page
        return redirect()->away($authUrl);
      }

      public function callback(Request $request)
      {
        // Validate state
        $expectedState = session('oauthState');
        $request->session()->forget('oauthState');
        $providedState = $request->query('state');

        if (!isset($expectedState)) {
          // If there is no expected state in the session,
          // do nothing and redirect to the home page.
          return redirect('/');
        }

        if (!isset($providedState) || $expectedState != $providedState) {
          return redirect('/')
            ->with('error', 'Invalid auth state')
            ->with('errorDetail', 'The provided auth state did not match the expected value');
        }

        // Authorization code should be in the "code" query param
        $authCode = $request->query('code');
        if (isset($authCode)) {
          // Initialize the OAuth client
          $oauthClient = new \League\OAuth2\Client\Provider\GenericProvider([
            'clientId'                => config('azure.appId'),
            'clientSecret'            => config('azure.appSecret'),
            'redirectUri'             => config('azure.redirectUri'),
            'urlAuthorize'            => config('azure.authority').config('azure.authorizeEndpoint'),
            'urlAccessToken'          => config('azure.authority').config('azure.tokenEndpoint'),
            'urlResourceOwnerDetails' => '',
            'scopes'                  => config('azure.scopes')
          ]);

          try {
            // Make the token request
            $accessToken = $oauthClient->getAccessToken('authorization_code', [
              'code' => $authCode
            ]);

            // TEMPORARY FOR TESTING!
            return redirect('/')
              ->with('error', 'Access token received')
              ->with('errorDetail', $accessToken->getToken());
          }
          catch (League\OAuth2\Client\Provider\Exception\IdentityProviderException $e) {
            return redirect('/')
              ->with('error', 'Error requesting access token')
              ->with('errorDetail', $e->getMessage());
          }
        }

        return redirect('/')
          ->with('error', $request->query('error'))
          ->with('errorDetail', $request->query('error_description'));
      }
    }
    ```

    这将使用两个操作定义控制器： `signin` 和 `callback` 。

    此操作将生成 Azure AD 登录 URL，保存 OAuth 客户端生成的值，然后将浏览器重定向到 `signin` `state` Azure AD 登录页。

    此操作 `callback` 是 Azure 在登录完成后重定向的地方。 该操作确保值与保存的值匹配，然后用户获取 Azure 发送的授权代码， `state` 以请求访问令牌。 然后，它会使用临时错误值中的访问令牌重定向回主页。 在继续之前，你将使用此验证登录是否正常工作。

1. 将路由添加到 **./routes/web.php**。

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. 启动服务器并浏览到 `https://localhost:8000` 。 单击登录按钮，应重定向到 `https://login.microsoftonline.com` 。 使用 Microsoft 帐户登录。

1. 检查同意提示。 权限列表对应于 **.env** 中配置的权限范围列表。

    - **保持对已授予** 其访问权限的数据的访问权限： () MSAL 请求获取此权限， `offline_access` 以便检索刷新令牌。
    - **登录并读取个人资料： ()** 此权限允许应用获取登录用户的 `User.Read` 配置文件和个人资料照片。
    - **读取邮箱设置： ()** 此权限允许应用读取用户的邮箱设置，包括时区 `MailboxSettings.Read` 和时间格式。
    - **具有对** 日历的完全访问权限： () 此权限允许应用读取用户日历上的事件、添加新事件和 `Calendars.ReadWrite` 修改现有事件。

1. 同意请求的权限。 浏览器重定向到应用，显示令牌。

### <a name="get-user-details"></a>获取用户详细信息

在此部分中，你将更新方法，以从 `callback` Microsoft Graph 获取用户配置文件。

1. 将以下语句 `use` 添加到行下方 **/app/Http/Controllers/AuthController.php** `namespace App\Http\Controllers;` 的顶部。

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. 将 `try` 方法中的 `callback` 块替换为以下代码。

    ```php
    try {
      // Make the token request
      $accessToken = $oauthClient->getAccessToken('authorization_code', [
        'code' => $authCode
      ]);

      $graph = new Graph();
      $graph->setAccessToken($accessToken->getToken());

      $user = $graph->createRequest('GET', '/me?$select=displayName,mail,mailboxSettings,userPrincipalName')
        ->setReturnType(Model\User::class)
        ->execute();

      // TEMPORARY FOR TESTING!
      return redirect('/')
        ->with('error', 'Access token received')
        ->with('errorDetail', 'User:'.$user->getDisplayName().', Token:'.$accessToken->getToken());
    }
    ```

新代码将创建 `Graph` 一个对象，分配访问令牌，然后使用它请求用户配置文件。 它会将用户显示名称添加到临时输出中进行测试。

## <a name="storing-the-tokens"></a>存储令牌

现在，你可以获取令牌，是时候实现在应用中存储令牌的方法了。 由于这是一个示例应用，为了简单起见，你将将它们存储在会话中。 实际应用将使用更可靠的安全存储解决方案，如数据库。

1. 在名为 **./app** 的目录中创建新目录，然后在名为该目录中创建新文件， `TokenStore` `TokenCache.php` 并添加以下代码。

    ```php
    <?php

    namespace App\TokenStore;

    class TokenCache {
      public function storeTokens($accessToken, $user) {
        session([
          'accessToken' => $accessToken->getToken(),
          'refreshToken' => $accessToken->getRefreshToken(),
          'tokenExpires' => $accessToken->getExpires(),
          'userName' => $user->getDisplayName(),
          'userEmail' => null !== $user->getMail() ? $user->getMail() : $user->getUserPrincipalName(),
          'userTimeZone' => $user->getMailboxSettings()->getTimeZone()
        ]);
      }

      public function clearTokens() {
        session()->forget('accessToken');
        session()->forget('refreshToken');
        session()->forget('tokenExpires');
        session()->forget('userName');
        session()->forget('userEmail');
        session()->forget('userTimeZone');
      }

      public function getAccessToken() {
        // Check if tokens exist
        if (empty(session('accessToken')) ||
            empty(session('refreshToken')) ||
            empty(session('tokenExpires'))) {
          return '';
        }

        return session('accessToken');
      }
    }
    ```

1. 将以下语句 `use` 添加到行下方 **./app/Http/Controllers/AuthController.php** `namespace App\Http\Controllers;` 的顶部。

    ```php
    use App\TokenStore\TokenCache;
    ```

1. 将 `try` 现有函数中的 `callback` 块替换为以下内容。

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a>实现注销

测试此新功能之前，请添加一种注销方法。

1. 将以下操作添加到 `AuthController` 类。

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. 将此操作添加到 **./routes/web.php**。

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. 重新启动服务器并完成登录过程。 你最终应返回主页，但 UI 应更改以指示你已登录。

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. 单击右上角的用户头像以访问 **"注销"** 链接。 单击 **"注销** "可重置会话，并返回到主页。

    ![包含"注销"链接的下拉菜单屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>刷新令牌

此时，应用程序具有访问令牌，该令牌在 API 调用 `Authorization` 标头中发送。 这是允许应用代表用户访问 Microsoft Graph 的令牌。

但是，此令牌是短期的。 令牌在颁发后一小时过期。 此时刷新令牌将变得有用。 刷新令牌允许应用请求新的访问令牌，而无需用户重新登录。 更新令牌管理代码以实施令牌刷新。

1. 打开 **./app/TokenStore/TokenCache.php，** 将以下函数添加到 `TokenCache` 类中。

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. 将现有的 `getAccessToken` 函数替换为以下内容。

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

此方法首先检查访问令牌是否已过期或接近过期。 如果是，则使用刷新令牌获取新令牌，然后更新缓存并返回新的访问令牌。
