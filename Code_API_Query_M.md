
### 📌 Etapas:
1. Crie uma nova consulta → **Consulta nula**
2. Vá no **Editor Avançado**
3. Cole o código abaixo:
   
```
//Primeiro arquivo a ser criado no Query M é o arquivo da função que vai receber a url precisa ser com esse nome 'character' 

(url)=>
let
    Fonte = Json.Document(Web.Contents(url)),
    #"Convertido para Tabela" = Table.FromRecords({Fonte}),
    #"info Expandido" = Table.ExpandRecordColumn(#"Convertido para Tabela", "info", {"count", "pages", "next", "prev"}, {"info.count", "info.pages", "info.next", "info.prev"}),
    #"results Expandido" = Table.ExpandListColumn(#"info Expandido", "results"),
    #"results Expandido1" = Table.ExpandRecordColumn(#"results Expandido", "results", {"id", "name", "status", "species", "type", "gender", "origin", "location", "image", "episode", "url", "created"}, {"results.id", "results.name", "results.status", "results.species", "results.type", "results.gender", "results.origin", "results.location", "results.image", "results.episode", "results.url", "results.created"}),
    #"results.origin Expandido" = Table.ExpandRecordColumn(#"results Expandido1", "results.origin", {"name", "url"}, {"results.origin.name", "results.origin.url"}),
    #"results.location Expandido" = Table.ExpandRecordColumn(#"results.origin Expandido", "results.location", {"name", "url"}, {"results.location.name", "results.location.url"}),
    #"results.episode Expandido" = Table.ExpandListColumn(#"results.location Expandido", "results.episode"),
    #"Tipo Alterado" = Table.TransformColumnTypes(#"results.episode Expandido",{{"info.count", Int64.Type}, {"info.pages", Int64.Type}, {"info.next", type text}, {"info.prev", type any}, {"results.id", Int64.Type}, {"results.name", type text}, {"results.status", type text}, {"results.species", type text}, {"results.type", type any}, {"results.gender", type text}, {"results.origin.name", type text}, {"results.origin.url", type text}, {"results.location.name", type text}, {"results.location.url", type text}, {"results.image", type text}, {"results.episode", type text}, {"results.url", type text}, {"results.created", type datetime}})
in
    #"Tipo Alterado"
```

![image](https://github.com/user-attachments/assets/763bb602-ca2f-4305-ac4a-cd239fa5e924)


//Feito isso você precisa criar o segundo arquivo que recebe as chamadas da API com esse nome 'All_characters'

## 🔄 Consumindo API com Power Query (M)

Este trecho mostra como consumir todos os dados da [Rick and Morty API](https://rickandmortyapi.com/) usando Power Query (linguagem M), iterando por todas as páginas da API e expandindo os resultados.

### 📌 Etapas:
1. Crie uma nova consulta → **Consulta nula**
2. Vá no **Editor Avançado**
3. Cole o código abaixo:

```

let
    // Chamada inicial à API para pegar apenas o info.pages
    FonteInicial = Json.Document(Web.Contents("https://rickandmortyapi.com/api/character")),
    Paginas = FonteInicial[info][pages],
    ListaPaginas = List.Numbers(1, Paginas),
    
    // Transformar lista em tabela
    TabelaPaginas = Table.FromList(ListaPaginas, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    TipoCorrigido = Table.TransformColumnTypes(TabelaPaginas,{{"Column1", type text}}),
    
    // Construir os links
    ComLinks = Table.AddColumn(TipoCorrigido, "Link", each "https://rickandmortyapi.com/api/character/?page=" & [Column1]),
    
    // Chamar a função personalizada
    ComDados = Table.AddColumn(ComLinks, "character", each character([Link])),
    
    // Expandir os dados
    Expandido = Table.ExpandTableColumn(ComDados, "character", {"info.count", "info.pages", "info.next", "info.prev", "results.id", "results.name", "results.status", "results.species", "results.type", "results.gender", "results.origin.name", "results.origin.url", "results.location.name", "results.location.url", "results.image", "results.episode", "results.url", "results.created"}, {"character.info.count", "character.info.pages", "character.info.next", "character.info.prev", "character.results.id", "character.results.name", "character.results.status", "character.results.species", "character.results.type", "character.results.gender", "character.results.origin.name", "character.results.origin.url", "character.results.location.name", "character.results.location.url", "character.results.image", "character.results.episode", "character.results.url", "character.results.created"})
in
    Expandido
```
    
![image](https://github.com/user-attachments/assets/e5cede0e-095f-4d89-b9fd-de21cc983bd8)

Dados retornados de forma dinâmica da API:
![image](https://github.com/user-attachments/assets/88338c2d-482b-4921-94a8-3ead55211ee3)

Lembrando que você pode seguir esse passo a passo para qualquer conexão de API paginada.
