<!-- markdownlint-disable MD002 MD041 -->

Nesta seção, você adicionará a capacidade de criar eventos no calendário do usuário.

## <a name="create-the-new-event-tab"></a>Criar a nova guia de evento

1. Crie um novo arquivo no diretório **./Pages** chamado **NewEvent. cshtml** e adicione o código a seguir.

    :::code language="razor" source="../demo/GraphTutorial/Pages/NewEvent.cshtml":::

    Isso implementa um formulário simples e adiciona JavaScript para postar os dados do formulário na API Web.

## <a name="implement-the-web-api"></a>Implementar a API Web

1. Crie um novo diretório chamado **modelos** na raiz do projeto.

1. Crie um novo arquivo no diretório **./Models** chamado **NewEvent.cs** e adicione o código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Models/NewEvent.cs" id="NewEventSnippet":::

1. Abra **./Controllers/CalendarController.cs** e adicione a instrução a seguir `using` na parte superior do arquivo.

    ```csharp
    using GraphTutorial.Models;
    ```

1. Adicione a função a seguir à classe **CalendarController** .

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="PostSnippet":::

    Isso permite que um HTTP POST para a API da Web com os campos do formulário.

1. Salve todas as suas alterações e reinicie o aplicativo. Atualize o aplicativo no Microsoft Teams e selecione a guia **criar evento** . Preencha o formulário e selecione **criar** para adicionar um evento ao calendário do usuário.

    ![Uma captura de tela da guia criar evento](images/create-event.png)
