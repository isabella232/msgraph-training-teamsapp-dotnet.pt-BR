<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="47284-101">Neste exercício, você estenderá o aplicativo do exercício anterior para suportar a autenticação de logon único com o Azure AD.</span><span class="sxs-lookup"><span data-stu-id="47284-101">In this exercise you will extend the application from the previous exercise to support single sign-on authentication with Azure AD.</span></span> <span data-ttu-id="47284-102">Isso é necessário para obter o token de acesso OAuth necessário para chamar a API do Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="47284-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="47284-103">Nesta etapa, você irá configurar a biblioteca [Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) .</span><span class="sxs-lookup"><span data-stu-id="47284-103">In this step you will configure the [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) library.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="47284-104">Para evitar o armazenamento da ID do aplicativo e o segredo na fonte, você usará o [Gerenciador de segredo do .net](/aspnet/core/security/app-secrets) para armazenar esses valores.</span><span class="sxs-lookup"><span data-stu-id="47284-104">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="47284-105">O gerente secreto é apenas para fins de desenvolvimento, os aplicativos de produção devem usar um Gerenciador de segredo confiável para armazenar segredos.</span><span class="sxs-lookup"><span data-stu-id="47284-105">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

1. <span data-ttu-id="47284-106">Abra **./appsettings.jsem** e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="47284-106">Open **./appsettings.json** and replace its contents with the following.</span></span>

    :::code language="json" source="../demo/GraphTutorial/appsettings.example.json" highlight="2-8":::

1. <span data-ttu-id="47284-107">Abra a CLI no diretório onde o **GraphTutorial. csproj** está localizado e execute os seguintes comandos, substituindo `YOUR_APP_ID` pela ID do aplicativo do portal do Azure e `YOUR_APP_SECRET` com o segredo do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="47284-107">Open your CLI in the directory where **GraphTutorial.csproj** is located, and run the following commands, substituting `YOUR_APP_ID` with your application ID from the Azure portal, and `YOUR_APP_SECRET` with your application secret.</span></span>

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="47284-108">Implementar logon</span><span class="sxs-lookup"><span data-stu-id="47284-108">Implement sign-in</span></span>

<span data-ttu-id="47284-109">Primeiro, implemente o logon único no código JavaScript do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="47284-109">First, implement single sign-on in the app's JavaScript code.</span></span> <span data-ttu-id="47284-110">Você usará o [SDK do JavaScript do Microsoft Teams](/javascript/api/overview/msteams-client) para obter um token de acesso que permite que o código JavaScript em execução no cliente do teams faça chamadas AJAX para a API Web que você implementará mais tarde.</span><span class="sxs-lookup"><span data-stu-id="47284-110">You will use the [Microsoft Teams JavaScript SDK](/javascript/api/overview/msteams-client) to get an access token which allows the JavaScript code running in the Teams client to make AJAX calls to Web API you will implement later.</span></span>

1. <span data-ttu-id="47284-111">Abra **./pages/index.cshtml** e adicione o código a seguir dentro da `<script>` marca.</span><span class="sxs-lookup"><span data-stu-id="47284-111">Open **./Pages/Index.cshtml** and add the following code inside the `<script>` tag.</span></span>

    ```javascript
    (function () {
      if (microsoftTeams) {
        microsoftTeams.initialize();

        microsoftTeams.authentication.getAuthToken({
          successCallback: (token) => {
            // TEMPORARY: Display the access token for debugging
            $('#tab-container').empty();

            $('<code/>', {
              text: token,
              style: 'word-break: break-all;'
            }).appendTo('#tab-container');
          },
          failureCallback: (error) => {
            renderError(error);
          }
        });
      }
    })();

    function renderError(error) {
      $('#tab-container').empty();

      $('<h1/>', {
        text: 'Error'
      }).appendTo('#tab-container');

      $('<code/>', {
        text: JSON.stringify(error, Object.getOwnPropertyNames(error)),
        style: 'word-break: break-all;'
      }).appendTo('#tab-container');
    }
    ```

    <span data-ttu-id="47284-112">Isso chama o `microsoftTeams.authentication.getAuthToken` para autenticar silenciosamente como o usuário conectado ao Teams.</span><span class="sxs-lookup"><span data-stu-id="47284-112">This calls the `microsoftTeams.authentication.getAuthToken` to silently authenticate as the user that is signed in to Teams.</span></span> <span data-ttu-id="47284-113">Normalmente, não há nenhum prompt de interface do usuário envolvido, a menos que o usuário tenha que dar o consentimento.</span><span class="sxs-lookup"><span data-stu-id="47284-113">There is typically not any UI prompts involved, unless the user has to consent.</span></span> <span data-ttu-id="47284-114">O código, em seguida, exibe o token na guia.</span><span class="sxs-lookup"><span data-stu-id="47284-114">The code then displays the token in the tab.</span></span>

1. <span data-ttu-id="47284-115">Salve suas alterações e inicie o aplicativo executando o seguinte comando em sua CLI.</span><span class="sxs-lookup"><span data-stu-id="47284-115">Save your changes and start your application by running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

    > [!IMPORTANT]
    > <span data-ttu-id="47284-116">Se você tiver reiniciado o ngrok e sua URL do ngrok tiver mudado, certifique-se de atualizar o valor de ngrok no seguinte local **antes** de testar.</span><span class="sxs-lookup"><span data-stu-id="47284-116">If you have restarted ngrok and your ngrok URL has changed, be sure to update the ngrok value in the following place **before** you test.</span></span>
    >
    > - <span data-ttu-id="47284-117">O URI de redirecionamento em seu registro de aplicativo</span><span class="sxs-lookup"><span data-stu-id="47284-117">The redirect URI in your app registration</span></span>
    > - <span data-ttu-id="47284-118">O URI da ID do aplicativo em seu registro de aplicativo</span><span class="sxs-lookup"><span data-stu-id="47284-118">The application ID URI in your app registration</span></span>
    > - <span data-ttu-id="47284-119">`contentUrl` em manifest.jsem</span><span class="sxs-lookup"><span data-stu-id="47284-119">`contentUrl` in manifest.json</span></span>
    > - <span data-ttu-id="47284-120">`validDomains` em manifest.jsem</span><span class="sxs-lookup"><span data-stu-id="47284-120">`validDomains` in manifest.json</span></span>
    > - <span data-ttu-id="47284-121">`resource` em manifest.jsem</span><span class="sxs-lookup"><span data-stu-id="47284-121">`resource` in manifest.json</span></span>

1. <span data-ttu-id="47284-122">Crie um arquivo ZIP com **manifest.jsem**, **color.png** e **outline.png**.</span><span class="sxs-lookup"><span data-stu-id="47284-122">Create a ZIP file with **manifest.json**, **color.png**, and **outline.png**.</span></span>

1. <span data-ttu-id="47284-123">No Microsoft Teams, selecione **aplicativos** na barra esquerda, selecione **carregar um aplicativo personalizado** e, em seguida, selecione **carregar para mim ou minhas equipes**.</span><span class="sxs-lookup"><span data-stu-id="47284-123">In Microsoft Teams, select **Apps** in the left-hand bar, select **Upload a custom app**, then select **Upload for me or my teams**.</span></span>

    ![Uma captura de tela do link carregar um aplicativo personalizado no Microsoft Teams](images/upload-custom-app.png)

1. <span data-ttu-id="47284-125">Navegue até o arquivo ZIP que você criou anteriormente e selecione **abrir**.</span><span class="sxs-lookup"><span data-stu-id="47284-125">Browse to the ZIP file you created previously and select **Open**.</span></span>

1. <span data-ttu-id="47284-126">Revise as informações do aplicativo e selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="47284-126">Review the application information and select **Add**.</span></span>

1. <span data-ttu-id="47284-127">O aplicativo é aberto no Teams e exibe um token de acesso.</span><span class="sxs-lookup"><span data-stu-id="47284-127">The application opens in Teams and displays an access token.</span></span>

<span data-ttu-id="47284-128">Se você copiar o token, poderá colá-lo no [JWT.ms](https://jwt.ms).</span><span class="sxs-lookup"><span data-stu-id="47284-128">If you copy the token, you can paste it into [jwt.ms](https://jwt.ms).</span></span> <span data-ttu-id="47284-129">Verifique se a audiência (a `aud` declaração) é a ID do aplicativo e o único escopo (a `scp` declaração) é o `access_as_user` escopo da API que você criou.</span><span class="sxs-lookup"><span data-stu-id="47284-129">Verify that the audience (the `aud` claim) is your application ID, and the only scope (the `scp` claim) is the `access_as_user` API scope you created.</span></span> <span data-ttu-id="47284-130">Isso significa que esse token não concede acesso direto ao Microsoft Graph!</span><span class="sxs-lookup"><span data-stu-id="47284-130">That means that this token does not grant direct access to Microsoft Graph!</span></span> <span data-ttu-id="47284-131">Em vez disso, a API Web que você irá implementar logo precisará trocar esse token usando o [fluxo em nome de](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) para obter um token que funcione com as chamadas do Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="47284-131">Instead, the Web API you will implement soon will need to exchange this token using the [on-behalf-of flow](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to get a token that will work with Microsoft Graph calls.</span></span>

## <a name="configure-authentication-in-the-aspnet-core-app"></a><span data-ttu-id="47284-132">Configurar a autenticação no aplicativo principal do ASP.NET</span><span class="sxs-lookup"><span data-stu-id="47284-132">Configure authentication in the ASP.NET Core app</span></span>

<span data-ttu-id="47284-133">Comece adicionando os serviços da plataforma de identidade da Microsoft ao aplicativo.</span><span class="sxs-lookup"><span data-stu-id="47284-133">Start by adding the Microsoft Identity platform services to the application.</span></span>

1. <span data-ttu-id="47284-134">Abra o arquivo **./Startup.cs** e adicione a instrução a seguir `using` à parte superior do arquivo.</span><span class="sxs-lookup"><span data-stu-id="47284-134">Open the **./Startup.cs** file and add the following `using` statement to the top of the file.</span></span>

    ```csharp
    using Microsoft.Identity.Web;
    ```

1. <span data-ttu-id="47284-135">Adicione a seguinte linha antes da `app.UseAuthorization();` linha na `Configure` função.</span><span class="sxs-lookup"><span data-stu-id="47284-135">Add the following line just before the `app.UseAuthorization();` line in the `Configure` function.</span></span>

    ```csharp
    app.UseAuthentication();
    ```

1. <span data-ttu-id="47284-136">Adicione a linha a seguir logo após a `endpoints.MapRazorPages();` linha na `Configure` função.</span><span class="sxs-lookup"><span data-stu-id="47284-136">Add the following line just after the `endpoints.MapRazorPages();` line in the `Configure` function.</span></span>

    ```csharp
    endpoints.MapControllers();
    ```

1. <span data-ttu-id="47284-137">Substitua a função `ConfigureServices` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="47284-137">Replace the existing `ConfigureServices` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="ConfigureServicesSnippet":::

    <span data-ttu-id="47284-138">Este código configura o aplicativo para permitir que as chamadas de APIs da Web sejam autenticadas com base no token de portador JWT no `Authorization` cabeçalho.</span><span class="sxs-lookup"><span data-stu-id="47284-138">This code configures the application to allow calls to Web APIs to be authenticated based on the JWT bearer token in the `Authorization` header.</span></span> <span data-ttu-id="47284-139">Ele também adiciona os serviços de aquisição de token que podem trocar o token por meio do fluxo em nome de.</span><span class="sxs-lookup"><span data-stu-id="47284-139">It also adds the token acquisition services that can exchange that token via the on-behalf-of flow.</span></span>

## <a name="create-the-web-api-controller"></a><span data-ttu-id="47284-140">Criar o controlador de API Web</span><span class="sxs-lookup"><span data-stu-id="47284-140">Create the Web API controller</span></span>

1. <span data-ttu-id="47284-141">Crie um novo diretório na raiz do projeto chamado **controladores**.</span><span class="sxs-lookup"><span data-stu-id="47284-141">Create a new directory in the root of the project named **Controllers**.</span></span>

1. <span data-ttu-id="47284-142">Crie um novo arquivo no diretório **./Controllers** chamado **CalendarController.cs** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="47284-142">Create a new file in the **./Controllers** directory named **CalendarController.cs** and add the following code.</span></span>

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Net;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Http;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using Microsoft.Identity.Web.Resource;
    using Microsoft.Graph;
    using TimeZoneConverter;

    namespace GraphTutorial.Controllers
    {
        [ApiController]
        [Route("[controller]")]
        [Authorize]
        public class CalendarController : ControllerBase
        {
            private static readonly string[] apiScopes = new[] { "access_as_user" };

            private readonly GraphServiceClient _graphClient;
            private readonly ITokenAcquisition _tokenAcquisition;
            private readonly ILogger<CalendarController> _logger;

            public CalendarController(ITokenAcquisition tokenAcquisition, GraphServiceClient graphClient, ILogger<CalendarController> logger)
            {
                _tokenAcquisition = tokenAcquisition;
                _graphClient = graphClient;
                _logger = logger;
            }

            [HttpGet]
            public async Task<string> Get()
            {
                // This verifies that the access_as_user scope is
                // present in the bearer token, throws if not
                HttpContext.VerifyUserHasAnyAcceptedScope(apiScopes);

                // To verify that the identity libraries have authenticated
                // based on the token, log the user's name
                _logger.LogInformation($"Authenticated user: {User.GetDisplayName()}");

                try
                {
                    // TEMPORARY
                    // Get a Graph token via OBO flow
                    var token = await _tokenAcquisition
                        .GetAccessTokenForUserAsync(new[]{
                            "User.Read",
                            "MailboxSettings.Read",
                            "Calendars.ReadWrite" });

                    // Log the token
                    _logger.LogInformation($"Access token for Graph: {token}");
                    return "{ \"status\": \"OK\" }";
                }
                catch (MicrosoftIdentityWebChallengeUserException ex)
                {
                    _logger.LogError(ex, "Consent required");
                    // This exception indicates consent is required.
                    // Return a 403 with "consent_required" in the body
                    // to signal to the tab it needs to prompt for consent
                    HttpContext.Response.ContentType = "text/plain";
                    HttpContext.Response.StatusCode = (int)HttpStatusCode.Forbidden;
                    await HttpContext.Response.WriteAsync("consent_required");
                    return null;
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Error occurred");
                    return null;
                }
            }
        }
    }
    ```

    <span data-ttu-id="47284-143">Isso implementa uma API Web ( `GET /calendar` ) que pode ser chamada na guia Teams. Por ora, ele simplesmente tenta trocar o token de portador por um token de gráfico.</span><span class="sxs-lookup"><span data-stu-id="47284-143">This implements a Web API (`GET /calendar`) that can be called from the Teams tab. For now it simply tries to exchange the bearer token for a Graph token.</span></span> <span data-ttu-id="47284-144">Na primeira vez que um usuário carrega a guia, ele falhará porque ainda não enviou permissão para permitir que o aplicativo acesse o Microsoft Graph em seu nome.</span><span class="sxs-lookup"><span data-stu-id="47284-144">The first time a user loads the tab, this will fail because they have not yet consented to allow the app access to Microsoft Graph on their behalf.</span></span>

1. <span data-ttu-id="47284-145">Abra **./pages/index.cshtml** e substitua a `successCallback` função com o seguinte.</span><span class="sxs-lookup"><span data-stu-id="47284-145">Open **./Pages/Index.cshtml** and replace the `successCallback` function with the following.</span></span>

    ```javascript
    successCallback: (token) => {
      // TEMPORARY: Call the Web API
      fetch('/calendar', {
        headers: {
          'Authorization': `Bearer ${token}`
        }
      }).then(response => {
        response.text()
          .then(body => {
            $('#tab-container').empty();
            $('<code/>', {
              text: body
            }).appendTo('#tab-container');
          });
      }).catch(error => {
        console.error(error);
        renderError(error);
      });
    }
    ```

    <span data-ttu-id="47284-146">Isso chamará a API Web e exibirá a resposta.</span><span class="sxs-lookup"><span data-stu-id="47284-146">This will call the Web API and display the response.</span></span>

1. <span data-ttu-id="47284-147">Salve suas alterações e reinicie o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="47284-147">Save your changes and restart the app.</span></span> <span data-ttu-id="47284-148">Atualize a guia no Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="47284-148">Refresh the tab in Microsoft Teams.</span></span> <span data-ttu-id="47284-149">A página deve ser exibida `consent_required` .</span><span class="sxs-lookup"><span data-stu-id="47284-149">The page should display `consent_required`.</span></span>

1. <span data-ttu-id="47284-150">Revise o log de saída na sua CLI.</span><span class="sxs-lookup"><span data-stu-id="47284-150">Review the log output in your CLI.</span></span> <span data-ttu-id="47284-151">Observe duas coisas.</span><span class="sxs-lookup"><span data-stu-id="47284-151">Notice two things.</span></span>

    - <span data-ttu-id="47284-152">Uma entrada como `Authenticated user: MeganB@contoso.com` .</span><span class="sxs-lookup"><span data-stu-id="47284-152">An entry like `Authenticated user: MeganB@contoso.com`.</span></span> <span data-ttu-id="47284-153">A API da Web autenticou o usuário com base no token enviado com a solicitação de API.</span><span class="sxs-lookup"><span data-stu-id="47284-153">The Web API has authenticated the user based on the token sent with the API request.</span></span>
    - <span data-ttu-id="47284-154">Uma entrada como `AADSTS65001: The user or administrator has not consented to use the application with ID...` .</span><span class="sxs-lookup"><span data-stu-id="47284-154">An entry like `AADSTS65001: The user or administrator has not consented to use the application with ID...`.</span></span> <span data-ttu-id="47284-155">Isso é esperado, pois o usuário ainda não foi solicitado a dar o consentimento para os escopos de permissão do Microsoft Graph solicitados.</span><span class="sxs-lookup"><span data-stu-id="47284-155">This is expected, since the user has not yet been prompted to consent for the requested Microsoft Graph permission scopes.</span></span>

## <a name="implement-consent-prompt"></a><span data-ttu-id="47284-156">Implementar prompt de consentimento</span><span class="sxs-lookup"><span data-stu-id="47284-156">Implement consent prompt</span></span>

<span data-ttu-id="47284-157">Como a API Web não pode avisar o usuário, a guia Teams precisará implementar um prompt.</span><span class="sxs-lookup"><span data-stu-id="47284-157">Because the Web API cannot prompt the user, the Teams tab will need to implement a prompt.</span></span> <span data-ttu-id="47284-158">Isso só precisará ser feito uma vez para cada usuário.</span><span class="sxs-lookup"><span data-stu-id="47284-158">This will only need to be done once for each user.</span></span> <span data-ttu-id="47284-159">Após um usuário, ele não precisará ser reconsente, a menos que ele explicitamente revogue o acesso ao seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="47284-159">Once a user consents, they do not need to reconsent unless they explicitly revoke access to your application.</span></span>

1. <span data-ttu-id="47284-160">Crie um novo arquivo no diretório **./Pages** chamado **Authenticate.cshtml.cs** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="47284-160">Create a new file in the **./Pages** directory named **Authenticate.cshtml.cs** and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Pages/Authenticate.cshtml.cs" id="AuthenticateModelSnippet":::

1. <span data-ttu-id="47284-161">Crie um novo arquivo no diretório **./Pages** chamado **Authenticate. cshtml** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="47284-161">Create a new file in the **./Pages** directory named **Authenticate.cshtml** and add the following code.</span></span>

    :::code language="razor" source="../demo/GraphTutorial/Pages/Authenticate.cshtml":::

1. <span data-ttu-id="47284-162">Crie um novo arquivo no diretório **./Pages** chamado **AuthComplete. cshtml** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="47284-162">Create a new file in the **./Pages** directory named **AuthComplete.cshtml** and add the following code.</span></span>

    :::code language="razor" source="../demo/GraphTutorial/Pages/AuthComplete.cshtml":::

1. <span data-ttu-id="47284-163">Abra **./pages/index.cshtml** e adicione as seguintes funções dentro da `<script>` marca.</span><span class="sxs-lookup"><span data-stu-id="47284-163">Open **./Pages/Index.cshtml** and add the following functions inside the `<script>` tag.</span></span>

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="LoadUserCalendarSnippet":::

1. <span data-ttu-id="47284-164">Adicione a seguinte função dentro da `<script>` marca para exibir um resultado bem-sucedido da API Web.</span><span class="sxs-lookup"><span data-stu-id="47284-164">Add the following function inside the `<script>` tag to display a successful result from the Web API.</span></span>

    ```javascript
    function renderCalendar(events) {
      $('#tab-container').empty();

      $('<pre/>').append($('<code/>', {
        text: JSON.stringify(events, null, 2),
        style: 'word-break: break-all;'
      })).appendTo('#tab-container');
    }
    ```

1. <span data-ttu-id="47284-165">Substitua o existente `successCallback` pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="47284-165">Replace the existing `successCallback` with the following code.</span></span>

    ```javascript
    successCallback: (token) => {
      loadUserCalendar(token, (events) => {
        renderCalendar(events);
      });
    }
    ```

1. <span data-ttu-id="47284-166">Salve suas alterações e reinicie o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="47284-166">Save your changes and restart the app.</span></span> <span data-ttu-id="47284-167">Atualize a guia no Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="47284-167">Refresh the tab in Microsoft Teams.</span></span> <span data-ttu-id="47284-168">A guia deve ser exibida `{ "status": "OK" }` .</span><span class="sxs-lookup"><span data-stu-id="47284-168">The tab should display `{ "status": "OK" }`.</span></span>

1. <span data-ttu-id="47284-169">Revise a saída do log.</span><span class="sxs-lookup"><span data-stu-id="47284-169">Review the log output.</span></span> <span data-ttu-id="47284-170">Você deve ver a `Access token for Graph` entrada.</span><span class="sxs-lookup"><span data-stu-id="47284-170">You should see the `Access token for Graph` entry.</span></span> <span data-ttu-id="47284-171">Se você analisar esse token, observará que ele contém os escopos do Microsoft Graph configurados em **appsettings.js**.</span><span class="sxs-lookup"><span data-stu-id="47284-171">If you parse that token, you'll notice that it contains the Microsoft Graph scopes configured in **appsettings.json**.</span></span>

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="47284-172">Armazenar e atualizar tokens</span><span class="sxs-lookup"><span data-stu-id="47284-172">Storing and refreshing tokens</span></span>

<span data-ttu-id="47284-173">Nesse ponto, seu aplicativo tem um token de acesso, que é enviado no `Authorization` cabeçalho das chamadas de API.</span><span class="sxs-lookup"><span data-stu-id="47284-173">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="47284-174">Este é o token que permite ao aplicativo acessar o Microsoft Graph em nome do usuário.</span><span class="sxs-lookup"><span data-stu-id="47284-174">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="47284-175">No entanto, esse token é de vida curta.</span><span class="sxs-lookup"><span data-stu-id="47284-175">However, this token is short-lived.</span></span> <span data-ttu-id="47284-176">O token expira uma hora após sua emissão.</span><span class="sxs-lookup"><span data-stu-id="47284-176">The token expires an hour after it is issued.</span></span> <span data-ttu-id="47284-177">É onde o token de atualização se torna útil.</span><span class="sxs-lookup"><span data-stu-id="47284-177">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="47284-178">O token de atualização permite que o aplicativo solicite um novo token de acesso sem exigir que o usuário entre novamente.</span><span class="sxs-lookup"><span data-stu-id="47284-178">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="47284-179">Como o aplicativo está usando a biblioteca Microsoft. Identity. Web, você não precisa implementar qualquer lógica de armazenamento ou de atualização.</span><span class="sxs-lookup"><span data-stu-id="47284-179">Because the app is using the Microsoft.Identity.Web library, you do not have to implement any token storage or refresh logic.</span></span>

<span data-ttu-id="47284-180">O aplicativo usa o cache de token na memória, o que é suficiente para aplicativos que não precisam persistir tokens quando o aplicativo é reiniciado.</span><span class="sxs-lookup"><span data-stu-id="47284-180">The app uses the in-memory token cache, which is sufficient for apps that do not need to persist tokens when the app restarts.</span></span> <span data-ttu-id="47284-181">Em vez disso, os aplicativos de produção podem usar as [Opções de cache distribuído](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) na biblioteca Microsoft. Identity. Web.</span><span class="sxs-lookup"><span data-stu-id="47284-181">Production apps may instead use the [distributed cache options](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in the Microsoft.Identity.Web library.</span></span>

<span data-ttu-id="47284-182">O `GetAccessTokenForUserAsync` método manipula a expiração do token e a atualização para você.</span><span class="sxs-lookup"><span data-stu-id="47284-182">The `GetAccessTokenForUserAsync` method handles token expiration and refresh for you.</span></span> <span data-ttu-id="47284-183">Primeiro ele verifica o token em cache e, se ele não tiver expirado, ele o retornará.</span><span class="sxs-lookup"><span data-stu-id="47284-183">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="47284-184">Se ele tiver expirado, ele usará o token de atualização em cache para obter um novo.</span><span class="sxs-lookup"><span data-stu-id="47284-184">If it is expired, it uses the cached refresh token to obtain a new one.</span></span>

<span data-ttu-id="47284-185">O **GraphServiceClient** que os controladores recebem por meio da injeção de dependência é pré-configurado com um provedor de autenticação que usa `GetAccessTokenForUserAsync` para você.</span><span class="sxs-lookup"><span data-stu-id="47284-185">The **GraphServiceClient** that controllers get via dependency injection is pre-configured with an authentication provider that uses `GetAccessTokenForUserAsync` for you.</span></span>
