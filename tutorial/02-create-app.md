<!-- markdownlint-disable MD002 MD041 -->

Os aplicativos de guia do Microsoft Teams têm várias opções para autenticar o usuário e chamar o Microsoft Graph. Neste exercício, você implementará uma guia que faz [logon único](/microsoftteams/platform/tabs/how-to/authentication/auth-aad-sso) para obter um token de autenticação no cliente e, em seguida, usa o [fluxo em nome de](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) no servidor para trocar esse token para obter acesso ao Microsoft Graph.

Para outras alternativas, confira o seguinte.

- [Crie uma guia do Microsoft Teams com o Microsoft Graph Toolkit](/graph/toolkit/get-started/build-a-microsoft-teams-tab). Este exemplo é completamente do lado do cliente e usa o Microsoft Graph Toolkit para lidar com a autenticação e fazendo chamadas para o Microsoft Graph.
- [Exemplo de autenticação do Microsoft Teams](https://github.com/OfficeDev/microsoft-teams-sample-auth-node). Este exemplo contém vários exemplos que abrangem diferentes cenários de autenticação.

## <a name="create-the-project"></a>Criar o projeto

Comece criando um aplicativo Web do ASP.NET Core.

1. Abra a interface de linha de comando (CLI) em um diretório onde você deseja criar o projeto. Execute o seguinte comando.

    ```Shell
    dotnet new webapp -o GraphTutorial
    ```

1. Depois que o projeto for criado, verifique se ele funciona alterando o diretório atual para o diretório **GraphTutorial** e executando o seguinte comando na sua CLI.

    ```Shell
    dotnet run
    ```

1. Abra o navegador e navegue até `https://localhost:5001` . Se tudo estiver funcionando, você deverá ver uma página padrão do ASP.NET Core.

> [!IMPORTANT]
> Se você receber um aviso de que o certificado para **localhost** é não confiável, você pode usar a CLI do .NET Core para instalar e confiar no certificado de desenvolvimento. Consulte [impor HTTPS no ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-3.1) para obter instruções para sistemas operacionais específicos.

## <a name="add-nuget-packages"></a>Adicionar pacotes NuGet

Antes de prosseguir, instale alguns pacotes NuGet adicionais que serão usados posteriormente.

- [Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) para autenticação e solicitação de tokens de acesso.
- [Microsoft. Identity. Web. MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) para adicionar o suporte do Microsoft Graph configurado com Microsoft. Identity. Web.
- [Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) para atualizar a versão deste pacote instalado pelo Microsoft. Identity. Web. MicrosoftGraph.
- [Microsoft. AspNetCore. Mvc. NewtonsoftJson](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) para habilitar conversores JSON do Newtonsoft para compatibilidade com o SDK do Microsoft Graph.
- [Timezoneconverter](https://github.com/mj1856/TimeZoneConverter) para conversão de identificadores de fuso horário do Windows em identificadores da IANA.

1. Execute os seguintes comandos em sua CLI para instalar as dependências.

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.1.0
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.1.0
    dotnet add package Microsoft.Graph --version 3.16.0
    dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a>Projetar o aplicativo

Nesta seção, você criará a estrutura básica de interface do usuário do aplicativo.

> [!TIP]
> Você pode usar qualquer editor de texto para editar os arquivos de origem deste tutorial. No entanto, o [código do Visual Studio](https://code.visualstudio.com/) fornece recursos adicionais, como depuração e IntelliSense.

1. Abra **./Pages/Shared/_Layout. cshtml** e substitua todo o conteúdo pelo código a seguir para atualizar o layout global do aplicativo.

    :::code language="cshtml" source="../demo/GraphTutorial/Pages/Shared/_Layout.cshtml" id="LayoutSnippet":::

    Isso substitui a inicialização pela [interface do usuário fluente](https://developer.microsoft.com/fluentui), adiciona o [SDK do Microsoft Teams](/javascript/api/overview/msteams-client)e simplifica o layout.

1. Abra **./wwwroot/js/site.js** e adicione o código a seguir.

    :::code language="javascript" source="../demo/GraphTutorial/wwwroot/js/site.js" id="ThemeSupportSnippet":::

    Isso adiciona um manipulador de alteração de tema simples para alterar a cor de texto padrão para temas escuros e de alto contraste.

1. Abra **./wwwroot/CSS/site.css** e substitua seu conteúdo pelo seguinte.

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. Abra **./pages/index.cshtml** e substitua seu conteúdo pelo código a seguir.

    ```cshtml
    @page
    @model IndexModel
    @{
      ViewData["Title"] = "Home page";
    }

    <div id="tab-container">
      <h1 class="ms-fontSize-24 ms-fontWeight-semibold">Loading...</h1>
    </div>

    @section Scripts
    {
      <script>
      </script>
    }
    ```

1. Abra **./Startup.cs** e remova a `app.UseHttpsRedirection();` linha no `Configure` método. Isso é necessário para que o tunelamento do ngrok funcione.

## <a name="run-ngrok"></a>Executar ngrok

O Microsoft Teams não oferece suporte a hospedagem local para aplicativos. O servidor que hospeda o aplicativo deve estar disponível na nuvem usando pontos de extremidade HTTPS. Para depuração local, você pode usar o ngrok para criar uma URL pública para seu projeto hospedado localmente.

1. Abra sua CLI e execute o seguinte comando para iniciar o ngrok.

    ```Shell
    ngrok http 5000
    ```

1. Após a inicialização do ngrok, copie a URL de encaminhamento HTTPS. Deve parecer `https://50153897dd4d.ngrok.io` . Você precisará desse valor nas etapas posteriores.

> [!IMPORTANT]
> Se você estiver usando a versão gratuita do ngrok, a URL de encaminhamento será alterada toda vez que você reiniciar o ngrok. É recomendável deixar o ngrok em execução até que este tutorial seja concluído para manter a mesma URL. Se for necessário reiniciar o ngrok, você precisará atualizar sua URL em todos os lugares em que ela é usada e reinstalar o aplicativo no Microsoft Teams.
