<!-- markdownlint-disable MD002 MD041 -->

Nesta seção, você incorporará o Microsoft Graph no aplicativo. Para este aplicativo, você usará a [biblioteca de cliente do Microsoft Graph para .net](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para fazer chamadas para o Microsoft Graph.

# <a name="get-a-calendar-view"></a>Obter um modo de exibição de calendário

Um modo de exibição de calendário é um conjunto de eventos do calendário do usuário que ocorrem entre dois pontos de tempo. Você o usará para obter os eventos do usuário para a semana atual.

1. Abra **./Controllers/CalendarController.cs** e adicione a função a seguir à classe **CalendarController** .

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetStartOfWeekSnippet":::

1. Adicione a seguinte função para lidar com exceções retornadas pelas chamadas do Microsoft Graph.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="HandleGraphExceptionSnippet":::

1. Substitua a função `Get` existente pelo seguinte.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetSnippet" highlight="2,14-57":::

    Revise as alterações. Esta nova versão da função:

    - Retorna `IEnumerable<Event>` em vez de `string` .
    - Obtém as configurações de caixa de correio do usuário usando o Microsoft Graph.
    - Usa o fuso horário do usuário para calcular o início e o fim da semana atual.
    - Obtém uma visão de calendário
        - Usa a `.Header()` função para incluir um `Prefer: outlook.timezone` cabeçalho, o que faz com que os eventos retornados tenham seus horários de início e término convertidos no fuso horário do usuário.
        - Usa a `.Top()` função para solicitar no máximo 50 eventos.
        - Usa a `.Select()` função para solicitar apenas os campos usados pelo aplicativo.
        - Usa a `OrderBy()` função para classificar os resultados pela hora de início.

1. Salve suas alterações e reinicie o aplicativo. Atualize a guia no Microsoft Teams. O aplicativo exibe uma lista de JSON dos eventos.

## <a name="display-the-results"></a>Exibir os resultados

Agora você pode exibir a lista de eventos de forma mais amigável.

1. Abra **./pages/index.cshtml** e adicione as seguintes funções dentro da `<script>` marca.

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="RenderHelpersSnippet":::

1. Substitua a função `renderCalendar` existente pelo seguinte.

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="RenderCalendarSnippet":::

1. Salve suas alterações e reinicie o aplicativo. Atualize a guia no Microsoft Teams. O aplicativo exibe eventos no calendário do usuário.

    ![Uma captura de tela do aplicativo exibindo o calendário do usuário](images/calendar-view.png)
