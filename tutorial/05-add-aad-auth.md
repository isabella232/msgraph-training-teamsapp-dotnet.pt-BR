<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você estenderá o aplicativo do exercício anterior para suportar a autenticação de logon único com o Azure AD. Isso é necessário para obter o token de acesso OAuth necessário para chamar a API do Microsoft Graph. Nesta etapa, você irá configurar a biblioteca [Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) .

> [!IMPORTANT]
> Para evitar o armazenamento da ID do aplicativo e o segredo na fonte, você usará o [Gerenciador de segredo do .net](/aspnet/core/security/app-secrets) para armazenar esses valores. O gerente secreto é apenas para fins de desenvolvimento, os aplicativos de produção devem usar um Gerenciador de segredo confiável para armazenar segredos.

1. Abra **./appsettings.jsem** e substitua seu conteúdo pelo seguinte.

    :::code language="json" source="../demo/GraphTutorial/appsettings.example.json" highlight="2-8":::

1. Abra a CLI no diretório onde o **GraphTutorial. csproj** está localizado e execute os seguintes comandos, substituindo `YOUR_APP_ID` pela ID do aplicativo do portal do Azure e `YOUR_APP_SECRET` com o segredo do aplicativo.

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>Implementar logon

Primeiro, implemente o logon único no código JavaScript do aplicativo. Você usará o [SDK do JavaScript do Microsoft Teams](/javascript/api/overview/msteams-client) para obter um token de acesso que permite que o código JavaScript em execução no cliente do teams faça chamadas AJAX para a API Web que você implementará mais tarde.

1. Abra **./pages/index.cshtml** e adicione o código a seguir dentro da `<script>` marca.

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

    Isso chama o `microsoftTeams.authentication.getAuthToken` para autenticar silenciosamente como o usuário conectado ao Teams. Normalmente, não há nenhum prompt de interface do usuário envolvido, a menos que o usuário tenha que dar o consentimento. O código, em seguida, exibe o token na guia.

1. Salve suas alterações e inicie o aplicativo executando o seguinte comando em sua CLI.

    ```Shell
    dotnet run
    ```

    > [!IMPORTANT]
    > Se você tiver reiniciado o ngrok e sua URL do ngrok tiver mudado, certifique-se de atualizar o valor de ngrok no seguinte local **antes** de testar.
    >
    > - O URI de redirecionamento em seu registro de aplicativo
    > - O URI da ID do aplicativo em seu registro de aplicativo
    > - `contentUrl` em manifest.jsem
    > - `validDomains` em manifest.jsem
    > - `resource` em manifest.jsem

1. Crie um arquivo ZIP com **manifest.jsem**, **color.png** e **outline.png**.

1. No Microsoft Teams, selecione **aplicativos** na barra esquerda, selecione **carregar um aplicativo personalizado** e, em seguida, selecione **carregar para mim ou minhas equipes**.

    ![Uma captura de tela do link carregar um aplicativo personalizado no Microsoft Teams](images/upload-custom-app.png)

1. Navegue até o arquivo ZIP que você criou anteriormente e selecione **abrir**.

1. Revise as informações do aplicativo e selecione **Adicionar**.

1. O aplicativo é aberto no Teams e exibe um token de acesso.

Se você copiar o token, poderá colá-lo no [JWT.ms](https://jwt.ms). Verifique se a audiência (a `aud` declaração) é a ID do aplicativo e o único escopo (a `scp` declaração) é o `access_as_user` escopo da API que você criou. Isso significa que esse token não concede acesso direto ao Microsoft Graph! Em vez disso, a API Web que você irá implementar logo precisará trocar esse token usando o [fluxo em nome de](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) para obter um token que funcione com as chamadas do Microsoft Graph.

## <a name="configure-authentication-in-the-aspnet-core-app"></a>Configurar a autenticação no aplicativo principal do ASP.NET

Comece adicionando os serviços da plataforma de identidade da Microsoft ao aplicativo.

1. Abra o arquivo **./Startup.cs** e adicione a instrução a seguir `using` à parte superior do arquivo.

    ```csharp
    using Microsoft.Identity.Web;
    ```

1. Adicione a seguinte linha antes da `app.UseAuthorization();` linha na `Configure` função.

    ```csharp
    app.UseAuthentication();
    ```

1. Adicione a linha a seguir logo após a `endpoints.MapRazorPages();` linha na `Configure` função.

    ```csharp
    endpoints.MapControllers();
    ```

1. Substitua a função `ConfigureServices` existente pelo seguinte.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="ConfigureServicesSnippet":::

    Este código configura o aplicativo para permitir que as chamadas de APIs da Web sejam autenticadas com base no token de portador JWT no `Authorization` cabeçalho. Ele também adiciona os serviços de aquisição de token que podem trocar o token por meio do fluxo em nome de.

## <a name="create-the-web-api-controller"></a>Criar o controlador de API Web

1. Crie um novo diretório na raiz do projeto chamado **controladores**.

1. Crie um novo arquivo no diretório **./Controllers** chamado **CalendarController.cs** e adicione o código a seguir.

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

    Isso implementa uma API Web ( `GET /calendar` ) que pode ser chamada na guia Teams. Por ora, ele simplesmente tenta trocar o token de portador por um token de gráfico. Na primeira vez que um usuário carrega a guia, ele falhará porque ainda não enviou permissão para permitir que o aplicativo acesse o Microsoft Graph em seu nome.

1. Abra **./pages/index.cshtml** e substitua a `successCallback` função com o seguinte.

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

    Isso chamará a API Web e exibirá a resposta.

1. Salve suas alterações e reinicie o aplicativo. Atualize a guia no Microsoft Teams. A página deve ser exibida `consent_required` .

1. Revise o log de saída na sua CLI. Observe duas coisas.

    - Uma entrada como `Authenticated user: MeganB@contoso.com` . A API da Web autenticou o usuário com base no token enviado com a solicitação de API.
    - Uma entrada como `AADSTS65001: The user or administrator has not consented to use the application with ID...` . Isso é esperado, pois o usuário ainda não foi solicitado a dar o consentimento para os escopos de permissão do Microsoft Graph solicitados.

## <a name="implement-consent-prompt"></a>Implementar prompt de consentimento

Como a API Web não pode avisar o usuário, a guia Teams precisará implementar um prompt. Isso só precisará ser feito uma vez para cada usuário. Após um usuário, ele não precisará ser reconsente, a menos que ele explicitamente revogue o acesso ao seu aplicativo.

1. Crie um novo arquivo no diretório **./Pages** chamado **Authenticate.cshtml.cs** e adicione o código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Pages/Authenticate.cshtml.cs" id="AuthenticateModelSnippet":::

1. Crie um novo arquivo no diretório **./Pages** chamado **Authenticate. cshtml** e adicione o código a seguir.

    :::code language="razor" source="../demo/GraphTutorial/Pages/Authenticate.cshtml":::

1. Crie um novo arquivo no diretório **./Pages** chamado **AuthComplete. cshtml** e adicione o código a seguir.

    :::code language="razor" source="../demo/GraphTutorial/Pages/AuthComplete.cshtml":::

1. Abra **./pages/index.cshtml** e adicione as seguintes funções dentro da `<script>` marca.

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="LoadUserCalendarSnippet":::

1. Adicione a seguinte função dentro da `<script>` marca para exibir um resultado bem-sucedido da API Web.

    ```javascript
    function renderCalendar(events) {
      $('#tab-container').empty();

      $('<pre/>').append($('<code/>', {
        text: JSON.stringify(events, null, 2),
        style: 'word-break: break-all;'
      })).appendTo('#tab-container');
    }
    ```

1. Substitua o existente `successCallback` pelo código a seguir.

    ```javascript
    successCallback: (token) => {
      loadUserCalendar(token, (events) => {
        renderCalendar(events);
      });
    }
    ```

1. Salve suas alterações e reinicie o aplicativo. Atualize a guia no Microsoft Teams. A guia deve ser exibida `{ "status": "OK" }` .

1. Revise a saída do log. Você deve ver a `Access token for Graph` entrada. Se você analisar esse token, observará que ele contém os escopos do Microsoft Graph configurados em **appsettings.js**.

## <a name="storing-and-refreshing-tokens"></a>Armazenar e atualizar tokens

Nesse ponto, seu aplicativo tem um token de acesso, que é enviado no `Authorization` cabeçalho das chamadas de API. Este é o token que permite ao aplicativo acessar o Microsoft Graph em nome do usuário.

No entanto, esse token é de vida curta. O token expira uma hora após sua emissão. É onde o token de atualização se torna útil. O token de atualização permite que o aplicativo solicite um novo token de acesso sem exigir que o usuário entre novamente.

Como o aplicativo está usando a biblioteca Microsoft. Identity. Web, você não precisa implementar qualquer lógica de armazenamento ou de atualização.

O aplicativo usa o cache de token na memória, o que é suficiente para aplicativos que não precisam persistir tokens quando o aplicativo é reiniciado. Em vez disso, os aplicativos de produção podem usar as [Opções de cache distribuído](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) na biblioteca Microsoft. Identity. Web.

O `GetAccessTokenForUserAsync` método manipula a expiração do token e a atualização para você. Primeiro ele verifica o token em cache e, se ele não tiver expirado, ele o retornará. Se ele tiver expirado, ele usará o token de atualização em cache para obter um novo.

O **GraphServiceClient** que os controladores recebem por meio da injeção de dependência é pré-configurado com um provedor de autenticação que usa `GetAccessTokenForUserAsync` para você.
