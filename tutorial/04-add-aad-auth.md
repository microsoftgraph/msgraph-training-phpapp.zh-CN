<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a3a37-101">在此练习中，你将扩展上一练习中的应用程序，以支持使用 Azure AD 进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="a3a37-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="a3a37-102">这是获取调用 Microsoft Graph 所需的 OAuth 访问令牌所必需的。</span><span class="sxs-lookup"><span data-stu-id="a3a37-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="a3a37-103">在此步骤中，您将 [oauth2-client](https://github.com/thephpleague/oauth2-client) 库集成到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="a3a37-103">In this step you will integrate the [oauth2-client](https://github.com/thephpleague/oauth2-client) library into the application.</span></span>

1. <span data-ttu-id="a3a37-104">打开 PHP 应用程序的根中的 **.env** 文件，将以下代码添加到文件的末尾。</span><span class="sxs-lookup"><span data-stu-id="a3a37-104">Open the **.env** file in the root of your PHP application, and add the following code to the end of the file.</span></span>

    :::code language="ini" source="../demo/graph-tutorial/example.env" range="48-54":::

1. <span data-ttu-id="a3a37-105">替换为 `YOUR_APP_ID_HERE` 应用程序注册门户中的应用程序 ID，并 `YOUR_APP_PASSWORD_HERE` 替换为生成的密码。</span><span class="sxs-lookup"><span data-stu-id="a3a37-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="a3a37-106">如果你使用的是源代码管理（如 git），那么现在应该将文件从源代码管理中排除，以避免意外泄露应用 ID 和 `.env` 密码。</span><span class="sxs-lookup"><span data-stu-id="a3a37-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="a3a37-107">实现登录</span><span class="sxs-lookup"><span data-stu-id="a3a37-107">Implement sign-in</span></span>

1. <span data-ttu-id="a3a37-108">在 **./app/Http/Controllers** 目录中创建一个名为的新文件 `AuthController.php` 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="a3a37-108">Create a new file in the **./app/Http/Controllers** directory named `AuthController.php` and add the following code.</span></span>

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

    <span data-ttu-id="a3a37-109">这将使用两个操作定义控制器： `signin` 和 `callback` 。</span><span class="sxs-lookup"><span data-stu-id="a3a37-109">This defines a controller with two actions: `signin` and `callback`.</span></span>

    <span data-ttu-id="a3a37-110">该操作将生成 Azure AD 登录 URL，保存 OAuth 客户端生成的值，然后将浏览器重定向到 `signin` `state` Azure AD 登录页。</span><span class="sxs-lookup"><span data-stu-id="a3a37-110">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    <span data-ttu-id="a3a37-111">此操作 `callback` 是 Azure 在登录完成后重定向的地方。</span><span class="sxs-lookup"><span data-stu-id="a3a37-111">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="a3a37-112">该操作确保值与保存的值匹配，然后用户获取 Azure 发送的授权代码 `state` ，以请求访问令牌。</span><span class="sxs-lookup"><span data-stu-id="a3a37-112">That action makes sure the `state` value matches the saved value, then users the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="a3a37-113">然后，它会使用临时错误值中的访问令牌重定向回主页。</span><span class="sxs-lookup"><span data-stu-id="a3a37-113">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="a3a37-114">在继续之前，你将使用此验证登录是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="a3a37-114">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="a3a37-115">将路由添加到 **./routes/web.php**。</span><span class="sxs-lookup"><span data-stu-id="a3a37-115">Add the routes to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. <span data-ttu-id="a3a37-116">启动服务器并浏览到 `https://localhost:8000` 。</span><span class="sxs-lookup"><span data-stu-id="a3a37-116">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="a3a37-117">单击登录按钮，应重定向到 `https://login.microsoftonline.com` 。</span><span class="sxs-lookup"><span data-stu-id="a3a37-117">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="a3a37-118">使用 Microsoft 帐户登录。</span><span class="sxs-lookup"><span data-stu-id="a3a37-118">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="a3a37-119">检查同意提示。</span><span class="sxs-lookup"><span data-stu-id="a3a37-119">Examine the consent prompt.</span></span> <span data-ttu-id="a3a37-120">权限列表对应于 **.env** 中配置的权限范围列表。</span><span class="sxs-lookup"><span data-stu-id="a3a37-120">The list of permissions correspond to list of permissions scopes configured in **.env**.</span></span>

    - <span data-ttu-id="a3a37-121">**保持对已授予** 其访问权限的数据的访问权限： () MSAL 请求获取此权限， `offline_access` 以便检索刷新令牌。</span><span class="sxs-lookup"><span data-stu-id="a3a37-121">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="a3a37-122">**登录并读取个人资料： ()** 此权限允许应用获取登录用户的 `User.Read` 配置文件和个人资料照片。</span><span class="sxs-lookup"><span data-stu-id="a3a37-122">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="a3a37-123">**读取邮箱设置： ()** 此权限允许应用读取用户的邮箱设置，包括 `MailboxSettings.Read` 时区和时间格式。</span><span class="sxs-lookup"><span data-stu-id="a3a37-123">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="a3a37-124">**具有对** 日历的完全访问权限： () 此权限允许应用读取用户日历上的事件、添加新事件和修改现有 `Calendars.ReadWrite` 事件。</span><span class="sxs-lookup"><span data-stu-id="a3a37-124">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

1. <span data-ttu-id="a3a37-125">同意请求的权限。</span><span class="sxs-lookup"><span data-stu-id="a3a37-125">Consent to the requested permissions.</span></span> <span data-ttu-id="a3a37-126">浏览器重定向到应用，显示令牌。</span><span class="sxs-lookup"><span data-stu-id="a3a37-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="a3a37-127">获取用户详细信息</span><span class="sxs-lookup"><span data-stu-id="a3a37-127">Get user details</span></span>

<span data-ttu-id="a3a37-128">在此部分中，你将更新方法，以从 `callback` Microsoft Graph 获取用户配置文件。</span><span class="sxs-lookup"><span data-stu-id="a3a37-128">In this section you'll update the `callback` method to get the user's profile from Microsoft Graph.</span></span>

1. <span data-ttu-id="a3a37-129">将以下语句 `use` 添加到行下方 **/app/Http/Controllers/AuthController.php** `namespace App\Http\Controllers;` 的顶部。</span><span class="sxs-lookup"><span data-stu-id="a3a37-129">Add the following `use` statements to the top of **/app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. <span data-ttu-id="a3a37-130">将 `try` 方法中的 `callback` 块替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="a3a37-130">Replace the `try` block in the `callback` method with the following code.</span></span>

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

<span data-ttu-id="a3a37-131">新代码将创建 `Graph` 一个对象，分配访问令牌，然后使用它请求用户配置文件。</span><span class="sxs-lookup"><span data-stu-id="a3a37-131">The new code creates a `Graph` object, assigns the access token, then uses it to request the user's profile.</span></span> <span data-ttu-id="a3a37-132">它会将用户显示名称添加到临时输出中进行测试。</span><span class="sxs-lookup"><span data-stu-id="a3a37-132">It adds the user's display name to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="a3a37-133">存储令牌</span><span class="sxs-lookup"><span data-stu-id="a3a37-133">Storing the tokens</span></span>

<span data-ttu-id="a3a37-134">现在，你可以获取令牌，是时候实现在应用中存储令牌的方法了。</span><span class="sxs-lookup"><span data-stu-id="a3a37-134">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="a3a37-135">由于这是一个示例应用，为了简单起见，你将将它们存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="a3a37-135">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="a3a37-136">实际应用将使用更可靠的安全存储解决方案，如数据库。</span><span class="sxs-lookup"><span data-stu-id="a3a37-136">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="a3a37-137">在名为 **./app** 的目录中创建新目录，然后在名为该目录中创建新文件， `TokenStore` `TokenCache.php` 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="a3a37-137">Create a new directory in the **./app** directory named `TokenStore`, then create a new file in that directory named `TokenCache.php`, and add the following code.</span></span>

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

1. <span data-ttu-id="a3a37-138">将以下语句 `use` 添加到行下方 **./app/Http/Controllers/AuthController.php** `namespace App\Http\Controllers;` 的顶部。</span><span class="sxs-lookup"><span data-stu-id="a3a37-138">Add the following `use` statement to the top of **./app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use App\TokenStore\TokenCache;
    ```

1. <span data-ttu-id="a3a37-139">将 `try` 现有函数中的 `callback` 块替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="a3a37-139">Replace the `try` block in the existing `callback` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="a3a37-140">实现注销</span><span class="sxs-lookup"><span data-stu-id="a3a37-140">Implement sign-out</span></span>

<span data-ttu-id="a3a37-141">测试此新功能之前，请添加一种注销方法。</span><span class="sxs-lookup"><span data-stu-id="a3a37-141">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="a3a37-142">将以下操作添加到 `AuthController` 类。</span><span class="sxs-lookup"><span data-stu-id="a3a37-142">Add the following action to the `AuthController` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. <span data-ttu-id="a3a37-143">将此操作添加到 **./routes/web.php**。</span><span class="sxs-lookup"><span data-stu-id="a3a37-143">Add this action to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. <span data-ttu-id="a3a37-144">重新启动服务器并完成登录过程。</span><span class="sxs-lookup"><span data-stu-id="a3a37-144">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="a3a37-145">你最终应返回主页，但 UI 应更改以指示你已登录。</span><span class="sxs-lookup"><span data-stu-id="a3a37-145">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. <span data-ttu-id="a3a37-147">单击右上角的用户头像以访问 **"注销"** 链接。</span><span class="sxs-lookup"><span data-stu-id="a3a37-147">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="a3a37-148">单击 **"注销** "可重置会话，并返回到主页。</span><span class="sxs-lookup"><span data-stu-id="a3a37-148">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![包含"注销"链接的下拉菜单屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="a3a37-150">刷新令牌</span><span class="sxs-lookup"><span data-stu-id="a3a37-150">Refreshing tokens</span></span>

<span data-ttu-id="a3a37-151">此时，应用程序具有访问令牌，该令牌在 API 调用 `Authorization` 标头中发送。</span><span class="sxs-lookup"><span data-stu-id="a3a37-151">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="a3a37-152">这是允许应用代表用户访问 Microsoft Graph 的令牌。</span><span class="sxs-lookup"><span data-stu-id="a3a37-152">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="a3a37-153">但是，此令牌是短期的。</span><span class="sxs-lookup"><span data-stu-id="a3a37-153">However, this token is short-lived.</span></span> <span data-ttu-id="a3a37-154">令牌在颁发后一小时过期。</span><span class="sxs-lookup"><span data-stu-id="a3a37-154">The token expires an hour after it is issued.</span></span> <span data-ttu-id="a3a37-155">刷新令牌在这里变得有用。</span><span class="sxs-lookup"><span data-stu-id="a3a37-155">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="a3a37-156">刷新令牌允许应用请求新的访问令牌，而无需用户重新登录。</span><span class="sxs-lookup"><span data-stu-id="a3a37-156">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="a3a37-157">更新令牌管理代码以实施令牌刷新。</span><span class="sxs-lookup"><span data-stu-id="a3a37-157">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="a3a37-158">打开 **./app/TokenStore/TokenCache.php，** 将以下函数添加到 `TokenCache` 类中。</span><span class="sxs-lookup"><span data-stu-id="a3a37-158">Open **./app/TokenStore/TokenCache.php** and add the following function to the `TokenCache` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. <span data-ttu-id="a3a37-159">将现有的 `getAccessToken` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="a3a37-159">Replace the existing `getAccessToken` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

<span data-ttu-id="a3a37-160">此方法首先检查访问令牌是否已过期或接近过期。</span><span class="sxs-lookup"><span data-stu-id="a3a37-160">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="a3a37-161">如果是，则使用刷新令牌获取新令牌，然后更新缓存并返回新的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="a3a37-161">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
