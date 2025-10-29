# 📝 DOCUMENTAÇÃO DA ARQUITETURA (Clean Architecture / MVVM)

**Solução:** `RealTimeApp.sln`

**Objetivo Estratégico do MVP:** Entregar um aplicativo móvel (Avalonia para iOS/Android) focado em Tomada de Decisão (Venda/Margem em tempo real) e Automação de Processos Críticos (Captura Móvel e Classificação Mercadológica via IA).

---

## 1. 🌐 RealTimeApp.Domain (O Cérebro do Negócio)

* **Tipo de Projeto:** Biblioteca de Classes (.NET Padrão)
* **Responsabilidade:** Definir as regras e as estruturas de dados do negócio.

| Caminho do Arquivo | Descrição |
| :--- | :--- |
| `Pedido.cs` | Entidade para rastreio de pedidos e O.S. |
| `CategoriaMercadologica.cs` | **NOVO:** Entidade de auto-referência para modelar a hierarquia de 4 níveis (conforme exemplo). |
| `Produto.cs` | Entidade central que será alimentada pela automação (incluir campos para Imagem e Cód. de Barras). |
| `Interfaces/IPedidoRepository.cs` | Contrato de acesso a dados para Pedidos. |

---

## 2. 📱 RealTimeApp.Application (A Lógica de Cliente e Apresentação)

* **Tipo de Projeto:** Biblioteca de Classes (.NET Padrão)
* **Responsabilidade:** Contém a lógica de apresentação (ViewModels) e orquestração de dados.

| Caminho do Arquivo | Descrição |
| :--- | :--- |
| `ViewModels/MainViewModel.cs` | Dashboard Executivo: **Binding** para Venda Líquida e Margem em tempo real. |
| `ViewModels/NewProductViewModel.cs` | **NOVO:** Lógica para controle da câmera, envio de imagem/código de barras e exibição da sugestão de categoria da IA. |
| `DTOs/FinanceiroDto.cs` | Modelo de transferência para métricas (Venda Líquida, Margem). |
| `DTOs/CategoriaSugestaoDto.cs` | Modelo de transferência para a resposta da IA. |

---

## 3. 💾 RealTimeApp.Infrastructure (O Acesso aos Recursos)

* **Tipo de Projeto:** Biblioteca de Classes (.NET Padrão)
* **Responsabilidade:** Contém as implementações concretas de DB (PostgreSQL) e integração com IA/Serviços.

| Caminho do Arquivo | Descrição |
| :--- | :--- |
| `Data/ApplicationDbContext.cs` | Mapeamento EF Core para Pedidos, Produtos e **CategoriaMercadologica**. |
| `RealTime/PostgreSqlChangeListener.cs` | Serviço `LISTEN/NOTIFY` para detectar alterações e enviar **Margem/Venda** pelo SignalR. |
| `Services/MlIntegrationService.cs` | **NOVO:** Lógica de comunicação com o modelo de Machine Learning para a classificação mercadológica. |
| `Data/Repositories/...` | Implementações dos contratos de repositório. |

---

## 4. ⚙️ RealTimeApp.Server (O Backend - ASP.NET Core API)

* **Tipo de Projeto:** ASP.NET Core Web API (Executável)
* **Responsabilidade:** Ponto de entrada da rede, hospedagem de APIs, SignalR e lógica de IA/Infraestrutura.

### Operações Essenciais do Backend (Ampliado)

| Componente | Operação | Responsabilidade |
| :--- | :--- | :--- |
| `FinanceiroHub.cs` (SignalR) | `SendMetrics` | Distribui **Venda Líquida e Margem** em tempo real para o Dashboard do CEO. |
| `ProdutosController.cs` (REST) | `POST /api/produtos/captura` | Recebe a imagem e o código de barras do Avalonia e inicia o processo de IA. |
| `AiController.cs` (REST) | `GET /api/ai/sugestao` | Endpoint para o **`MlIntegrationService`** retornar a sugestão de categoria de 4 níveis. |
| `PostgreSqlChangeListener.cs` | `LISTEN/NOTIFY` | Monitora o DB por novas vendas (para o cálculo da Margem) e por novos produtos. |

---

## 5. 🎨 RealTimeApp.Mobile (O Frontend - Avalonia UI)

* **Tipo de Projeto:** Avalonia Cross-Platform Application (Executável)
* **Responsabilidade:** Exibir a UI no Android, iOS e lidar com a experiência de captura móvel.

| Caminho do Arquivo | Descrição |
| :--- | :--- |
| `Views/MainView.axaml` | Dashboard Executivo: Exibição da Venda Líquida e Margem (Binding com `MainViewModel`). |
| `Views/NewProductView.axaml` | **NOVO:** View para controle da câmera, visualização do código de barras escaneado e apresentação da sugestão de categoria da IA para validação do usuário. |
| `Services/CameraService.cs` | **NOVO:** Utiliza `Avalonia.Essentials` para acesso nativo à câmera e escaneamento de código de barras. |
| `Services/RealTimeService.cs` | Conexão com o **`FinanceiroHub`** para as métricas executivas. |

---
