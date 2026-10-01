# MVP Calendário Acadêmico iCalendar

## Objetivo

Este MVP valida a assinatura de um calendário acadêmico remoto via feed iCalendar (`.ics`). A ideia é confirmar se Google Calendar, Outlook e Apple Calendar conseguem adicionar o calendário por URL, exibir eventos, receber alterações posteriores e lidar com múltiplos lembretes.

O arquivo principal é `syncenade-prototipo.ics`, que deve ser publicado em um GitHub Gist público e assinado pelos clientes de calendário pela URL Raw.

## Como publicar no GitHub Gist

1. Crie um Gist público em <https://gist.github.com/>.
2. Adicione o arquivo `syncenade-prototipo.ics`.
3. Salve o Gist.
4. Abra o arquivo no Gist e clique em `Raw`.
5. Copie a Raw URL.
6. Use essa URL como endereço do calendário assinado.

Use uma Raw URL que continue apontando para a versão atual do Gist. Evite URLs presas exclusivamente a uma revisão específica se isso impedir que alterações futuras sejam percebidas pelos calendários assinantes.

## Como automatizar a atualização do Gist

O repositório inclui a workflow `.github/workflows/sync-calendar-gist.yml`, que atualiza o arquivo `syncenade-prototipo.ics` no Gist sempre que houver push na branch `main` alterando esse arquivo.

Para ativar:

1. Crie um token em <https://github.com/settings/tokens> com permissão para editar Gists.
   - Fine-grained token: habilite permissão de Gists, se disponível para sua conta.
   - Classic token: marque o escopo `gist`.
2. No repositório, abra `Settings` > `Secrets and variables` > `Actions`.
3. Crie o secret `CALENDAR_GIST_TOKEN` com o token criado.
4. Crie o secret `CALENDAR_GIST_ID` com o ID do Gist.

O ID do Gist é a parte final da URL. Exemplo:

```text
https://gist.github.com/seu-usuario/abc123def456
                                ^^^^^^^^^^^^^
```

Depois disso, qualquer alteração em `syncenade-prototipo.ics` enviada para `main` será publicada automaticamente no Gist.

## Como testar

Importar arquivo ICS ≠ assinar calendário ICS.

Importar cria uma cópia estática dos eventos. Para este MVP, o teste precisa ser feito como assinatura por URL, pois queremos validar atualizações posteriores do feed.

### Google Calendar

1. Abra o Google Calendar no navegador.
2. Em `Outras agendas`, escolha adicionar calendário por URL.
3. Cole a Raw URL do `syncenade-prototipo.ics`.
4. Confirme se o calendário foi adicionado e se os 3 eventos aparecem.

### Outlook

1. Abra o Outlook Calendar.
2. Escolha adicionar calendário pela internet ou por assinatura da Web.
3. Cole a Raw URL do `syncenade-prototipo.ics`.
4. Confirme se o calendário foi adicionado e se os 3 eventos aparecem.

### Apple Calendar

1. Abra o Apple Calendar.
2. Use a opção de nova assinatura de calendário.
3. Cole a Raw URL do `syncenade-prototipo.ics`.
4. Confirme se o calendário foi adicionado e se os 3 eventos aparecem.

## Matriz de testes

| Teste | Google | Outlook | Apple |
|---|---|---|---|
| Calendário foi adicionado | ⬜ | ⬜ | ⬜ |
| 3 eventos apareceram | ⬜ | ⬜ | ⬜ |
| Timezone correto | ⬜ | ⬜ | ⬜ |
| Lembrete 7 dias | ⬜ | ⬜ | ⬜ |
| Lembrete 1 dia | ⬜ | ⬜ | ⬜ |
| Lembrete 2 horas | ⬜ | ⬜ | ⬜ |
| Alteração foi sincronizada | ⬜ | ⬜ | ⬜ |
| Evento removido desapareceu | ⬜ | ⬜ | ⬜ |

## Teste de atualização

Para validar sincronização, modifique o Evento 2 (`UID:inovaday-deadline-002@mvp.local`) mantendo o mesmo `UID`.

Altere a data de:

```ics
DTSTART;TZID=America/Sao_Paulo:20260813T200000
DTEND;TZID=America/Sao_Paulo:20260813T210000
```

para:

```ics
DTSTART;TZID=America/Sao_Paulo:20260814T200000
DTEND;TZID=America/Sao_Paulo:20260814T210000
```

Também altere:

```ics
SEQUENCE:0
```

para:

```ics
SEQUENCE:1
```

Atualize o `DTSTAMP`, salve novamente o arquivo no Gist e aguarde o provedor de calendário consultar o feed. A atualização não necessariamente será instantânea, pois Google Calendar, Outlook e Apple Calendar controlam seus próprios intervalos de atualização de calendários assinados.

Campos úteis para testar atualização:

- `DTSTART`
- `DTEND`
- `SUMMARY`
- `DESCRIPTION`
- `SEQUENCE`
- `DTSTAMP`

O `UID` nunca deve mudar ao editar um evento existente. Ele é a identidade permanente do evento para os clientes de calendário.

## Teste de remoção

Depois dos testes iniciais, remova um dos blocos `VEVENT` do arquivo `syncenade-prototipo.ics`, salve novamente no Gist e verifique se os calendários assinantes eventualmente removem o evento localmente.

## Limitações que queremos validar

- Não podemos assumir sincronização em tempo real.
- Diferentes provedores podem atualizar o feed em intervalos diferentes.
- `VALARM` faz parte do padrão iCalendar, mas alguns clientes podem ignorar ou alterar lembretes recebidos de calendários externos.
- Múltiplos `VALARM`s podem não se comportar igualmente em todos os clientes.
- O MVP usa um calendário público e não contém dados pessoais.
- Um produto real provavelmente precisaria de URLs privadas ou tokenizadas se houvesse informações individualizadas de alunos.
