# 📝 DOCUMENTAÇÃO DA ARQUITETURA (Clean Architecture / MVVM)

Este documento descreve a função de cada projeto e os principais arquivos dentro da Solução `RealTimeApp.sln`, que utiliza Avalonia UI para o cliente mobile e ASP.NET Core para o backend (PostgreSQL/SignalR).

---

## 1. 🌐 RealTimeApp.Domain (O Cérebro do Negócio)

* **Tipo de Projeto:** Biblioteca de Classes (.NET Padrão)
* **Responsabilidade:** Define o *O QUÊ* do sistema (Entidades, Regras de Negócio e Contratos). Não possui dependências externas.

| Caminho do Arquivo | Descrição |
| :--- | :--- |
| `Pedido.cs` | **Entidade Principal:** Define a estrutura de um Pedido (Id, Descrição, Status, DataAtualizacao). É o objeto que será persistido no DB. |
| `PedidoStatus.cs` | **Enums/Value Objects:** Tipos específicos de valor usados no Domínio (Ex: status do pedido). |
| `Interfaces/IPedidoRepository.cs` | **Contrato de Repositório:** Define os métodos necessários para interagir com a persistência de Pedidos. |

---

## 2. 📱 RealTimeApp.Application (A Lógica de Cliente e Apresentação)

* **Tipo de Projeto:** Biblioteca de Classes (.NET Padrão)
* **Responsabilidade:** Contém a lógica de apresentação e orquestração dos dados para a UI (os ViewModels). Lógica de negócios compartilhada entre cliente e servidor.

| Caminho do Arquivo | Descrição |
| :--- | :--- |
| `ViewModels/ViewModelBase.cs` | **Base do MVVM:** Implementa a interface `INotifyPropertyChanged` para atualização automática da UI. |
| `ViewModels/MainViewModel.cs` | **Controle da Tela Principal:** Lida com a busca inicial de dados e se inscreve para atualizações em tempo real. |
| `DTOs/PedidoDto.cs` | **Modelo de Transferência:** Estrutura de dados usada para comunicação através da rede. |

---

## 3. 💾 RealTimeApp.Infrastructure (O Acesso aos Recursos)

* **Tipo de Projeto:** Biblioteca de Classes (.NET Padrão)
* **Responsabilidade:** Contém as implementações concretas (o *COMO*) de acesso a dados (PostgreSQL/EF Core/Npgsql) e recursos externos.

| Caminho do Arquivo | Descrição |
| :--- | :--- |
| `Data/ApplicationDbContext.cs` | **Contexto EF Core:** Mapeia Entidades para o PostgreSQL, gerencia a Connection String e as Migrations. |
| `Data/Repositories/PostgreSqlPedidoRepository.cs` | **Implementação de Contrato:** Implementa a `IPedidoRepository` usando o EF Core/Npgsql. |
| `RealTime/PostgreSqlChangeListener.cs` | **Serviço de Background:** Usa a conexão Npgsql para executar `LISTEN/NOTIFY` do DB, integrando-se ao SignalR. |

---

## 4. ⚙️ RealTimeApp.Server (O Backend - ASP.NET Core API)

* **Tipo de Projeto:** ASP.NET Core Web API (Executável)
* **Responsabilidade:** Hospeda a API REST, o SignalR Hub e configura a infraestrutura (DI, DB).

| Caminho do Arquivo | Descrição |
| :--- | :--- |
| `Program.cs` | **Bootstrapper:** Ponto de entrada da API. Configura DI, EF Core e SignalR. |
| `appsettings.json` | **Configuração:** Armazena parâmetros externos, como a **ConnectionString do PostgreSQL**. |
| `Controllers/PedidosController.cs` | **API REST:** Lida com as requisições HTTP (GET, POST) do cliente. |
| `Hubs/PedidosHub.cs` | **SignalR Hub:** O ponto de conexão persistente para enviar atualizações em tempo real para o Avalonia. |

---

## 5. 🎨 RealTimeApp.Mobile (O Frontend - Avalonia UI)

* **Tipo de Projeto:** Avalonia Cross-Platform Application (Executável)
* **Responsabilidade:** Exibe a Interface do Usuário (UI) no Android, iOS e Desktop.

| Caminho do Arquivo | Descrição |
| :--- | :--- |
| `App.axaml.cs` | **Inicialização da UI:** Configura o tema e o registro de ViewModels (DI) para as Views. |
| `Views/MainView.axaml` | **O Design da Tela:** Arquivo XAML que define os controles visuais e o *Binding* com o `MainViewModel`. |
| `Services/HttpDataService.cs` | **Cliente HTTP:** Classe que usa `HttpClient` para fazer as chamadas REST iniciais para o `RealTimeApp.Server`. |
| `Services/RealTimeService.cs` | **Cliente SignalR:** Classe que mantém a conexão com o `PedidosHub` para receber os dados em tempo real. |

