<!-- markdownlint-disable MD002 MD041 -->

O manifesto do aplicativo descreve como o aplicativo se integra ao Microsoft Teams e é necessário para instalar aplicativos. Nesta seção, você usará o app Studio no cliente Microsoft Teams para gerar um manifesto.

1. Se você ainda não tiver o app Studio instalado no Teams, [Instale-o agora](/microsoftteams/platform/concepts/build-and-test/app-studio-overview).

1. Inicie o app Studio no Microsoft Teams e selecione o **Editor de manifesto**.

1. Selecione **criar um novo aplicativo**.

    ![Uma captura de tela do editor de manifesto no app Studio no Microsoft Teams](images/app-studio-01.png)

1. Na página **detalhes do aplicativo** , preencha os campos obrigatórios.

    > [!NOTE]
    > Você pode usar os ícones padrão na seção **branding** ou carregar seus próprios.

1. No menu à esquerda, selecione **guias** em **recursos**.

1. Selecione **Adicionar** em **Adicionar uma guia pessoal**.

    ![Uma captura de tela da página guias no app Studio](images/app-studio-02.png)

1. Preencha os campos da seguinte maneira, onde `YOUR_NGROK_URL` é a URL de encaminhamento que você copiou na seção anterior. Selecione **salvar** quando terminar.

    - **Nome:**`Create event`
    - **ID da entidade:**`createEventTab`
    - **URL do conteúdo:**`YOUR_NGROK_URL/newevent`

1. Selecione **Adicionar** em **Adicionar uma guia pessoal**.

1. Preencha os campos da seguinte maneira, onde `YOUR_NGROK_URL` é a URL de encaminhamento que você copiou na seção anterior. Selecione **salvar** quando terminar.

    - **Nome:**`Graph calendar`
    - **ID da entidade:**`calendarTab`
    - **URL do conteúdo:**`YOUR_NGROK_URL`

1. No menu à esquerda, selecione **domínios e permissões** em **concluir**.

1. Defina a **ID do aplicativo AAD** para a ID do aplicativo do registro do aplicativo.

1. Defina o campo **logon único** como o URI da ID do aplicativo do registro do aplicativo.

1. No menu à esquerda, selecione **testar e distribuir** em **concluir**. Selecione **baixar**.

1. Crie um novo diretório na raiz do projeto chamado **manifest**. Extraia o conteúdo do arquivo ZIP baixado para esse diretório.
