graph LR

%% ControleOnline — Focus MVP Restaurante (canais→pedido→pagamento→KDS→entrega→backoffice)

subgraph Canais_Handshake
  direction TB
  dispatch_motoboy[Despacho Motoboy (coleta item pronto / diária)]
  channel_physical[Canal: Recepção / Mesa / Balcão]
  channel_phone[Canal: Telefone]
  channel_whatsapp[Canal: WhatsApp]
  channel_marketplace[Canal: Cardápio/Loja (Web/App)]
  channel_marketplaces[Canal: iFood / Keeta / 99 (adapters)]
  channel_ai[Canal: Agente IA / Chatbot]
end

subgraph Apps
  ppc_community_master[Ppc Community Master]
  pos_community_master[Pos Community Master]
  crm_community_master[Crm Community Master]
  marketplace_community_master[Marketplace Community Master]
  checkout_community_master[Checkout Community Master]
  delivery_community_master[Delivery Community Master]
  whaticket_community_master[Whaticket Community Master]
end

subgraph UI_Modulos
  ui_common_master((ui-common))
  ui_layout_master((ui-layout))
  ui_login_master((ui-login))
  ui_translate_master((ui-translate))
  ui_orders_master((ui-orders))
  ui_products_master((ui-products))
  ui_shop_master((ui-shop))
  ui_financial_master((ui-financial))
  ui_people_master((ui-people))
  ui_ppc_master((ui-ppc))
  ui_manager_master((ui-manager))
  ui_crm_master((ui-crm))
  ui_contracts_master((ui-contracts))
  ui_customers_master((ui-customers))
  ui_tasks_master((ui-tasks))
  ui_users_master((ui-users))
  ui_dashboard_master((ui-dashboard))
  ui_support_master((ui-support))
  ui_logistic_master((ui-logistic))
end

subgraph Backend
  api_community_master[api-community (Symfony app)]
  api_admin_master[api-admin (Symfony app)]
  api_whatsapp_master[api-whatsapp]
  api_whisper_master[api-whisper]
  api_platform_common_master[[api-platform-common]]
  api_platform_users_master[[api-platform-users]]
  api_platform_multi_tenancy_master[[api-platform-multi-tenancy]]
  api_platform_people_master[[api-platform-people]]
  api_platform_products_master[[api-platform-products]]
  api_platform_orders_master[[api-platform-orders]]
  api_platform_financial_master[[api-platform-financial]]
  api_platform_queue_master[[api-platform-queue]]
  api_platform_logistic_master[[api-platform-logistic]]
  api_platform_tasks_master[[api-platform-tasks]]
  api_platform_integration_master[[api-platform-integration]]
  api_platform_websocket_server_master[[api-platform-websocket-server]]
end

subgraph SDKs
  messages_sdk_php_master{messages-sdk-php}
  whatsapp_sdk_php_master{whatsapp-sdk-php}
  braspag_split_sdk_php_main{braspag-split-sdk-php}
  react_native_cielo_payment_master{@controleonline/react-native-cielo-payment}
  react_native_infinitepay_payment_master{@controleonline/react-native-infinitepay-payment}
end

%% Handshake/entrada (canais)
channel_physical --> pos_community_master
channel_physical --> ppc_community_master
channel_phone --> crm_community_master
channel_whatsapp --> whaticket_community_master
channel_whatsapp --> api_whatsapp_master
channel_marketplace --> marketplace_community_master
channel_marketplace --> checkout_community_master
channel_ai --> api_whisper_master
channel_marketplaces -. adapter .-> api_platform_integration_master

%% Cadeia canônica (conceitual)
channel_marketplace --> order_node[Pedido (Order)]
channel_physical --> order_node
channel_whatsapp --> order_node
channel_marketplaces --> order_node
order_node --> pay_node[Pagamento (Invoice/Payment)]
pay_node --> kds_node[KDS / Produção (Queue/Display)]
kds_node --> del_node[Entrega/Retirada (Logistic)]
del_node --> back_node[Backoffice (Dash/Finance/Produtos)]
back_node --> post_feedback[Feedback / Avaliação]

%% Implementação no ecossistema (repos)
marketplace_community_master --> ui_shop_master
marketplace_community_master --> ui_orders_master
checkout_community_master --> ui_shop_master
checkout_community_master --> ui_orders_master
pos_community_master --> ui_orders_master
pos_community_master --> ui_financial_master

whaticket_community_master --> api_whatsapp_master
api_whatsapp_master --> whatsapp_sdk_php_master
whatsapp_sdk_php_master --> messages_sdk_php_master

ui_orders_master --> api_community_master
ui_shop_master --> api_community_master
ui_products_master --> api_community_master
ui_financial_master --> api_community_master

api_community_master --> api_platform_orders_master
api_community_master --> api_platform_products_master
api_community_master --> api_platform_people_master
api_community_master --> api_platform_financial_master
api_community_master --> api_platform_queue_master
api_community_master --> api_platform_logistic_master
api_community_master --> api_platform_users_master
api_community_master --> api_platform_multi_tenancy_master
api_community_master --> api_platform_common_master

api_platform_financial_master --> braspag_split_sdk_php_main
pos_community_master --> react_native_cielo_payment_master
ppc_community_master --> react_native_infinitepay_payment_master

ppc_community_master --> ui_ppc_master
ui_ppc_master --> api_community_master
api_platform_websocket_server_master -. realtime .-> api_platform_queue_master

delivery_community_master --> ui_logistic_master
ui_logistic_master --> api_community_master
delivery_community_master --> ui_orders_master

api_admin_master --> api_platform_common_master
api_admin_master --> api_platform_users_master
api_admin_master --> api_platform_people_master

ui_dashboard_master --> api_community_master
ui_support_master --> api_community_master
crm_community_master --> ui_support_master
crm_community_master --> ui_crm_master

%% Motoboy (hoje e futuro)
del_node --> dispatch_motoboy
dispatch_motoboy -. hoje .-> delivery_community_master
dispatch_motoboy -. futuro: carteiras .-> api_platform_financial_master
