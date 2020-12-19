<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1b3a6-101">Os aplicativos de guia do Microsoft Teams têm várias opções para autenticar o usuário e chamar o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-101">Microsoft Teams tab applications have multiple options to authenticate the user and call Microsoft Graph.</span></span> <span data-ttu-id="1b3a6-102">Neste exercício, você implementará uma guia que faz [logon único](/microsoftteams/platform/tabs/how-to/authentication/auth-aad-sso) para obter um token de autenticação no cliente e, em seguida, usa o [fluxo em nome de](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) no servidor para trocar esse token para obter acesso ao Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-102">In this exercise, you'll implement a tab that does [single sign-on](/microsoftteams/platform/tabs/how-to/authentication/auth-aad-sso) to get an auth token on the client, then uses the [on-behalf-of flow](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) on the server to exchange that token to get access to Microsoft Graph.</span></span>

<span data-ttu-id="1b3a6-103">Para outras alternativas, confira o seguinte.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-103">For other alternatives, see the following.</span></span>

- <span data-ttu-id="1b3a6-104">[Crie uma guia do Microsoft Teams com o Microsoft Graph Toolkit](/graph/toolkit/get-started/build-a-microsoft-teams-tab).</span><span class="sxs-lookup"><span data-stu-id="1b3a6-104">[Build a Microsoft Teams tab with the Microsoft Graph Toolkit](/graph/toolkit/get-started/build-a-microsoft-teams-tab).</span></span> <span data-ttu-id="1b3a6-105">Este exemplo é completamente do lado do cliente e usa o Microsoft Graph Toolkit para lidar com a autenticação e fazendo chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-105">This sample is completely client-side, and uses the Microsoft Graph Toolkit to handle authentication and making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="1b3a6-106">[Exemplo de autenticação do Microsoft Teams](https://github.com/OfficeDev/microsoft-teams-sample-auth-node).</span><span class="sxs-lookup"><span data-stu-id="1b3a6-106">[Microsoft Teams Authentication Sample](https://github.com/OfficeDev/microsoft-teams-sample-auth-node).</span></span> <span data-ttu-id="1b3a6-107">Este exemplo contém vários exemplos que abrangem diferentes cenários de autenticação.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-107">This sample contains multiple examples covering different authentication scenarios.</span></span>

## <a name="create-the-project"></a><span data-ttu-id="1b3a6-108">Criar o projeto</span><span class="sxs-lookup"><span data-stu-id="1b3a6-108">Create the project</span></span>

<span data-ttu-id="1b3a6-109">Comece criando um aplicativo Web do ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-109">Start by creating an ASP.NET Core web app.</span></span>

1. <span data-ttu-id="1b3a6-110">Abra a interface de linha de comando (CLI) em um diretório onde você deseja criar o projeto.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-110">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="1b3a6-111">Execute o seguinte comando.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-111">Run the following command.</span></span>

    ```Shell
    dotnet new webapp -o GraphTutorial
    ```

1. <span data-ttu-id="1b3a6-112">Depois que o projeto for criado, verifique se ele funciona alterando o diretório atual para o diretório **GraphTutorial** e executando o seguinte comando na sua CLI.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-112">Once the project is created, verify that it works by changing the current directory to the **GraphTutorial** directory and running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

1. <span data-ttu-id="1b3a6-113">Abra o navegador e navegue até `https://localhost:5001` .</span><span class="sxs-lookup"><span data-stu-id="1b3a6-113">Open your browser and browse to `https://localhost:5001`.</span></span> <span data-ttu-id="1b3a6-114">Se tudo estiver funcionando, você deverá ver uma página padrão do ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-114">If everything is working, you should see a default ASP.NET Core page.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="1b3a6-115">Se você receber um aviso de que o certificado para **localhost** é não confiável, você pode usar a CLI do .NET Core para instalar e confiar no certificado de desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-115">If you receive a warning that the certificate for **localhost** is un-trusted you can use the .NET Core CLI to install and trust the development certificate.</span></span> <span data-ttu-id="1b3a6-116">Consulte [impor HTTPS no ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-3.1) para obter instruções para sistemas operacionais específicos.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-116">See [Enforce HTTPS in ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-3.1) for instructions for specific operating systems.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="1b3a6-117">Adicionar pacotes NuGet</span><span class="sxs-lookup"><span data-stu-id="1b3a6-117">Add NuGet packages</span></span>

<span data-ttu-id="1b3a6-118">Antes de prosseguir, instale alguns pacotes NuGet adicionais que serão usados posteriormente.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-118">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="1b3a6-119">[Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) para autenticação e solicitação de tokens de acesso.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-119">[Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) for authenticating and requesting access tokens.</span></span>
- <span data-ttu-id="1b3a6-120">[Microsoft. Identity. Web. MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) para adicionar o suporte do Microsoft Graph configurado com Microsoft. Identity. Web.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-120">[Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) for adding Microsoft Graph support configured with Microsoft.Identity.Web.</span></span>
- <span data-ttu-id="1b3a6-121">[Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) para atualizar a versão deste pacote instalado pelo Microsoft. Identity. Web. MicrosoftGraph.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-121">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) to update the version of this package installed by Microsoft.Identity.Web.MicrosoftGraph.</span></span>
- <span data-ttu-id="1b3a6-122">[Microsoft. AspNetCore. Mvc. NewtonsoftJson](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) para habilitar conversores JSON do Newtonsoft para compatibilidade com o SDK do Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-122">[Microsoft.AspNetCore.Mvc.NewtonsoftJson](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) to enable Newtonsoft JSON converters for compatibility with the Microsoft Graph SDK.</span></span>
- <span data-ttu-id="1b3a6-123">[Timezoneconverter](https://github.com/mj1856/TimeZoneConverter) para conversão de identificadores de fuso horário do Windows em identificadores da IANA.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-123">[TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) for translating Windows time zone identifiers to IANA identifiers.</span></span>

1. <span data-ttu-id="1b3a6-124">Execute os seguintes comandos em sua CLI para instalar as dependências.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-124">Run the following commands in your CLI to install the dependencies.</span></span>

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.1.0
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.1.0
    dotnet add package Microsoft.Graph --version 3.16.0
    dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a><span data-ttu-id="1b3a6-125">Projetar o aplicativo</span><span class="sxs-lookup"><span data-stu-id="1b3a6-125">Design the app</span></span>

<span data-ttu-id="1b3a6-126">Nesta seção, você criará a estrutura básica de interface do usuário do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-126">In this section you will create the basic UI structure of the application.</span></span>

> [!TIP]
> <span data-ttu-id="1b3a6-127">Você pode usar qualquer editor de texto para editar os arquivos de origem deste tutorial.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-127">You can use any text editor to edit the source files for this tutorial.</span></span> <span data-ttu-id="1b3a6-128">No entanto, o [código do Visual Studio](https://code.visualstudio.com/) fornece recursos adicionais, como depuração e IntelliSense.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-128">However, [Visual Studio Code](https://code.visualstudio.com/) provides additional features, such as debugging and Intellisense.</span></span>

1. <span data-ttu-id="1b3a6-129">Abra **./Pages/Shared/_Layout. cshtml** e substitua todo o conteúdo pelo código a seguir para atualizar o layout global do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-129">Open **./Pages/Shared/_Layout.cshtml** and replace its entire contents with the following code to update the global layout of the app.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Pages/Shared/_Layout.cshtml" id="LayoutSnippet":::

    <span data-ttu-id="1b3a6-130">Isso substitui a inicialização pela [interface do usuário fluente](https://developer.microsoft.com/fluentui), adiciona o [SDK do Microsoft Teams](/javascript/api/overview/msteams-client)e simplifica o layout.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-130">This replaces Bootstrap with [Fluent UI](https://developer.microsoft.com/fluentui), adds the [Microsoft Teams SDK](/javascript/api/overview/msteams-client), and simplifies the layout.</span></span>

1. <span data-ttu-id="1b3a6-131">Abra **./wwwroot/js/site.js** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-131">Open **./wwwroot/js/site.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/GraphTutorial/wwwroot/js/site.js" id="ThemeSupportSnippet":::

    <span data-ttu-id="1b3a6-132">Isso adiciona um manipulador de alteração de tema simples para alterar a cor de texto padrão para temas escuros e de alto contraste.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-132">This adds a simple theme change handler to change the default text color for dark and high contrast themes.</span></span>

1. <span data-ttu-id="1b3a6-133">Abra **./wwwroot/CSS/site.css** e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-133">Open **./wwwroot/css/site.css** and replace its contents with the following.</span></span>

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. <span data-ttu-id="1b3a6-134">Abra **./pages/index.cshtml** e substitua seu conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-134">Open **./Pages/Index.cshtml** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="1b3a6-135">Abra **./Startup.cs** e remova a `app.UseHttpsRedirection();` linha no `Configure` método.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-135">Open **./Startup.cs** and remove the `app.UseHttpsRedirection();` line in the `Configure` method.</span></span> <span data-ttu-id="1b3a6-136">Isso é necessário para que o tunelamento do ngrok funcione.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-136">This is necessary for ngrok tunneling to work.</span></span>

## <a name="run-ngrok"></a><span data-ttu-id="1b3a6-137">Executar ngrok</span><span class="sxs-lookup"><span data-stu-id="1b3a6-137">Run ngrok</span></span>

<span data-ttu-id="1b3a6-138">O Microsoft Teams não oferece suporte a hospedagem local para aplicativos.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-138">Microsoft Teams does not support local hosting for apps.</span></span> <span data-ttu-id="1b3a6-139">O servidor que hospeda o aplicativo deve estar disponível na nuvem usando pontos de extremidade HTTPS.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-139">The server hosting your app must be available from the cloud using HTTPS endpoints.</span></span> <span data-ttu-id="1b3a6-140">Para depuração local, você pode usar o ngrok para criar uma URL pública para seu projeto hospedado localmente.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-140">For debugging locally, you can use ngrok to create a public URL for your locally-hosted project.</span></span>

1. <span data-ttu-id="1b3a6-141">Abra sua CLI e execute o seguinte comando para iniciar o ngrok.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-141">Open your CLI and run the following command to start ngrok.</span></span>

    ```Shell
    ngrok http 5000
    ```

1. <span data-ttu-id="1b3a6-142">Após a inicialização do ngrok, copie a URL de encaminhamento HTTPS.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-142">Once ngrok starts, copy the HTTPS Forwarding URL.</span></span> <span data-ttu-id="1b3a6-143">Deve parecer `https://50153897dd4d.ngrok.io` .</span><span class="sxs-lookup"><span data-stu-id="1b3a6-143">It should look like `https://50153897dd4d.ngrok.io`.</span></span> <span data-ttu-id="1b3a6-144">Você precisará desse valor nas etapas posteriores.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-144">You'll need this value in later steps.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="1b3a6-145">Se você estiver usando a versão gratuita do ngrok, a URL de encaminhamento será alterada toda vez que você reiniciar o ngrok.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-145">If you are using the free version of ngrok, the forwarding URL changes every time you restart ngrok.</span></span> <span data-ttu-id="1b3a6-146">É recomendável deixar o ngrok em execução até que este tutorial seja concluído para manter a mesma URL.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-146">It's recommended that you leave ngrok running until you complete this tutorial to keep the same URL.</span></span> <span data-ttu-id="1b3a6-147">Se for necessário reiniciar o ngrok, você precisará atualizar sua URL em todos os lugares em que ela é usada e reinstalar o aplicativo no Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="1b3a6-147">If you have to restart ngrok, you'll need to update your URL everywhere that it is used and reinstall the app in Microsoft Teams.</span></span>
