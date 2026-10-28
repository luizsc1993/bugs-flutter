# Laudo técnico — Central de Achados e Perdidos

> Observação: o projeto foi corrigido por inspeção estática. Os testes abaixo são o roteiro de validação manual no Flutter/Firebase; precisam ser executados no ambiente com Firebase configurado para confirmar o comportamento real.

## Bug 01
**Arquivo:** `lib/models/ocorrencia.dart`  
**Sintoma:** o identificador do responsável era carregado do nome, então a comparação de propriedade falhava.  
**Causa:** `autorId` lia o campo `autorNome` do Firestore.  
**Correção:** carregar `autorId` do campo `autorId`.  
**Teste:** cadastrar com a conta A e conferir que o UID salvo/carregado corresponde ao UID autenticado; o botão de status deve aparecer para A, não para B.

## Bug 02
**Arquivo:** `lib/models/ocorrencia.dart`  
**Sintoma:** ocorrências abertas apareciam como resolvidas e vice-versa.  
**Causa:** o valor booleano `resolvido` era invertido durante a leitura.  
**Correção:** ler o booleano armazenado diretamente, usando `false` como padrão.  
**Teste:** criar uma ocorrência e confirmar “Em aberto”; marcar como resolvida e confirmar “Resolvido”.

## Bug 03
**Arquivo:** `lib/pages/central_page.dart`  
**Sintoma:** o formulário aceitava cadastro quando apenas título ou local estava preenchido.  
**Causa:** a validação usava `&&`, exigindo que ambos estivessem vazios para bloquear.  
**Correção:** usar `||`, bloqueando quando qualquer campo obrigatório estiver vazio.  
**Teste:** tentar enviar sem título e depois sem local; ambos devem ser recusados. Preencher os dois deve permitir cadastro.

## Bug 04
**Arquivo:** `lib/pages/central_page.dart`  
**Sintoma:** o interruptor do filtro podia ficar com estado diferente do valor selecionado.  
**Causa:** o callback invertia `value` em vez de atribuí-lo.  
**Correção:** atribuir `somenteAbertos = value`.  
**Teste:** ligar e desligar o filtro repetidamente e verificar que o interruptor acompanha a lista.

## Bug 05
**Arquivo:** `lib/pages/central_page.dart`  
**Sintoma:** o filtro “Somente não resolvidos” mostrava ocorrências resolvidas.  
**Causa:** a condição selecionava `o.resolvido`.  
**Correção:** filtrar por `!o.resolvido`.  
**Teste:** manter uma ocorrência aberta e outra resolvida; com o filtro ligado, apenas a aberta deve aparecer.

## Bug 06
**Arquivo:** `lib/pages/central_page.dart`  
**Sintoma:** abrir uma ocorrência não construía corretamente a tela de detalhes.  
**Causa:** `DetalhePage` exige a ocorrência, mas a navegação não passava o parâmetro obrigatório.  
**Correção:** passar `ocorrencia: o` ao construir `DetalhePage`.  
**Teste:** tocar em duas ocorrências diferentes e confirmar que cada detalhe exibe os dados correspondentes.

## Bug 07
**Arquivo:** `lib/pages/detalhe_page.dart`  
**Sintoma:** o envio de pista era interrompido quando o campo continha texto.  
**Causa:** a condição de validação estava invertida.  
**Correção:** retornar apenas quando o texto estiver vazio.  
**Teste:** enviar uma pista preenchida e confirmar que ela é registrada; tentar enviar vazia e confirmar que nada é enviado.

## Bug 08
**Arquivo:** `lib/pages/detalhe_page.dart`  
**Sintoma:** as pistas eram enviadas sem o texto digitado.  
**Causa:** a chamada ao serviço passava uma string vazia.  
**Correção:** passar `pista.text` para `enviarPista`.  
**Teste:** enviar uma frase única e verificar que a mesma frase aparece na lista de pistas.

## Bug 09
**Arquivo:** `lib/services/central_service.dart`  
**Sintoma:** pistas não apareciam no detalhe da ocorrência correta.  
**Causa:** todas eram gravadas no documento fixo `ocorrencias/pistas`, em vez de usar o ID da ocorrência.  
**Correção:** gravar em `ocorrencias/{ocorrenciaId}/pistas`.  
**Teste:** criar duas ocorrências, enviar uma pista para cada e confirmar que cada pista aparece somente no detalhe correspondente.

## Bug 10
**Arquivo:** `lib/services/central_service.dart`  
**Sintoma:** a autorização de alteração de status não identificava corretamente o dono.  
**Causa:** o campo `autorId` armazenava e-mail, mas a autorização compara com o UID do Firebase Authentication.  
**Correção:** armazenar `u.uid` em `autorId`; o nome continua no campo `autorNome`.  
**Teste:** com a conta A, verificar que ela pode alterar seu item; com a conta B, verificar que o botão não aparece e que a operação é recusada pelo serviço.

## Bug 11
**Arquivo:** `lib/services/central_service.dart`  
**Sintoma:** clicar em “Marcar como resolvido” não alterava o estado salvo corretamente.  
**Causa:** a atualização gravava o mesmo valor atual de `item.resolvido`.  
**Correção:** gravar `!item.resolvido`, alternando entre resolvido e aberto.  
**Teste:** marcar uma ocorrência como resolvida e depois reabri-la; conferir o campo `resolvido` no Firestore e a interface.

## Bug 12
**Arquivo:** `lib/pages/central_page.dart`  
**Sintoma:** após registrar uma ocorrência, a navegação podia fechar a tela principal em vez de fechar apenas o formulário.  
**Causa:** o método `cadastrar` chamava `Navigator.pop` usando o contexto da página, não o contexto do diálogo.  
**Correção:** passar `dialogContext` para `cadastrar` e fechar o diálogo com esse contexto após o cadastro bem-sucedido.  
**Teste:** registrar uma ocorrência e confirmar que o formulário fecha, enquanto a Central de Achados permanece aberta e a nova ocorrência aparece na lista.

## Observação sobre a ordenação
A consulta mantém a ordenação crescente por `criadoEm` (mais antigas primeiro), conforme a implementação original. Validar com dois cadastros se essa é a ordem esperada pela especificação da turma; se a regra for mostrar as mais recentes primeiro, trocar para `descending: true`.

## Verificação integrada multiusuário
1. Entrar com duas contas Google em perfis/janelas separados.
2. Criar uma ocorrência na conta A e verificar que aparece na conta B sem atualizar a página.
3. Enviar uma pista pela conta B e verificar que ela aparece em tempo real no detalhe aberto pela conta A.
4. Confirmar que a conta A pode alterar o próprio status e que a conta B não pode alterar esse item.
5. Conferir no Firestore `autorId` (UID), `autorNome`, `resolvido`, `criadoEm` e a subcoleção de pistas da ocorrência correta.

**Limitação:** a validação multiusuário depende de credenciais e projeto Firebase válidos; não é possível comprová-la apenas pela inspeção dos arquivos.
