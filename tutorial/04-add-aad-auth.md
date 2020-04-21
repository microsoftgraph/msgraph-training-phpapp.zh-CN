<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="61190-101">在本练习中，你将扩展上一练习中的应用程序，以支持 Azure AD 的身份验证。</span><span class="sxs-lookup"><span data-stu-id="61190-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="61190-102">若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph，这是必需的。</span><span class="sxs-lookup"><span data-stu-id="61190-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="61190-103">在此步骤中，将[oauth2-客户端](https://github.com/thephpleague/oauth2-client)库集成到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="61190-103">In this step you will integrate the [oauth2-client](https://github.com/thephpleague/oauth2-client) library into the application.</span></span>

1. <span data-ttu-id="61190-104">打开 PHP `.env`应用程序的根目录中的文件，并将以下代码添加到该文件的末尾。</span><span class="sxs-lookup"><span data-stu-id="61190-104">Open the `.env` file in the root of your PHP application, and add the following code to the end of the file.</span></span>

    :::code language="ini" source="../demo/graph-tutorial/.env.example" id="OAuthSettingsSnippet":::

1. <span data-ttu-id="61190-105">将`YOUR_APP_ID_HERE`替换为应用程序注册门户中的应用程序 ID， `YOUR_APP_PASSWORD_HERE`并将替换为您生成的密码。</span><span class="sxs-lookup"><span data-stu-id="61190-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="61190-106">如果您使用的是源代码管理（如 git），现在可以从源代码管理中排除`.env`该文件，以避免无意中泄漏您的应用程序 ID 和密码。</span><span class="sxs-lookup"><span data-stu-id="61190-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="61190-107">实施登录</span><span class="sxs-lookup"><span data-stu-id="61190-107">Implement sign-in</span></span>

1. <span data-ttu-id="61190-108">在 **/app/Http/Controllers**目录中创建一个名为`AuthController.php`的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="61190-108">Create a new file in the **./app/Http/Controllers** directory named `AuthController.php` and add the following code.</span></span>

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
          'clientId'                => env('OAUTH_APP_ID'),
          'clientSecret'            => env('OAUTH_APP_PASSWORD'),
          'redirectUri'             => env('OAUTH_REDIRECT_URI'),
          'urlAuthorize'            => env('OAUTH_AUTHORITY').env('OAUTH_AUTHORIZE_ENDPOINT'),
          'urlAccessToken'          => env('OAUTH_AUTHORITY').env('OAUTH_TOKEN_ENDPOINT'),
          'urlResourceOwnerDetails' => '',
          'scopes'                  => env('OAUTH_SCOPES')
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
            'clientId'                => env('OAUTH_APP_ID'),
            'clientSecret'            => env('OAUTH_APP_PASSWORD'),
            'redirectUri'             => env('OAUTH_REDIRECT_URI'),
            'urlAuthorize'            => env('OAUTH_AUTHORITY').env('OAUTH_AUTHORIZE_ENDPOINT'),
            'urlAccessToken'          => env('OAUTH_AUTHORITY').env('OAUTH_TOKEN_ENDPOINT'),
            'urlResourceOwnerDetails' => '',
            'scopes'                  => env('OAUTH_SCOPES')
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

    <span data-ttu-id="61190-109">这将定义包含两个操作的`signin`控制器`callback`：和。</span><span class="sxs-lookup"><span data-stu-id="61190-109">This defines a controller with two actions: `signin` and `callback`.</span></span>

    <span data-ttu-id="61190-110">该`signin`操作将生成 Azure AD 登录 URL，保存 OAuth `state`客户端生成的值，然后将浏览器重定向到 Azure AD 登录页。</span><span class="sxs-lookup"><span data-stu-id="61190-110">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    <span data-ttu-id="61190-111">`callback`操作是 Azure 在登录完成后重定向的位置。</span><span class="sxs-lookup"><span data-stu-id="61190-111">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="61190-112">该操作确保`state`值与保存的值相匹配，然后由 Azure 发送的授权代码请求访问令牌。</span><span class="sxs-lookup"><span data-stu-id="61190-112">That action makes sure the `state` value matches the saved value, then users the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="61190-113">然后，它使用临时错误值中的访问令牌重定向回主页。</span><span class="sxs-lookup"><span data-stu-id="61190-113">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="61190-114">在继续操作之前，你将使用此操作来验证登录是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="61190-114">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="61190-115">将路由添加到 **/routes/web.php**。</span><span class="sxs-lookup"><span data-stu-id="61190-115">Add the routes to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. <span data-ttu-id="61190-116">启动服务器并浏览到`https://localhost:8000`。</span><span class="sxs-lookup"><span data-stu-id="61190-116">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="61190-117">单击 "登录" 按钮，您应会被重定向`https://login.microsoftonline.com`到。</span><span class="sxs-lookup"><span data-stu-id="61190-117">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="61190-118">使用你的 Microsoft 帐户登录，并同意请求的权限。</span><span class="sxs-lookup"><span data-stu-id="61190-118">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="61190-119">浏览器重定向到应用程序，并显示令牌。</span><span class="sxs-lookup"><span data-stu-id="61190-119">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="61190-120">获取用户详细信息</span><span class="sxs-lookup"><span data-stu-id="61190-120">Get user details</span></span>

<span data-ttu-id="61190-121">在本节中，您将更新`callback`方法，以从 Microsoft Graph 获取用户的配置文件。</span><span class="sxs-lookup"><span data-stu-id="61190-121">In this section you'll update the `callback` method to get the user's profile from Microsoft Graph.</span></span>

1. <span data-ttu-id="61190-122">将以下`use`语句添加到 **/app/Http/Controllers/AuthController.php**顶部的`namespace App\Http\Controllers;`行下方。</span><span class="sxs-lookup"><span data-stu-id="61190-122">Add the following `use` statements to the top of **/app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. <span data-ttu-id="61190-123">将`callback`方法`try`中的块替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="61190-123">Replace the `try` block in the `callback` method with the following code.</span></span>

    ```php
    try {
      // Make the token request
      $accessToken = $oauthClient->getAccessToken('authorization_code', [
        'code' => $authCode
      ]);

      $graph = new Graph();
      $graph->setAccessToken($accessToken->getToken());

      $user = $graph->createRequest('GET', '/me')
        ->setReturnType(Model\User::class)
        ->execute();

      // TEMPORARY FOR TESTING!
      return redirect('/')
        ->with('error', 'Access token received')
        ->with('errorDetail', 'User:'.$user->getDisplayName().', Token:'.$accessToken->getToken());
    }
    ```

<span data-ttu-id="61190-124">新代码创建一个`Graph`对象，分配访问令牌，然后使用该令牌请求用户的配置文件。</span><span class="sxs-lookup"><span data-stu-id="61190-124">The new code creates a `Graph` object, assigns the access token, then uses it to request the user's profile.</span></span> <span data-ttu-id="61190-125">它将用户的显示名称添加到临时输出中进行测试。</span><span class="sxs-lookup"><span data-stu-id="61190-125">It adds the user's display name to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="61190-126">存储令牌</span><span class="sxs-lookup"><span data-stu-id="61190-126">Storing the tokens</span></span>

<span data-ttu-id="61190-127">现在，您可以获取令牌，现在是时候实现将它们存储在应用程序中的方法了。</span><span class="sxs-lookup"><span data-stu-id="61190-127">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="61190-128">由于这是一个示例应用程序，因此为简单起见，你将把它们存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="61190-128">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="61190-129">实际应用程序将使用更可靠的安全存储解决方案，就像数据库一样。</span><span class="sxs-lookup"><span data-stu-id="61190-129">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="61190-130">在 **/app**目录中创建一个名为`TokenStore`的新目录，然后在名为`TokenCache.php`的该目录中创建一个新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="61190-130">Create a new directory in the **./app** directory named `TokenStore`, then create a new file in that directory named `TokenCache.php`, and add the following code.</span></span>

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
          'userEmail' => null !== $user->getMail() ? $user->getMail() : $user->getUserPrincipalName()
        ]);
      }

      public function clearTokens() {
        session()->forget('accessToken');
        session()->forget('refreshToken');
        session()->forget('tokenExpires');
        session()->forget('userName');
        session()->forget('userEmail');
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

1. <span data-ttu-id="61190-131">将以下`use`语句添加到位于 **/app/Http/Controllers/AuthController.php**顶部的`namespace App\Http\Controllers;`行下方。</span><span class="sxs-lookup"><span data-stu-id="61190-131">Add the following `use` statement to the top of **./app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use App\TokenStore\TokenCache;
    ```

1. <span data-ttu-id="61190-132">将现有`try` `callback`函数中的块替换为以下项。</span><span class="sxs-lookup"><span data-stu-id="61190-132">Replace the `try` block in the existing `callback` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="61190-133">实现注销</span><span class="sxs-lookup"><span data-stu-id="61190-133">Implement sign-out</span></span>

<span data-ttu-id="61190-134">在测试此新功能之前，请添加一种注销方式。</span><span class="sxs-lookup"><span data-stu-id="61190-134">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="61190-135">将以下操作添加到`AuthController`类中。</span><span class="sxs-lookup"><span data-stu-id="61190-135">Add the following action to the `AuthController` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. <span data-ttu-id="61190-136">将此操作添加到 **/routes/web.php**。</span><span class="sxs-lookup"><span data-stu-id="61190-136">Add this action to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. <span data-ttu-id="61190-137">重新启动服务器并完成登录过程。</span><span class="sxs-lookup"><span data-stu-id="61190-137">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="61190-138">您应该最后返回到主页，但 UI 应更改以指示您已登录。</span><span class="sxs-lookup"><span data-stu-id="61190-138">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. <span data-ttu-id="61190-140">单击右上角的用户头像以访问 "**注销**" 链接。</span><span class="sxs-lookup"><span data-stu-id="61190-140">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="61190-141">单击 "**注销**" 重置会话并返回到主页。</span><span class="sxs-lookup"><span data-stu-id="61190-141">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![带有 "注销" 链接的下拉菜单的屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="61190-143">刷新令牌</span><span class="sxs-lookup"><span data-stu-id="61190-143">Refreshing tokens</span></span>

<span data-ttu-id="61190-144">此时，您的应用程序具有访问令牌，该令牌是在 API `Authorization`调用的标头中发送的。</span><span class="sxs-lookup"><span data-stu-id="61190-144">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="61190-145">这是允许应用代表用户访问 Microsoft Graph 的令牌。</span><span class="sxs-lookup"><span data-stu-id="61190-145">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="61190-146">但是，此令牌的生存期较短。</span><span class="sxs-lookup"><span data-stu-id="61190-146">However, this token is short-lived.</span></span> <span data-ttu-id="61190-147">令牌在发出后会过期一小时。</span><span class="sxs-lookup"><span data-stu-id="61190-147">The token expires an hour after it is issued.</span></span> <span data-ttu-id="61190-148">这就是刷新令牌变得有用的地方。</span><span class="sxs-lookup"><span data-stu-id="61190-148">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="61190-149">刷新令牌允许应用在不要求用户重新登录的情况下请求新的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="61190-149">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="61190-150">更新令牌管理代码以实现令牌刷新。</span><span class="sxs-lookup"><span data-stu-id="61190-150">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="61190-151">打开 **/app/TokenStore/TokenCache.php** ，并将以下函数添加到`TokenCache`类中。</span><span class="sxs-lookup"><span data-stu-id="61190-151">Open **./app/TokenStore/TokenCache.php** and add the following function to the `TokenCache` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. <span data-ttu-id="61190-152">将现有的 `getAccessToken` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="61190-152">Replace the existing `getAccessToken` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

<span data-ttu-id="61190-153">此方法首先检查访问令牌是否已过期或接近即将过期。</span><span class="sxs-lookup"><span data-stu-id="61190-153">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="61190-154">如果是，则它使用刷新令牌获取新令牌，然后更新缓存并返回新的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="61190-154">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
