# 3. Plano de tratamento das ameaças

## 3.1 Abordagem

Para os cenários analisados, foi escolhida a estratégia **mitigar**. O objetivo é reduzir a probabilidade de exploração e/ou o impacto por meio de controles técnicos e operacionais.

As propostas abaixo são recomendações de projeto. A documentação não afirma que os controles já foram implementados.

## RT-001 — Tratamento de Spoofing

**Ameaça relacionada:** TM-001 — Spoofing.

**Estratégia:** Mitigar.

**Risco tratado:** um atacante utiliza um JWT administrativo roubado para assumir a identidade de um administrador.

**Controles definidos**
1. Utilizar HTTPS em toda a comunicação.
2. Proteger o armazenamento e o ciclo de vida dos tokens, evitando exposição a scripts não confiáveis.
3. Definir expiração adequada, rotação e revogação de sessões.
4. Implementar MFA para contas administrativas.
5. Corrigir vulnerabilidades XSS e aplicar práticas seguras de desenvolvimento no front-end.
6. Monitorar sessões e acessos administrativos suspeitos.

**Justificativa:** reduzir a possibilidade de roubo e reutilização de sessões, além de dificultar o acesso administrativo indevido.

**Validação proposta**
- Verificar se a aplicação rejeita tokens expirados, inválidos ou revogados.
- Verificar se operações administrativas exigem uma sessão válida e, quando definido, MFA.
- Avaliar armazenamento e exposição do token no navegador.
- Executar testes de segurança para identificar XSS e validar as medidas de proteção.

## RT-002 — Tratamento de Tampering

**Ameaça relacionada:** TM-002 — Tampering.

**Estratégia:** Mitigar.

**Risco tratado:** um cliente altera o preço ou outros valores enviados pelo front-end durante o checkout.

**Controles definidos**
1. Recalcular o preço no back-end usando os dados oficiais do PostgreSQL.
2. Não confiar em preço ou total enviado pelo navegador.
3. Validar quantidade, estoque, descontos e regras de negócio no servidor.
4. Utilizar transações no banco de dados para preservar consistência.
5. Registrar alterações administrativas de preço e dados críticos.

**Justificativa:** garantir que o valor cobrado seja determinado por regras e dados controlados pelo servidor, preservando a integridade financeira.

**Validação proposta**
- Interceptar uma requisição de checkout em ambiente de teste.
- Modificar o preço e o total enviados pelo cliente.
- Confirmar que o servidor ignora os valores manipulados e calcula o total correto.
- Testar quantidades inválidas, descontos indevidos e inconsistências de estoque.

## RT-003 — Tratamento de Repudiation

**Ameaça relacionada:** TM-003 — Repudiation.

**Estratégia:** Mitigar.

**Risco tratado:** um administrador altera informações críticas e posteriormente nega a autoria da ação.

**Controles definidos**
1. Implementar logs de auditoria detalhados para operações críticas.
2. Registrar identificador do usuário, data e hora, ação, recurso afetado e resultado.
3. Registrar valores anteriores e novos em alterações relevantes.
4. Sincronizar os relógios dos componentes que geram eventos.
5. Restringir o acesso aos registros de auditoria.
6. Centralizar e proteger os registros contra adulteração ou exclusão indevida.
7. Definir política de retenção e monitoramento.
8. Exigir MFA ou reautenticação para operações críticas.

**Justificativa:** garantir rastreabilidade, responsabilização e registros com integridade suficiente para apoiar investigações.

**Validação proposta**
- Executar uma alteração administrativa controlada e verificar se todos os campos necessários foram registrados.
- Confirmar se o evento pode ser associado à identidade autenticada que realizou a ação.
- Verificar sincronização de horário.
- Testar se usuários sem autorização conseguem alterar ou excluir registros.
- Verificar retenção, centralização e geração de alertas conforme a política definida.

## RT-004 — Tratamento de Information Disclosure (IDOR)

**Ameaça relacionada:** TM-004 — Information Disclosure.

**Estratégia:** Mitigar.

**Risco tratado:** um cliente autenticado acessa pedidos pertencentes a outros usuários ao modificar identificadores nas requisições.

**Controles definidos**
1. Implementar autorização por objeto no back-end Flask.
2. Verificar a propriedade do pedido antes de retornar seus dados.
3. Realizar consultas considerando o identificador do pedido e o usuário autenticado.
4. Aplicar controle de acesso em todas as operações de consulta e alteração.
5. Registrar e monitorar tentativas suspeitas de acesso a recursos de terceiros.

**Justificativa:** garantir que usuários autenticados acessem somente os recursos para os quais possuem autorização, preservando a confidencialidade dos dados dos clientes.

**Validação proposta**
- Criar duas contas de teste distintas, A e B.
- Criar um pedido associado à conta B.
- Autenticar como A e tentar acessar o pedido de B, alterando o identificador na requisição.
- Utilizar Burp Suite Repeater ou ferramenta equivalente para repetir a requisição.
- Confirmar que o back-end nega o acesso e não retorna dados do pedido de B.

## RT-005 — Tratamento de Denial of Service

**Ameaça relacionada:** TM-005 — Denial of Service.

**Estratégia:** Mitigar.

**Risco tratado:** volume abusivo de requisições sobrecarrega o back-end e o banco de dados, prejudicando a disponibilidade.

**Controles definidos**
1. Implementar rate limiting por IP, usuário e endpoint.
2. Configurar WAF para filtrar tráfego suspeito.
3. Implementar CDN e cache para conteúdo público, quando aplicável.
4. Configurar alertas para picos anormais de requisições.
5. Implementar bloqueios temporários e adaptativos.
6. Avaliar escalabilidade automática e otimização do banco de dados.

**Justificativa:** reduzir a exposição do back-end a tráfego abusivo, detectar comportamentos anormais e preservar a disponibilidade do serviço.

**Validação proposta**
- Executar testes controlados de carga e segurança em ambiente autorizado, utilizando Locust ou k6.
- Verificar aplicação dos limites de requisição.
- Monitorar latência, erros, uso de CPU/memória e carga do banco de dados.
- Verificar acionamento de alertas e manutenção do acesso para usuários legítimos.

**Limite de segurança:** não realizar testes de negação de serviço contra sistemas de terceiros ou ambientes de produção sem autorização explícita e planejamento.

## RT-006 — Tratamento de Elevation of Privilege

**Ameaça relacionada:** TM-006 — Elevation of Privilege.

**Estratégia:** Mitigar.

**Risco tratado:** um usuário comum executa operações exclusivas de administradores ou ultrapassa o escopo de suas permissões.

**Controles definidos**
1. Implementar RBAC para separar permissões de clientes e administradores.
2. Realizar validação de autorização diretamente no back-end Flask em todas as operações protegidas.
3. Aplicar o princípio do menor privilégio.
4. Implementar MFA para contas administrativas.
5. Exigir reautenticação para operações críticas.
6. Avaliar a separação da interface administrativa e dos acessos de clientes.
7. Registrar e monitorar operações administrativas.
8. Não confiar em informações de perfil ou permissões enviadas pelo front-end.

**Justificativa:** impedir que usuários executem operações incompatíveis com suas permissões, reduzindo o risco de acesso indevido e comprometimento da integridade do sistema.

**Validação proposta**
- Autenticar com uma conta de cliente e tentar executar endpoints administrativos diretamente.
- Tentar alterar campos como `role` ou `is_admin` na requisição.
- Testar contas administrativas com permissões limitadas.
- Confirmar que o back-end bloqueia ações fora do escopo autorizado.
- Verificar se operações administrativas relevantes são registradas.

---

## 3.2 Rastreabilidade entre ameaça e tratamento

| Ameaça | Registro de tratamento | Estratégia |
|---|---|---|
| TM-001 — Spoofing | RT-001 | Mitigar |
| TM-002 — Tampering | RT-002 | Mitigar |
| TM-003 — Repudiation | RT-003 | Mitigar |
| TM-004 — Information Disclosure / IDOR | RT-004 | Mitigar |
| TM-005 — Denial of Service | RT-005 | Mitigar |
| TM-006 — Elevation of Privilege | RT-006 | Mitigar |

## 3.3 Observação sobre risco residual

A implementação de controles não elimina automaticamente o risco. Após a implementação e os testes, a equipe deve reavaliar a probabilidade e o impacto, documentar eventuais limitações e decidir se o risco residual será reduzido, transferido, evitado ou formalmente aceito.
