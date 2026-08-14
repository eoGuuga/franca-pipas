# Notas do projeto

---

## PIPA NÃO VAI PRA RECORTE DE FUNDO

**Regra fechada em 14/08/2026. Não é "conferir com cuidado", é não
rodar.**

Pipa tem **haste, vareta e cabresto finos demais**. O recorte
automático trata isso como ruído e **sempre come alguma coisa**. O
resultado é produto parecendo quebrado, e isso é **alterar o
produto, não remover fundo** — o que a regra do projeto proíbe.

**O ganho de padronização não compensa.** Fundo bonito com pipa
quebrada é pior que foto de chão de madeira com pipa inteira.

Vale pra qualquer produto de pipa: Baruel, caçadeira, latão laminado,
latão de seda, corte e recorte, mucha, vazada, flechinha, peixinho.

### Como isso foi descoberto, em duas rodadas

Chegou ao ar recortado e teve que voltar:

| rodada | fotos revertidas |
|---|---|
| 1ª — haste cortada, achada pelo dono no site | 27 |
| 2ª — decisão de tirar pipa do recorte de vez | 15 |
| **total** | **42** |

**Por que a conferência não pegou:** aprovei olhando folha de contato
com miniatura de ~110 px. Nesse tamanho o corpo da pipa domina e a
vareta de 2 px não aparece. Quem pegou foi o dono, olhando o site em
tamanho real.

**Nem número pega:** perder uma vareta de 2 px não move área nem alfa
parcial na triagem. Só olho, e só em tamanho grande.

### Onde o recorte continua valendo

Carretilha, linha em cone, cola, engomador, base de corte — produto
de silhueta cheia, sem apêndice fino. Esses passaram e continuam
recortados.

As originais ficam intactas em `fotos-organizadas/produtos/`, que o
recorte nunca toca. Reverter é copiar de lá para `fotos/produtos/`.


Coisas medidas que não estão óbvias no código, e decisões que vão
voltar à tona depois.

---

## Limite da URL do WhatsApp (estrutural, sem contorno aplicado)

**O problema:** o pedido inteiro viaja dentro da URL do `wa.me`. O
site não manda o pedido pra lugar nenhum — ele monta um texto e
enfia no endereço. Quanto mais itens no carrinho, maior o endereço.

**Remedido em 13/08/2026**, com o catálogo real de 33 produtos,
depois que a quantidade saiu da mensagem e o marcador virou `- ` no
lugar de `• ` (o bullet gastava 9 bytes de escape na URL, o hífen
gasta 1):

| itens distintos | mensagem | URL do wa.me |
|---|---|---|
| 1 | 158 | 276 |
| 5 | 369 | 571 |
| 10 | 592 | 923 |
| 15 | 957 | 1.554 |
| **20** | ~1.240 | **~2.050** |
| 25 | 1.520 | 2.482 |

**Piorou, não melhorou — e o motivo importa.** Tirar a quantidade
economizou uns 10%, mas os nomes reais dos produtos são bem mais
longos que os inventados do primeiro teste ("Carretilha com catraca
destravada e linha Vera Cruz, 16 polegadas" contra "Carretilha
ProMax, 20 polegadas"). O nome longo comeu a economia e passou.

**O teto caiu de ~25 para ~20 itens.** É o número que vale, porque
foi medido no catálogo que está no ar.

Lição pra quando isso voltar: **a alavanca real é o tamanho do nome
do produto**, não a quantidade nem o marcador. Cada caractere no
nome multiplica pela quantidade de itens na lista. Esse é o ponto onde navegadores antigos e webviews (o
navegador embutido do próprio WhatsApp, por exemplo) começam a
cortar URL. Chrome e Safari modernos aguentam bem mais, mas **o
wa.me não documenta limite nenhum** — então não dá pra afirmar onde
quebra de verdade, só que acima disso é terreno sem garantia.

**Por que importa:** é exatamente o perfil do revendedor, que a loja
atende e que pede 30 itens de uma vez. Com 4 produtos no catálogo
nunca vai acontecer. Com atacado de verdade, vai.

**Não foi contornado.** Decisão consciente de 11/08/2026: registrar
e seguir. Quando o assunto voltar, as saídas conhecidas são:

- ~~Tirar a quantidade~~ — feito em 13/08/2026, por outro motivo:
  a vitrine não controla estoque, então o cliente pergunta se tem em
  vez de afirmar que quer. O ganho na URL foi efeito colateral.
- Encurtar a linha do item (tirar medida, abreviar nome)
- Limitar quantos itens distintos entram no pedido, com aviso claro
- Quebrar em mais de uma mensagem
- Parar de mandar o pedido pela URL — exigiria backend, o que muda
  a natureza do projeto e contraria o CLAUDE.md

O comentário equivalente está em `index.html`, em cima de
`mensagemDoPedido()`, pra quem mexer no código topar com ele.

---

## Teste de carga do catálogo (11/08/2026)

Rodado com `demo.html`, 100 produtos fictícios em 20 categorias.
O arquivo é **local, não publicado**, e está no `.gitignore`.

O que aguentou bem:

- Montar 100 cards: **3,5 ms**. Re-render ao filtrar: **0,1 ms**.
  Performance de JS não é gargalo, e paginação não se justifica
  por esse motivo.
- Arquivo de 60 KB com 100 produtos (contra 27 KB com 4).
- Carrinho com 15 itens: renderiza certo, botão de enviar continua
  alcançável na base fixa do modal.
- Zero estouro lateral em qualquer largura.

O que quebrou:

- **21 chips de filtro no celular viram 7 fileiras, 253 px, 28% da
  tela.** O primeiro produto vai pra y=713 px — mais de uma tela de
  rolagem antes de qualquer mercadoria.
- Página com 18.049 px de altura em 360 px (20 telas), sem busca,
  sem paginação e sem marco nenhum de orientação.
- 213 paradas de Tab, sem atalho pra pular pros produtos.

**Não medido:** tempo de carregamento com fotos de verdade. A demo
tem zero imagens. Com 100 produtos fotografados seriam 100 a 300
arquivos; as imagens já saem com `loading="lazy"`, mas a primeira
tela ainda puxa o que está visível. Qualquer número sobre isso hoje
seria chute.

---

## Decisões que dependem do tamanho do catálogo

Recusadas hoje **porque o catálogo é pequeno**, não porque são
ruins. Revisar quando ele crescer:

- **Fita horizontal de categorias** — recusada com 2 categorias,
  porque fita que não rola parece quebrada. Com 20, passa a ser a
  resposta certa (ou um `<select>` nativo, que resolve 20 opções em
  40 px de tela no celular).
- **Busca** — com 100 produtos vira a navegação principal. Com 4,
  não tem o que buscar.
- **Agrupar categorias em dois níveis** — depende do dono definir a
  taxonomia da loja dele, não é decisão técnica.
- **Banner promocional no topo** — recusado porque não há preço nem
  promoção pra anunciar. Reabrir quando houver.

---

## O que mais move o ponteiro, e não é código

A referência que o dono usa (alemaopipas.com.br) tem centenas de
produtos e **preço grande em todos**. A vitrine tem 4 produtos e
"Sob consulta" nos quatro. A sensação de "loja de verdade" vem daí,
não de banner nem de barra fixa. **Colocar preço nos produtos muda
mais do que qualquer ajuste de layout.**
