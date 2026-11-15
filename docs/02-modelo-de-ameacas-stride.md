# 2. Identificação de ameaças — STRIDE

## 2.1 Metodologia

Foi utilizada a metodologia **STRIDE** para identificar ameaças em seis categorias:

| Categoria | Significado | Propriedade / risco típico |
|---|---|---|
| Spoofing | Falsificação de identidade | Autenticidade |
| Tampering | Adulteração de dados | Integridade |
| Repudiation | Repúdio de ações | Rastreabilidade / responsabilização |
| Information Disclosure | Divulgação indevida de informações | Confidencialidade |
| Denial of Service | Negação de serviço | Disponibilidade |
| Elevation of Privilege | Elevação de privilégio | Autorização |

Os cenários abaixo são hipóteses plausíveis de exploração construídas a partir das informações disponíveis. A presença de uma ameaça no modelo não significa que a vulnerabilidade tenha sido confirmada em um sistema real.

---

## TM-001 — Spoofing (falsificação de identidade)

**Cenário:** um atacante obtém o JWT de um administrador legítimo e utiliza o token para se passar por ele na ShopFlow.

**Ativos ameaçados**
- Identidade administrativa.
- Dados de clientes.
- Produtos e pedidos.
- Funcionalidades administrativas.

**Componentes envolvidos**
- Navegador e front-end React.
- Back-end Flask.
- Mecanismo de autenticação e gerenciamento de sessão.

**Impacto potencial**

Acesso administrativo não autorizado, manipulação de dados, exposição de informações, prejuízo financeiro e dano reputacional.

**Controles propostos**
- Utilizar HTTPS em toda a comunicação.
- Proteger adequadamente o armazenamento e o ciclo de vida dos tokens.
- Definir expiração curta e mecanismos de renovação, rotação e revogação de sessões.
- Reduzir o risco de XSS e evitar exposição de tokens a scripts não confiáveis.
- Exigir MFA para contas administrativas.
- Monitorar acessos e comportamentos anormais, como sessões simultâneas ou origens incomuns.

**Ponto de atenção:** RBAC controla o que uma identidade pode fazer, mas não impede sozinho o uso indevido de um token válido roubado.

---

## TM-002 — Tampering (adulteração de dados)

**Cenário:** durante o checkout, um cliente intercepta a requisição HTTP e modifica o preço enviado ao back-end. Se o servidor confiar no valor recebido sem consultar o preço oficial, poderá registrar a compra por um valor inferior ao correto.

**Vulnerabilidade hipotética**

Confiança indevida em dados enviados pelo front-end e ausência de recálculo dos valores no servidor.

**Ativos ameaçados**
- Cadastro de produtos e preços.
- Pedidos.
- Integridade financeira das transações.

**Componente envolvido**
- Back-end Flask e sua integração com o PostgreSQL.

**Impacto potencial**

Fraude financeira, inconsistência nos registros, prejuízo e dano reputacional.

**Controles propostos**
- Nunca utilizar o preço enviado pelo navegador como fonte autoritativa.
- Consultar no PostgreSQL o preço vigente de cada produto.
- Calcular o total do pedido exclusivamente no back-end.
- Validar quantidade, estoque, descontos e regras de negócio.
- Garantir consistência das operações com transações no banco de dados.
- Registrar alterações administrativas de preço.

**Princípio central:** o cliente pode enviar a identificação do produto e a quantidade desejada; o servidor deve determinar o preço e validar se a compra é permitida.

---

## TM-003 — Repudiation (repúdio de ações)

**Cenário:** um administrador altera o preço de um produto de R$ 500 para R$ 50 e posteriormente nega ter realizado a operação. Sem registros confiáveis de autoria e histórico, a empresa não consegue comprovar a ação.

**Ativos ameaçados**
- Integridade dos dados.
- Histórico de alterações.
- Registros de auditoria.

**Vulnerabilidade hipotética**

Ausência de trilha de auditoria suficientemente detalhada e protegida para comprovar a autoria e o contexto de operações administrativas.

**Impacto potencial**

Prejuízo financeiro, dificuldade de investigação, ausência de responsabilização e perda de confiança nos registros do sistema.

**Controles propostos**
- Registrar eventos de auditoria com identificador do usuário, data e hora, ação executada e recurso afetado.
- Registrar valores anteriores e novos para alterações relevantes.
- Sincronizar os relógios dos componentes que geram eventos.
- Restringir o acesso aos registros de auditoria.
- Centralizar e proteger os logs contra alteração ou exclusão indevida.
- Definir política de retenção e monitoramento.
- Exigir MFA ou reautenticação para operações críticas.

**Ponto de atenção:** pedir confirmação de identidade pode reforçar a segurança de uma operação, mas não substitui uma trilha de auditoria protegida. Logs comuns também não são automaticamente evidências confiáveis: precisam ter autoria, integridade, contexto e controles de acesso.

---

## TM-004 — Information Disclosure (divulgação indevida de informações)

**Cenário:** um cliente autenticado modifica o identificador de um pedido em uma requisição HTTP e consegue consultar informações pertencentes a outro usuário.

**Tipo de falha relacionada**

IDOR (*Insecure Direct Object Reference*) / controle de acesso quebrado.

**Vulnerabilidade hipotética**

O back-end verifica que o usuário está autenticado, mas não verifica se ele tem autorização para acessar aquele pedido específico.

**Ativos ameaçados**
- Dados pessoais.
- Histórico de compras.
- Endereços e informações dos pedidos.

**Impacto potencial**

Exposição de dados pessoais, fraudes, engenharia social, danos reputacionais e possíveis consequências regulatórias.

**Controles propostos**
- Implementar autorização por objeto no back-end.
- Verificar a propriedade do pedido antes de retornar seus dados.
- Consultar o pedido considerando tanto seu identificador quanto o usuário autenticado.
- Aplicar controle de acesso em todas as operações de consulta e alteração.
- Registrar e monitorar tentativas suspeitas de acesso a recursos de terceiros.

**Exemplo conceitual em Flask:**

```python
order = Order.query.filter_by(
    id=order_id,
    user_id=current_user.id
).first()

if not order:
    return {"error": "Pedido não encontrado"}, 404
```

O exemplo é ilustrativo e pressupõe que `current_user` tenha sido obtido por um mecanismo de autenticação confiável. A autorização deve ocorrer no servidor.

---

## TM-005 — Denial of Service (negação de serviço)

**Cenário:** um atacante utiliza um script para enviar milhares de requisições simultâneas ao endpoint de produtos, sobrecarregando o back-end Flask e o PostgreSQL.

**Vulnerabilidade hipotética**

Ausência ou insuficiência de mecanismos de limitação e controle de requisições nos endpoints expostos.

**Ativos ameaçados**
- Disponibilidade do e-commerce.
- Back-end Flask.
- PostgreSQL.
- Infraestrutura de hospedagem.

**Impacto potencial**

Indisponibilidade do sistema, perda de vendas, prejuízo financeiro e danos reputacionais.

**Controles propostos**
- Implementar rate limiting por IP, usuário e endpoint.
- Configurar WAF para filtrar tráfego suspeito.
- Utilizar CDN e cache para conteúdo público, quando aplicável.
- Configurar alertas para picos anormais de requisições.
- Implementar bloqueios temporários e adaptativos.
- Avaliar escalabilidade automática e otimização do banco de dados.

**Ponto de atenção:** cache no navegador, isoladamente, não impede que um atacante envie requisições diretamente à API. CDN, cache no servidor e controles de borda podem ajudar a reduzir a carga, conforme o tipo de endpoint e conteúdo.

---

## TM-006 — Elevation of Privilege (elevação de privilégio)

**Cenário:** um cliente autenticado identifica uma funcionalidade administrativa, como alteração de preços ou exclusão de produtos, e tenta acessá-la diretamente por meio de uma requisição HTTP à API.

**Criticidade potencial**

Alta, pois uma falha pode permitir que usuários comuns executem operações reservadas a administradores.

**Ativos ameaçados**
- Dados de produtos e preços.
- Pedidos e informações dos clientes.
- Contas e permissões administrativas.
- Integridade das operações do e-commerce.

**Vulnerabilidade hipotética**

Falha nos mecanismos de autorização do back-end Flask, permitindo que um usuário com privilégios comuns execute operações exclusivas de administradores.

**Impacto potencial**
- Alteração indevida de preços e informações de produtos.
- Acesso não autorizado a pedidos de outros clientes.
- Exclusão ou modificação de registros.
- Fraudes e prejuízos financeiros.
- Comprometimento da integridade e da confiabilidade do sistema.

**Controles propostos**
- Implementar RBAC para separar permissões de clientes e administradores.
- Validar permissões diretamente no back-end Flask em todas as operações protegidas.
- Aplicar o princípio do menor privilégio.
- Nunca confiar em informações de perfil ou permissões enviadas pelo front-end.
- Exigir MFA para contas administrativas.
- Exigir reautenticação para operações críticas.
- Registrar e monitorar operações administrativas.
- Avaliar a separação da interface administrativa, sem tratá-la como substituta da autorização no servidor.

---

## 2.2 Análises de Aprendizados

- **IDOR e elevação de privilégio podem se relacionar**, mas não são sinônimos. IDOR trata do acesso indevido a um objeto; elevação de privilégio trata da obtenção ou execução de capacidades além das permissões atribuídas.
- **Autenticação não é autorização.** Autenticação identifica quem está acessando; autorização verifica o que essa identidade pode fazer e quais recursos pode acessar.
- **Validações no front-end não são controles de segurança suficientes.** O back-end deve validar entradas, regras de negócio e permissões.
- As ameaças identificadas não representam uma lista exaustiva. A análise deve ser revisada conforme a arquitetura e os requisitos evoluam.
