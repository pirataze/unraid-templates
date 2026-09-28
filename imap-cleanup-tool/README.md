# IMAP Cleanup Tool — Unraid Template

Template Docker para instalar o **IMAP Cleanup Tool** no Unraid. A aplicação organiza e limpa e-mails por IMAP através de uma interface web.

> Compatível com qualquer provedor que ofereça IMAP: Gmail, Outlook/Microsoft 365, Yahoo, iCloud, mailbox.org, Fastmail, Zoho, servidores Dovecot/Cyrus e outros.

## Recursos

- Interface web para consultar e organizar mensagens
- Regras por remetente, assunto, data, domínio e outros critérios
- Ações de mover mensagens entre pastas IMAP, arquivar e apagar
- Funciona com contas IMAP existentes; não hospeda nem copia o seu serviço de e-mail
- Dados e credenciais ficam no volume local do Unraid; use proteção adicional para a interface web

## Instalação pelo Unraid

1. Faça upload de `imap-cleanup-tool.xml` e deste `README.md` para um repositório GitHub.
2. Edite o elemento abaixo no XML, substituindo os valores de exemplo:

   ```xml
   <TemplateURL>https://raw.githubusercontent.com/SEU_USUARIO/SEU_REPO/main/imap-cleanup-tool.xml</TemplateURL>
   ```

3. No Unraid, abra **Settings → Community Applications → Template repositories**.
4. Adicione a URL raw do seu XML e guarde.
5. Abra **Apps**, procure `imap-cleanup-tool`, instale e preencha os campos de IMAP.

Também pode importar o XML localmente pela página **Docker → Add Container → Template**.

## Campos do template

| Campo | Obrigatório | Valor normal | Descrição |
|---|---:|---|---|
| `IMAP_SERVER` | Sim | — | Hostname do servidor IMAP, sem `https://` |
| `IMAP_PORT` | Sim | `993` | Porta IMAPS mais comum |
| `IMAP_SSL` | Sim | `true` | Ative para IMAPS/TLS direto; confirme os requisitos do provedor |
| `IMAP_USER` | Sim | — | Normalmente o endereço de e-mail completo |
| `IMAP_PASS` | Sim | — | App password, token ou senha específica da conta |
| `WEB_HOST` | Não | `0.0.0.0` | Endereço onde a UI escuta no container |
| `WEB_PORT` | Não | `8000` | Porta interna da UI |

O volume persistente é mapeado de `/mnt/user/appdata/imap-cleanup-tool` para `/data`. Não o elimine durante atualizações se quiser manter a configuração.

## Configurações por provedor

| Provedor | IMAP server | Porta | Segurança | Credencial recomendada |
|---|---|---:|---|---|
| Gmail / Google Workspace | `imap.gmail.com` | 993 | SSL/TLS | App password com 2FA, ou OAuth se suportado pela app |
| Outlook.com / Microsoft 365 | `outlook.office365.com` | 993 | SSL/TLS | OAuth ou app password, conforme a política da conta |
| Yahoo Mail | `imap.mail.yahoo.com` | 993 | SSL/TLS | App password |
| iCloud Mail | `imap.mail.me.com` | 993 | SSL/TLS | App-specific password Apple |
| mailbox.org | `imap.mailbox.org` | 993 | SSL/TLS | App password recomendado |
| Fastmail | `imap.fastmail.com` | 993 | SSL/TLS | App password recomendado |
| Zoho Mail | `imap.zoho.eu` ou o host regional | 993 | SSL/TLS | App password recomendado |
| Servidor próprio | Host do servidor | Conforme configuração | Conforme configuração | Conta IMAP dedicada, se possível |

Consulte sempre a documentação do provedor: hosts regionais, OAuth, portas e permissões podem variar. Para Proton Mail é necessário o **Proton Mail Bridge**; o Bridge deve estar acessível a partir da rede Docker, não apenas em `127.0.0.1` no computador anfitrião.

## Primeira regra: mover e-mails antigos

Para manter a caixa de entrada limpa, crie uma regra com:

```text
Origem: INBOX
Condição: data anterior a 7 dias
Ação: Move
Destino: Archive/Old Inbox
```

Execute primeiro em modo de pré-visualização, ou aplique a uma pequena seleção. Confirme o nome real da pasta de destino mostrado pelo servidor IMAP — separadores e nomes de pastas podem variar entre provedores.

## Segurança

Não coloque credenciais neste repositório GitHub. Os campos `IMAP_USER` e `IMAP_PASS` permanecem em branco no XML e devem ser preenchidos no formulário Docker do Unraid.

A UI pode não disponibilizar autenticação própria. Não a exponha diretamente à Internet. Prefira uma destas abordagens:

1. **Acesso apenas pela LAN** e sem encaminhamento de portas no router.
2. **Reverse proxy com HTTPS e autenticação**, por exemplo Nginx Proxy Manager, Authelia ou Authentik.
3. **Túnel SSH**, sem publicar a porta:

   ```bash
   ssh -L 8765:127.0.0.1:8000 utilizador@IP_DO_UNRAID
   ```

   Depois abra `http://127.0.0.1:8765` no navegador.

Use uma app password limitada/revogável, em vez da senha principal, sempre que o provedor oferecer essa opção.

## Diagnóstico

```bash
# Ver logs do container
docker logs imap-cleanup-tool

# Confirmar que o container está ativo
docker ps --filter name=imap-cleanup-tool
```

Se não conectar ao IMAP, confirme hostname, porta, TLS, acesso IMAP habilitado no provedor e o tipo de credencial. Se a UI não abrir, confirme que a porta de host escolhida não está ocupada e que o container está em execução.

## Atualizações

Antes de atualizar, faça backup de `/mnt/user/appdata/imap-cleanup-tool`. Depois atualize a imagem pelo painel Docker do Unraid e verifique os logs antes de executar regras automáticas.
