# App_ReidosMotores

O Rei Avisos é a solução mobile oficial da Rei dos Motores. Ele atua como uma "casca nativa" moderna para o sistema legado da empresa (PHP), adicionando capacidades que a web sozinha não consegue entregar: Notificações Push Individuais e presença na tela inicial do colaborador.

🏗 Arquitetura do Projeto
O projeto utiliza a estratégia de App Híbrido com Remote Loader. O aplicativo Android não contém a lógica de negócio; ele carrega o sistema web existente dentro de uma WebView turbinada, injetando funcionalidades nativas.

Shutterstock

Fluxo de Dados (A "Ponte")
Login: O colaborador acessa o App e faz login no sistema PHP existente.

Captura de Identidade: Um script JS injetado (window.ReiAvisos) solicita ao Capacitor o Token FCM (Identidade única do aparelho).

Vínculo: O App envia esse Token silenciosamente para o backend (salvar_token.php), que o grava no banco de dados MySQL vinculado ao ID do usuário.

Disparo: Quando o sistema precisa avisar algo (ex: "Sua senha venceu"), o PHP busca o token desse usuário e dispara a notificação via Firebase.

🚀 Tecnologias
Mobile (Frontend Nativo)
Capacitor 6: Runtime para execução do Web App.

Plugin @capacitor-firebase/messaging: Gerenciamento de tokens e permissões de notificação.

Android Studio: Compilação e assinatura do APK.

Backend (Sistema Legado)
PHP: Sistema ERP/Intranet da empresa.

MySQL: Armazena o vínculo id_usuario <-> device_token.

Firebase Admin SDK (Python/PHP): Serviço responsável por autenticar no Google e despachar as mensagens.

Nuvem
Firebase Cloud Messaging (FCM): Motor de entrega das notificações Push.

📂 Estrutura do Projeto Mobile
Ao contrário de um app tradicional, a pasta www está vazia ou contém apenas uma página de "Sem Conexão", pois o conteúdo vem da nuvem.

Bash

projeto-aviso-hibrido/
├── android/                 # Projeto Nativo Android
├── capacitor.config.json    # ⚙️ Configura que aponta para a URL do PHP
├── package.json             # Dependências (Capacitor + Plugins)
└── ...
Configuração Chave (capacitor.config.json)
O segredo da integração está no apontamento do servidor:

JSON

{
  "appId": "com.reimotores.avisos",
  "appName": "Rei Avisos",
  "webDir": "www",
  "server": {
    "url": "https://sistema.reimotores.com.br",
    "androidScheme": "https",
    "cleartext": true
  }
}
🔌 Guia de Integração (Para o Backend)
Para que o sistema PHP converse com o App, dois arquivos são necessários no servidor:

1. O Receptor (salvar_token.php)
Script PHP simples que recebe um POST JSON e salva no banco.

PHP

// Exemplo simplificado
$sql = "UPDATE usuarios SET device_token = ? WHERE id = ?";
2. O Injetor (Frontend JS)
Snippet a ser adicionado no header.php ou footer.php do sistema:

JavaScript

import { FirebaseMessaging } from 'https://cdn.jsdelivr.net/npm/@capacitor-firebase/messaging@6.0.0/+esm';

// Detecta se está rodando no App
if (window.Capacitor) {
    FirebaseMessaging.requestPermissions().then(() => {
        FirebaseMessaging.getToken().then(result => {
            // Envia result.token para salvar_token.php
        });
    });
}
🛠 Como Rodar e Atualizar
Pré-requisitos
Node.js instalado.

Acesso ao código fonte do sistema PHP (para inserir os scripts).

Comandos Úteis
Atualizar Configurações: Se mudar a URL ou ícone:

Bash

npx cap copy
Sincronizar Plugins: Se instalar novos recursos nativos (Câmera, GPS):

Bash

npx cap sync
Abrir Projeto: Para gerar o APK ou rodar no emulador:

Bash

npx cap open android
✅ Checklist de Implementação
[x] Criação do projeto Capacitor.

[x] Configuração do google-services.json (Assinatura do App).

[x] Personalização (Ícone e Nome "Rei Avisos").

[x] Redirecionamento para URL de Produção (PHP).

[ ] Implementação do salvar_token.php no servidor.

[ ] Inserção do script JS no header.php.

Desenvolvido pela equipe de TI da Rei dos Motores.
