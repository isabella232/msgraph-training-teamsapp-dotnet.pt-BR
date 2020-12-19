<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="62ede-101">Nesta seção, você adicionará a capacidade de criar eventos no calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="62ede-101">In this section you will add the ability to create events on the user's calendar.</span></span>

## <a name="create-the-new-event-tab"></a><span data-ttu-id="62ede-102">Criar a nova guia de evento</span><span class="sxs-lookup"><span data-stu-id="62ede-102">Create the new event tab</span></span>

1. <span data-ttu-id="62ede-103">Crie um novo arquivo no diretório **./Pages** chamado **NewEvent. cshtml** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="62ede-103">Create a new file in the **./Pages** directory named **NewEvent.cshtml** and add the following code.</span></span>

    :::code language="razor" source="../demo/GraphTutorial/Pages/NewEvent.cshtml":::

    <span data-ttu-id="62ede-104">Isso implementa um formulário simples e adiciona JavaScript para postar os dados do formulário na API Web.</span><span class="sxs-lookup"><span data-stu-id="62ede-104">This implements a simple form, and adds JavaScript to post the form data to the Web API.</span></span>

## <a name="implement-the-web-api"></a><span data-ttu-id="62ede-105">Implementar a API Web</span><span class="sxs-lookup"><span data-stu-id="62ede-105">Implement the Web API</span></span>

1. <span data-ttu-id="62ede-106">Crie um novo diretório chamado **modelos** na raiz do projeto.</span><span class="sxs-lookup"><span data-stu-id="62ede-106">Create a new directory named **Models** in the root of the project.</span></span>

1. <span data-ttu-id="62ede-107">Crie um novo arquivo no diretório **./Models** chamado **NewEvent.cs** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="62ede-107">Create a new file in the **./Models** directory named **NewEvent.cs** and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/NewEvent.cs" id="NewEventSnippet":::

1. <span data-ttu-id="62ede-108">Abra **./Controllers/CalendarController.cs** e adicione a instrução a seguir `using` na parte superior do arquivo.</span><span class="sxs-lookup"><span data-stu-id="62ede-108">Open **./Controllers/CalendarController.cs** and add the following `using` statement at the top of the file.</span></span>

    ```csharp
    using GraphTutorial.Models;
    ```

1. <span data-ttu-id="62ede-109">Adicione a função a seguir à classe **CalendarController** .</span><span class="sxs-lookup"><span data-stu-id="62ede-109">Add the following function to the **CalendarController** class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="PostSnippet":::

    <span data-ttu-id="62ede-110">Isso permite que um HTTP POST para a API da Web com os campos do formulário.</span><span class="sxs-lookup"><span data-stu-id="62ede-110">This allows an HTTP POST to the Web API with the fields of the form.</span></span>

1. <span data-ttu-id="62ede-111">Salve todas as suas alterações e reinicie o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="62ede-111">Save all of your changes and restart the application.</span></span> <span data-ttu-id="62ede-112">Atualize o aplicativo no Microsoft Teams e selecione a guia **criar evento** . Preencha o formulário e selecione **criar** para adicionar um evento ao calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="62ede-112">Refresh the app in Microsoft Teams, and select the **Create event** tab. Fill out the form and select **Create** to add an event to the user's calendar.</span></span>

    ![Uma captura de tela da guia criar evento](images/create-event.png)
