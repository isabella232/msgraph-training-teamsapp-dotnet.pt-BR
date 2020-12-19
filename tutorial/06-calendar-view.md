<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="eb7c8-101">Nesta seção, você incorporará o Microsoft Graph no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-101">In this section you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="eb7c8-102">Para este aplicativo, você usará a [biblioteca de cliente do Microsoft Graph para .net](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

# <a name="get-a-calendar-view"></a><span data-ttu-id="eb7c8-103">Obter um modo de exibição de calendário</span><span class="sxs-lookup"><span data-stu-id="eb7c8-103">Get a calendar view</span></span>

<span data-ttu-id="eb7c8-104">Um modo de exibição de calendário é um conjunto de eventos do calendário do usuário que ocorrem entre dois pontos de tempo.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-104">A calendar view is a set of events from the user's calendar that occur between two points of time.</span></span> <span data-ttu-id="eb7c8-105">Você o usará para obter os eventos do usuário para a semana atual.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-105">You'll use this to get the user's events for the current week.</span></span>

1. <span data-ttu-id="eb7c8-106">Abra **./Controllers/CalendarController.cs** e adicione a função a seguir à classe **CalendarController** .</span><span class="sxs-lookup"><span data-stu-id="eb7c8-106">Open **./Controllers/CalendarController.cs** and add the following function to the **CalendarController** class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetStartOfWeekSnippet":::

1. <span data-ttu-id="eb7c8-107">Adicione a seguinte função para lidar com exceções retornadas pelas chamadas do Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-107">Add the following function to handle exceptions returned from Microsoft Graph calls.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="HandleGraphExceptionSnippet":::

1. <span data-ttu-id="eb7c8-108">Substitua a função `Get` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-108">Replace the existing `Get` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetSnippet" highlight="2,14-57":::

    <span data-ttu-id="eb7c8-109">Revise as alterações.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-109">Review the changes.</span></span> <span data-ttu-id="eb7c8-110">Esta nova versão da função:</span><span class="sxs-lookup"><span data-stu-id="eb7c8-110">This new version of the function:</span></span>

    - <span data-ttu-id="eb7c8-111">Retorna `IEnumerable<Event>` em vez de `string` .</span><span class="sxs-lookup"><span data-stu-id="eb7c8-111">Returns `IEnumerable<Event>` instead of `string`.</span></span>
    - <span data-ttu-id="eb7c8-112">Obtém as configurações de caixa de correio do usuário usando o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-112">Gets the user's mailbox settings using Microsoft Graph.</span></span>
    - <span data-ttu-id="eb7c8-113">Usa o fuso horário do usuário para calcular o início e o fim da semana atual.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-113">Uses the user's time zone to calculate the start and end of the current week.</span></span>
    - <span data-ttu-id="eb7c8-114">Obtém uma visão de calendário</span><span class="sxs-lookup"><span data-stu-id="eb7c8-114">Gets a calendar view</span></span>
        - <span data-ttu-id="eb7c8-115">Usa a `.Header()` função para incluir um `Prefer: outlook.timezone` cabeçalho, o que faz com que os eventos retornados tenham seus horários de início e término convertidos no fuso horário do usuário.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-115">Uses the `.Header()` function to include a `Prefer: outlook.timezone` header, which causes the returned events to have their start and end times converted to the user's timezone.</span></span>
        - <span data-ttu-id="eb7c8-116">Usa a `.Top()` função para solicitar no máximo 50 eventos.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-116">Uses the `.Top()` function to request at most 50 events.</span></span>
        - <span data-ttu-id="eb7c8-117">Usa a `.Select()` função para solicitar apenas os campos usados pelo aplicativo.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-117">Uses the `.Select()` function to request just the fields used by the app.</span></span>
        - <span data-ttu-id="eb7c8-118">Usa a `OrderBy()` função para classificar os resultados pela hora de início.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-118">Uses the `OrderBy()` function to sort the results by the start time.</span></span>

1. <span data-ttu-id="eb7c8-119">Salve suas alterações e reinicie o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-119">Save your changes and restart the app.</span></span> <span data-ttu-id="eb7c8-120">Atualize a guia no Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-120">Refresh the tab in Microsoft Teams.</span></span> <span data-ttu-id="eb7c8-121">O aplicativo exibe uma lista de JSON dos eventos.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-121">The app displays a JSON listing of the events.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="eb7c8-122">Exibir os resultados</span><span class="sxs-lookup"><span data-stu-id="eb7c8-122">Display the results</span></span>

<span data-ttu-id="eb7c8-123">Agora você pode exibir a lista de eventos de forma mais amigável.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-123">Now you can display the list of events in a more user friendly way.</span></span>

1. <span data-ttu-id="eb7c8-124">Abra **./pages/index.cshtml** e adicione as seguintes funções dentro da `<script>` marca.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-124">Open **./Pages/Index.cshtml** and add the following functions inside the `<script>` tag.</span></span>

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="RenderHelpersSnippet":::

1. <span data-ttu-id="eb7c8-125">Substitua a função `renderCalendar` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-125">Replace the existing `renderCalendar` function with the following.</span></span>

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="RenderCalendarSnippet":::

1. <span data-ttu-id="eb7c8-126">Salve suas alterações e reinicie o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-126">Save your changes and restart the app.</span></span> <span data-ttu-id="eb7c8-127">Atualize a guia no Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-127">Refresh the tab in Microsoft Teams.</span></span> <span data-ttu-id="eb7c8-128">O aplicativo exibe eventos no calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="eb7c8-128">The app displays events on the user's calendar.</span></span>

    ![Uma captura de tela do aplicativo exibindo o calendário do usuário](images/calendar-view.png)
