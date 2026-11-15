# ShopFlow — Modelagem de Ameaças

Projeto prático de **Modelagem de Ameaças (Threat Modeling)** aplicado a uma plataforma fictícia de e-commerce chamada ShopFlow. O trabalho utiliza o modelo de quatro perguntas de Adam Shostack e a metodologia **STRIDE** para estruturar a arquitetura, identificar ameaças e propor tratamentos e validações.

> **Escopo:** exercício educacional baseado em um cenário fictício. Não representa uma avaliação de segurança de um sistema real, nem comprova que os controles propostos foram implementados ou testados.

## Objetivos

- Representar os principais componentes, fluxos de dados e fronteiras de confiança.
- Identificar ameaças às propriedades de confidencialidade, integridade e disponibilidade.
- Descrever cenários plausíveis de exploração e seus impactos.
- Definir estratégias de tratamento e controles técnicos.
- Propor testes que permitam verificar se os controles funcionam.

## Cenário

A ShopFlow é uma startup fictícia que está desenvolvendo uma aplicação web para venda de produtos. Clientes podem criar contas, consultar produtos, gerenciar carrinho, finalizar pedidos, pagar por um gateway externo e acompanhar compras. Administradores possuem funções adicionais, como consultar pedidos de diferentes clientes e alterar informações administrativas.

### Arquitetura considerada

| Componente | Tecnologia / função |
|---|---|
| Front-end | React no navegador |
| Back-end | Python com Flask |
| Banco de dados | PostgreSQL |
| Hospedagem | AWS |
| Autenticação | Login e senha + JWT |
| Pagamentos | API de gateway externo |
| Repositório | GitHub |
| Comunicação | HTTPS / REST API |
| Arquivos de imagem | Amazon S3 (considerado no levantamento) |
| Logs | Registros de operações da aplicação |

Alguns detalhes não foram especificados no enunciado. Quando necessário, foram tratados como **hipóteses de modelagem**, e não como fatos confirmados.

## Método

O trabalho foi organizado pelas quatro perguntas:

1. **O que estamos construindo?** — entendimento da arquitetura, dos ativos, dos fluxos e das fronteiras de confiança.
2. **O que pode dar errado?** — identificação de ameaças usando STRIDE.
3. **O que vamos fazer a respeito?** — escolha de estratégias de tratamento, controles e validações.
4. **Fizemos um bom trabalho?** — revisão de cobertura, lacunas e validação contínua do modelo.

## Estrutura do repositório

```text
shopflow-threat-model/
├── README.md
└── docs/
    ├── 01-cenario-arquitetura-dfd.md
    ├── 02-modelo-de-ameacas-stride.md
    ├── 03-plano-de-tratamento.md
    ├── 04-plano-de-validacao-e-revisao.md
    └── images/
        └── dfd-shopflow.png
```

## Principais ameaças identificadas

| ID | Categoria STRIDE | Cenário resumido |
|---|---|---|
| TM-001 | Spoofing | Uso indevido de JWT administrativo roubado |
| TM-002 | Tampering | Adulteração do preço enviado pelo cliente no checkout |
| TM-003 | Repudiation | Administrador nega alteração de preço sem trilha de auditoria confiável |
| TM-004 | Information Disclosure | Cliente acessa pedido pertencente a outro usuário (IDOR) |
| TM-005 | Denial of Service | Volume abusivo de requisições afeta a disponibilidade |
| TM-006 | Elevation of Privilege | Cliente executa operação reservada a administrador |

Os IDs foram organizados em sequência para facilitar a leitura do repositório. Eles podem ser ajustados caso o modelo seja expandido.

## Estratégia de tratamento

Para os cenários documentados, a estratégia proposta é **mitigar**, por meio de controles como proteção de sessão, validação de dados no servidor, trilhas de auditoria protegidas, autorização por objeto, limitação de requisições e controle de privilégios.

O plano detalhado está em [`docs/03-plano-de-tratamento.md`](docs/03-plano-de-tratamento.md).

## Status e limitações

- [x] Cenário e arquitetura descritos.
- [x] Principais fluxos e fronteiras de confiança documentados.
- [x] Seis categorias STRIDE analisadas.
- [x] Estratégias e controles de tratamento propostos.
- [x] Plano de validação definido.
- [X] DFD gráfico final anexado ao repositório.
  

**Importante:** os testes descritos são propostas de validação. Não devem ser interpretados como testes já executados.

## Como utilizar

1. Leia o cenário e a arquitetura em `docs/01-cenario-arquitetura-dfd.md`.
2. Consulte os cenários de ameaça em `docs/02-modelo-de-ameacas-stride.md`.
3. Veja os controles propostos em `docs/03-plano-de-tratamento.md`.
4. Utilize `docs/04-plano-de-validacao-e-revisao.md` como roteiro para uma futura validação em laboratório.

## Referências conceituais

- STRIDE — Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service e Elevation of Privilege.
- Adam Shostack — abordagem de modelagem de ameaças baseada em quatro perguntas.
- OWASP — conceitos de Broken Access Control, IDOR e testes de segurança de aplicações.

---

**Autor:** Maicon Souza  
**Tipo:** Projeto de estudo / portfólio  
**Empresa e sistema analisados:** fictícios (ShopFlow)
