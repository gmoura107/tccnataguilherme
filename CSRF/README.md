# Sites didáticos de CSRF (Cross-Site Request Forgery)

Este projeto é um ambiente simulado para estudo e demonstração de ataques de **CSRF (Cross-Site Request Forgery)** e da contramedida com **Token Anti-CSRF (sincronizador)**.

O objetivo é visualizar como um site malicioso consegue forçar o navegador de uma vítima autenticada a executar ações (como uma transferência bancária) sem o seu consentimento — e como o uso de um token de validação impede esse ataque.

## Por onde começar

Abra o **`index.html`** — é a página central (hub) que reúne links para todas as outras e traz o passo a passo dos cenários. Cada link abre em uma nova aba, já que a simulação precisa de várias páginas abertas ao mesmo tempo (servidor + cliente + site do hacker).

## Arquivos do Projeto

| Arquivo | Papel |
|---------|-------|
| **`index.html`** | **Página inicial (hub)** com os links e instruções de todos os cenários. |
| **`1-cliente-vulneravel.html`** | Banco da vítima **sem** proteção anti-CSRF (login, Face ID e transferência). |
| **`2-servidor-vulneravel.html`** | API/servidor que aprova as transferências **sem** validar token. |
| **`3-cliente-protegido.html`** | Banco da vítima **com** Token Anti-CSRF (entregue via SMS simulado para fins didáticos). |
| **`4-servidor-protegido.html`** | API/servidor que **valida o token** antes de aprovar a transferência. |
| **`5-site-hacker.html`** | Site malicioso que dispara a requisição forjada contra o banco da vítima. |

---

## Importante: Simulação vs. Realidade

Para que o laboratório rode localmente sem precisar de um servidor Backend (Node, PHP, Java) ou Banco de Dados real, a comunicação entre as abas (cliente, servidor e hacker) é simulada usando o **`localStorage`** do navegador.

* **Neste Laboratório:** A "requisição HTTP" da vítima para o servidor é representada por uma chave no `localStorage`, que a aba do servidor fica "escutando". A sessão autenticada (cookie) e o saldo da conta também são simulados dessa forma.
* **Na Vida Real:** O ataque CSRF explora o fato de o navegador **enviar os cookies de sessão automaticamente** em toda requisição para o domínio do banco. O site do hacker submete um formulário (ou faz um `fetch`) para a API do banco, e o servidor — não conseguindo distinguir uma ação legítima de uma forjada — executa a transferência.

---

## Como executar a simulação

Abra os arquivos em abas/janelas do mesmo navegador (mesma origem, para compartilharem o `localStorage`).

### Cenário 1 — Ataque bem-sucedido (sem proteção)
1. Abra **`2-servidor-vulneravel.html`** (painel de logs do banco).
2. Abra **`1-cliente-vulneravel.html`** e faça login (CPF `123.456.789-00`, senha `123123`) + Face ID. Isso cria a sessão ativa.
3. Abra **`5-site-hacker.html`** e dispare o ataque.
4. Resultado: o servidor **aprova a transferência forjada**, pois não exige nenhum token. O saldo da vítima cai e a transação aparece no **Extrato** marcada como origem suspeita.

### Cenário 2 — Ataque bloqueado (com proteção)
1. Abra **`4-servidor-protegido.html`** (gera um token único da sessão).
2. Abra **`3-cliente-protegido.html`**, faça login e, ao transferir, use o botão **Obter SMS** para receber o token válido.
3. Abra **`5-site-hacker.html`** e tente o ataque.
4. Resultado: o servidor **recusa a transferência**, pois o site do hacker **não conhece o token** anti-CSRF da sessão da vítima.

---

## Contramedida utilizada: Token Anti-CSRF (Synchronizer Token)

No fluxo protegido, o servidor gera um **token aleatório e único por sessão** que só o usuário legítimo consegue obter (aqui, entregue via "SMS"). Toda transferência precisa enviar esse token, e o servidor só aprova se ele **bater exatamente** com o token esperado:

```javascript
// Lado do servidor (4-servidor-protegido.html)
const tokenServidor = 'CSRF-' + Math.random().toString(36).substr(2, 9).toUpperCase();

// Valida antes de mover o dinheiro
if (data.token === tokenServidor) {
    // ✔ Ação autêntica: aprova e debita o saldo
} else {
    // ❌ Token inválido/ausente: bloqueia (provável CSRF)
}
```

Como o site do hacker está em outra origem e **não tem acesso a esse token**, a requisição forjada é barrada.
