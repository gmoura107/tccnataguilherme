#  Sites didáticos de XSS (Cross-Site Scripting)

Este projeto consiste em um ambiente simulado para estudo e demonstração de vulnerabilidades de XSS (Cross-Site Scripting) e suas respectivas correções (Sanitização).
O objetivo é visualizar como scripts maliciosos podem ser injetados em aplicações web e como proteger o código contra esses ataques.

## Arquivos do Projeto
1. **`siteXSSvul.html`**: Página intencionalmente insegura contendo as 3 falhas de segurança ativas.
2. **`siteXSSprotegido.html`**: A mesma página, porém com uma camada de defesa (Sanitização) aplicada via JavaScript.

---
## Importante: Simulação vs. Realidade
Para fins didáticos e para permitir que este laboratório rode localmente sem a necessidade de configurar um servidor Backend (Node, PHP, Java) ou Banco de Dados (MySQL, MongoDB), utilizamos o **LocalStorage** do navegador.
* **Neste Laboratório:** A persistência dos dados (Salvar Comentário) é simulada usando o `localStorage`. Tecnicamente, como o dado nunca sai do navegador, isso classifica este ataque específico como *DOM-based com persistência local*.
* **Na Vida Real:** Um ataque de **XSS Persistente (Stored)** ocorre quando o script malicioso é enviado para o servidor e salvo em um **Banco de Dados real**. O servidor então entrega esse script para **todos os usuários** que acessarem a página, tornando o ataque muito mais perigoso e abrangente.
---

## Cenários de Teste (Vulnerabilidades)
O laboratório cobre os três tipos principais de XSS:

### 1. XSS Refletido 
* **Como funciona:** O ataque é entregue via URL. O site lê os parâmetros do link e os exibe na tela sem limpeza.
* **Adicione ao final da URL:** Adicione ao final da URL:
    ```
    ?msg=<img src=x onerror=alert('ATAQUE_XSS_REFLETIDO')>
    ```

### 2. XSS Persistente 
* **Como funciona:** O script malicioso é "salvo" no armazenamento. Mesmo recarregando a página ou fechando o navegador, o script executa automaticamente ao abrir o site.
* **Faça o seguinte comentário:**
    ```html
    <img src=x onerror=alert('ATAQUE_XSS_PERSISTENTE')>
    ```

### 3. XSS Baseado em DOM 
* **Como funciona:** A falha ocorre inteiramente no processamento do JavaScript no lado do cliente. O dado entra por um campo de input e é inserido no HTML via `innerHTML`.
* **Coloque o seguinte código no campo de busca:**
    ```html
    <img src=x onerror=alert('ATAQUE_XSS_DOM')>
    ```

---

## Contramedida utilizada: Sanitização
No arquivo `siteXSSprotegido.html`, implementamos uma função de sanitização que atua como **Contramedida**.

```javascript
function sanitizar(texto) {
    if (!texto) return "";
    return texto
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;") 
        .replace(/>/g, "&gt;") 
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
}
