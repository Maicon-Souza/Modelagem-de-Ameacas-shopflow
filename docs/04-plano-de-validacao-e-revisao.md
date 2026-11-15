# 4. Plano de validação e revisão

## 4.1 Objetivo

A quarta pergunta da abordagem de Adam Shostack é: **“Fizemos um bom trabalho?”**

Esta etapa verifica se o modelo está coerente, se as ameaças relevantes foram consideradas e se os controles propostos podem ser testados. A revisão não deve ser entendida como uma garantia de que todas as ameaças foram encontradas.

## 4.2 Plano de testes proposto

| ID | Cenário de teste | Resultado esperado | Ferramentas / abordagem |
|---|---|---|---|
| VAL-001 | Uso de JWT expirado, inválido ou revogado | Requisição rejeitada | Testes de API |
| VAL-002 | Tentativa de XSS para acessar sessão/token | Payload não consegue obter ou reutilizar credenciais; controles aplicáveis são acionados | OWASP ZAP e testes manuais |
| VAL-003 | Alterar preço enviado no checkout | Back-end recalcula o valor usando dados oficiais | Burp Suite Repeater / testes de API |
| VAL-004 | Alterar quantidade, desconto ou estoque | Regras de negócio são validadas no servidor | Testes automatizados |
| VAL-005 | Executar alteração administrativa | Evento de auditoria completo e protegido é gerado | Inspeção de logs e testes de permissão |
| VAL-006 | Tentar alterar ou excluir logs sem autorização | Operação negada e, quando aplicável, registrada | Testes de acesso e integridade |
| VAL-007 | Cliente A acessar pedido do cliente B | Acesso negado sem exposição de dados | Burp Suite Repeater |
| VAL-008 | Cliente acessar endpoint administrativo | Operação negada no back-end | Testes de API |
| VAL-009 | Alterar `role` ou `is_admin` no cliente | Permissão não é elevada | Testes de API |
| VAL-010 | Gerar carga controlada em endpoint público | Limites, alertas e capacidade de recuperação funcionam conforme critérios definidos | Locust ou k6 |

**Atenção:** os testes devem ser realizados apenas em ambiente próprio ou com autorização explícita. Testes de carga devem ter limites definidos para não causar indisponibilidade não planejada.

## 4.3 Critérios para considerar um controle validado

Um controle só deve ser marcado como validado após haver evidência verificável. Exemplos:

- Caso de teste documentado.
- Resultado esperado definido previamente.
- Resultado observado registrado.
- Evidência técnica adequada, sem expor segredos ou dados pessoais.
- Responsável e data da execução.
- Registro de falhas e ações corretivas.

Uma proposta de controle ou um teste planejado não equivale a uma implementação ou validação concluída.

## 4.4 Checklist de revisão do modelo

- [ ] Os componentes e ativos relevantes foram identificados?
- [ ] Os fluxos de dados entre navegador, back-end, banco, armazenamento e gateway estão representados?
- [ ] As fronteiras de confiança estão explícitas?
- [ ] As permissões de cliente e administrador foram consideradas separadamente?
- [ ] As seis categorias STRIDE foram avaliadas?
- [ ] Os cenários descrevem ativos, vulnerabilidade, exploração e impacto?
- [ ] Cada ameaça possui uma estratégia de tratamento?
- [ ] Os controles estão associados a testes verificáveis?
- [ ] Hipóteses e lacunas foram registradas?
- [ ] Os riscos residuais e as decisões de aceitação foram documentados?
- [ ] O modelo será atualizado quando arquitetura, APIs, dados ou integrações mudarem?

## 4.5 Limitações e próximos passos

Este exercício foi desenvolvido a partir de um cenário fictício e de informações limitadas. Para evoluir o trabalho:

1. Criar ou anexar uma versão gráfica do DFD com processos, entidades externas, depósitos de dados, fluxos e fronteiras.
2. Detalhar endpoints, métodos HTTP, dados trafegados e mecanismos de autorização.
3. Confirmar como JWT, sessões e webhooks são implementados.
4. Definir uma matriz de probabilidade e impacto para priorização dos riscos.
5. Transformar controles em requisitos de segurança e tarefas de desenvolvimento.
6. Implementar um pequeno laboratório e executar os testes propostos.
7. Registrar evidências, resultados, correções e riscos residuais.
8. Revisar o modelo sempre que houver mudanças relevantes no sistema.

## 4.6 Conclusão

A modelagem permitiu estruturar a compreensão da arquitetura ShopFlow, identificar cenários de ameaça nas seis categorias STRIDE e propor medidas de tratamento e validação. O resultado é um modelo inicial, sujeito a refinamento conforme novas informações sejam obtidas e os controles sejam implementados e testados.
