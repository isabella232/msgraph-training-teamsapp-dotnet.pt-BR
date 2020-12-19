<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="cba64-101">O manifesto do aplicativo descreve como o aplicativo se integra ao Microsoft Teams e é necessário para instalar aplicativos.</span><span class="sxs-lookup"><span data-stu-id="cba64-101">The app manifest describes how the app integrates with Microsoft Teams and is required to install apps.</span></span> <span data-ttu-id="cba64-102">Nesta seção, você usará o app Studio no cliente Microsoft Teams para gerar um manifesto.</span><span class="sxs-lookup"><span data-stu-id="cba64-102">In this section you'll use App Studio in the Microsoft Teams client to generate a manifest.</span></span>

1. <span data-ttu-id="cba64-103">Se você ainda não tiver o app Studio instalado no Teams, [Instale-o agora](/microsoftteams/platform/concepts/build-and-test/app-studio-overview).</span><span class="sxs-lookup"><span data-stu-id="cba64-103">If you do not already have App Studio installed in Teams, [install it now](/microsoftteams/platform/concepts/build-and-test/app-studio-overview).</span></span>

1. <span data-ttu-id="cba64-104">Inicie o app Studio no Microsoft Teams e selecione o **Editor de manifesto**.</span><span class="sxs-lookup"><span data-stu-id="cba64-104">Launch App Studio in Microsoft Teams and select the **Manifest editor**.</span></span>

1. <span data-ttu-id="cba64-105">Selecione **criar um novo aplicativo**.</span><span class="sxs-lookup"><span data-stu-id="cba64-105">Select **Create a new app**.</span></span>

    ![Uma captura de tela do editor de manifesto no app Studio no Microsoft Teams](images/app-studio-01.png)

1. <span data-ttu-id="cba64-107">Na página **detalhes do aplicativo** , preencha os campos obrigatórios.</span><span class="sxs-lookup"><span data-stu-id="cba64-107">On the **App details** page, fill in the required fields.</span></span>

    > [!NOTE]
    > <span data-ttu-id="cba64-108">Você pode usar os ícones padrão na seção **branding** ou carregar seus próprios.</span><span class="sxs-lookup"><span data-stu-id="cba64-108">You can use the default icons in the **Branding** section or upload your own.</span></span>

1. <span data-ttu-id="cba64-109">No menu à esquerda, selecione **guias** em **recursos**.</span><span class="sxs-lookup"><span data-stu-id="cba64-109">On the left-hand menu, select **Tabs** under **Capabilities**.</span></span>

1. <span data-ttu-id="cba64-110">Selecione **Adicionar** em **Adicionar uma guia pessoal**.</span><span class="sxs-lookup"><span data-stu-id="cba64-110">Select **Add** under **Add a personal tab**.</span></span>

    ![Uma captura de tela da página guias no app Studio](images/app-studio-02.png)

1. <span data-ttu-id="cba64-112">Preencha os campos da seguinte maneira, onde `YOUR_NGROK_URL` é a URL de encaminhamento que você copiou na seção anterior.</span><span class="sxs-lookup"><span data-stu-id="cba64-112">Fill in the fields as follows, where `YOUR_NGROK_URL` is the forwarding URL you copied in the previous section.</span></span> <span data-ttu-id="cba64-113">Selecione **salvar** quando terminar.</span><span class="sxs-lookup"><span data-stu-id="cba64-113">Select **Save** when done.</span></span>

    - <span data-ttu-id="cba64-114">**Nome:**`Create event`</span><span class="sxs-lookup"><span data-stu-id="cba64-114">**Name:** `Create event`</span></span>
    - <span data-ttu-id="cba64-115">**ID da entidade:**`createEventTab`</span><span class="sxs-lookup"><span data-stu-id="cba64-115">**Entity ID:** `createEventTab`</span></span>
    - <span data-ttu-id="cba64-116">**URL do conteúdo:**`YOUR_NGROK_URL/newevent`</span><span class="sxs-lookup"><span data-stu-id="cba64-116">**Content URL:** `YOUR_NGROK_URL/newevent`</span></span>

1. <span data-ttu-id="cba64-117">Selecione **Adicionar** em **Adicionar uma guia pessoal**.</span><span class="sxs-lookup"><span data-stu-id="cba64-117">Select **Add** under **Add a personal tab**.</span></span>

1. <span data-ttu-id="cba64-118">Preencha os campos da seguinte maneira, onde `YOUR_NGROK_URL` é a URL de encaminhamento que você copiou na seção anterior.</span><span class="sxs-lookup"><span data-stu-id="cba64-118">Fill in the fields as follows, where `YOUR_NGROK_URL` is the forwarding URL you copied in the previous section.</span></span> <span data-ttu-id="cba64-119">Selecione **salvar** quando terminar.</span><span class="sxs-lookup"><span data-stu-id="cba64-119">Select **Save** when done.</span></span>

    - <span data-ttu-id="cba64-120">**Nome:**`Graph calendar`</span><span class="sxs-lookup"><span data-stu-id="cba64-120">**Name:** `Graph calendar`</span></span>
    - <span data-ttu-id="cba64-121">**ID da entidade:**`calendarTab`</span><span class="sxs-lookup"><span data-stu-id="cba64-121">**Entity ID:** `calendarTab`</span></span>
    - <span data-ttu-id="cba64-122">**URL do conteúdo:**`YOUR_NGROK_URL`</span><span class="sxs-lookup"><span data-stu-id="cba64-122">**Content URL:** `YOUR_NGROK_URL`</span></span>

1. <span data-ttu-id="cba64-123">No menu à esquerda, selecione **domínios e permissões** em **concluir**.</span><span class="sxs-lookup"><span data-stu-id="cba64-123">On the left-hand menu, select **Domains and permissions** under **Finish**.</span></span>

1. <span data-ttu-id="cba64-124">Defina a **ID do aplicativo AAD** para a ID do aplicativo do registro do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="cba64-124">Set the **AAD App ID** to the application ID from your app registration.</span></span>

1. <span data-ttu-id="cba64-125">Defina o campo **logon único** como o URI da ID do aplicativo do registro do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="cba64-125">Set the **Single-Sign-On** field to the application ID URI from your app registration.</span></span>

1. <span data-ttu-id="cba64-126">No menu à esquerda, selecione **testar e distribuir** em **concluir**.</span><span class="sxs-lookup"><span data-stu-id="cba64-126">On the left-hand menu, select **Test and distribute** under **Finish**.</span></span> <span data-ttu-id="cba64-127">Selecione **baixar**.</span><span class="sxs-lookup"><span data-stu-id="cba64-127">Select **Download**.</span></span>

1. <span data-ttu-id="cba64-128">Crie um novo diretório na raiz do projeto chamado **manifest**.</span><span class="sxs-lookup"><span data-stu-id="cba64-128">Create a new directory in the root of the project named **Manifest**.</span></span> <span data-ttu-id="cba64-129">Extraia o conteúdo do arquivo ZIP baixado para esse diretório.</span><span class="sxs-lookup"><span data-stu-id="cba64-129">Extract the contents of the downloaded ZIP file to this directory.</span></span>
