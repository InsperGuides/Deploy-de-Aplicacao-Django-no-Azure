| **Lab**         |     **Data**   |
|  :----:     |  :----:    |
| Laboratório Ágil     | 2023       |

# Deploy-de-Aplicacao-Django-no-Azure
Guia de como ajustar sua aplicação para o deploy dentro do Microsoft Azure
===
## :warning: É RECOMENDADO SEGUIR ESTE GUIA EM UMA VERSÃO NOVA DA SUA APLICAÇÃO NO GITHUB PARA VOCÊ NÃO QUEBRAR SUA VERSÃO DE PRODUÇÃO(EM CASO DE DÚVIDAS, PESQUISE COMO CRIAR UM FORK DO SEU REPOSITÓRIO). <br /><br />Passo 1: Criando seu Web App no Azure:

### 1.a: Login:
Para começarmos, iremos fazer os ajustes necessários dentro do Microsoft Azure com sua conta. Para isso, acesse o link abaixo e **logue com sua conta do Insper no portal**. <br> https://portal.azure.com/#home

### 1.b: Criando o Web App:
Após acessar o portal da Azure, procure pela opção **Create a Resource**, e dentro desta opção, pesquise pela opção **Web App + Database**

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/a04840f5-774f-46ef-b0ea-d5331ac8180b)

Em seguida iremos configurar o Web App com as informações abaixo

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/08b440c3-0c24-4131-86a2-47f5f634f0a0)

**1 - Subscription: Selecione sua assinatura **Azure for Students**;**

**2 - Resource Group: Um novo grupo será criado automaticamente com o nome do seu Web App por padrão;**

**3 - Region: Mantenha a região mais próxima a você, que provavelmente já estará por padrão na configuração;**

**4 - Name: Nome da sua aplicação(Este nome deve ser único e irá definir a URL do seu App,pense com carinho;**

**5 - Runtime Stack: O Ambiente da sua Aplicação. Selecione o Python 3.11 ou o mais atual;**

**6 - Engine: Selecione o POSTGRESQL - Flexible Server;**

**7 -Server Name: pode manter o padrão criado pelo Azure;**

**8 - Database Name: pode manter o padrão criado pelo Azure;**

**9 - Hosting: Selecione o plano Basic.**

Após ajustar todas as configurações, clique no botão **Review + Create**, e após confirmar que todos os dados estão corretos, clique em **Create** para finalizar a configuração. Em seguida, o Azure irá iniciar o deploy da sua aplicação, e este processo pode levar vários minutos para ser finalizado.

Quando o processo de Deploy concluir, você tera um novo Resource Group no seu Azure similar como o abaixo. Para os próximos passos, iremos trabalhar dentro do **App Service** dentro deste grupo.

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/0244be5f-f630-489e-b129-33d82c789ea9)


### 1.c: Configurando o App Service

Agora que temos nosso Web App criado dentro do Azure, precisamos configurar algumas partes dele para que o nosso projeto Django funcione corretamente. Primeiro, vamos configurar as **Application Settings** para podermos conectar o banco de dados do PostgreSQL ao projeto, assim como fizemos com os containers anteriormente.

Para acessar, procura no painel esquerdo, na área de **Settings**, pela opção **Configuration**. Você deverá encontrar uma tela similar a abaixo:

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/a9d0e81e-9d67-4218-b2e3-d108a60c63f0)

Provavelmente você terá apenas uma Application Setting criada no seu painel, como nome de **AZURE_POSTGRESQL_CONNECTIONSTRING**. Esta configuração possui uma string contendo todas as informações necessárias para a aplicação conectar à base de dados do Azure. Porém, tratar esta string dentro do Django pode ser um tanto chato, então iremos por um caminho diferente, criando diversas Applications Settings, uma para cada dado que precisaremos dentro do nosso arquivo *settings.py* da aplicação(iremos configurar isto futuramente neste guia).

Agora, o que devemos fazer, é selecionar a opção **Advanced Edit**, que nos irá apresentar um arquivo JSON para modificar. Copie o código abaixo no seu JSON após a parte da Connection String existente:(**Não apague sua Connection String ainda, iremos utilizar informações dela primeiro**)

```
,
{
    "name": "ALLOWED_HOSTS",
    "value": "127.0.0.1 [::1] <link do App>",
    "slotSetting": true
  },
  {
    "name": "CSRF_TRUSTED_ORIGINS",
    "value": "https://<link do App>",
    "slotSetting": true
  },
  {
    "name": "DBHOST",
    "value": "<host>",
    "slotSetting": true
  },
  {
    "name": "DBNAME",
    "value": "<dbname>",
    "slotSetting": true
  },
  {
    "name": "DBPASS",
    "value": "<password>",
    "slotSetting": true
  },
  {
    "name": "DBUSER",
    "value": "<user>",
    "slotSetting": true
  },
  {
    "name": "DEGUB",
    "value": "1",
    "slotSetting": true
  },
  {
    "name": "SECRET_KEY",
    "value": "w7a8a@lj8nax7tem0caa2f2rjm2ahsascyf83sa5alyv68vea",
    "slotSetting": true
  },
  {
    "name": "SECURE_SSL_REDIRECT",
    "value": "0",
    "slotSetting": true
  }
```
Após copiar o código acima, você irá substituir os campos temporários `<exemplo>` pelas informações dentro da Connection String já criada pelo Azure. Depois deste passo, clique em **Ok** e, em seguida, clique em **Save** no topo da tela de Application Settings para finalizar a edição. Você deverá ter todas as variáveis de ambiente agora como a imagem acima estava mostrando.

**Obs: o link do seu App se encontra na parte de *Overview* da página**
