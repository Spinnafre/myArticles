# **O que é BLOB ?**
Blob significa *Binary Large Object*, ou seja, um objeto binário que é muito usado em aplicações web para trabalhar com representações binárias de sequência de dados. 

Geralmente quando usa Blob é mais para trabalhar com *Uploads/Downloads* de dados binários, ou até mesmo criar um arquivo no lado do cliente (browser) e exibir ele na página sem precisar de um servidor para enviar tal imagem ou arquivo.

É importante ressaltar que Blob é **imutável**, ou seja, não podemos alterar seus dados diretamente a não ser que crie um novo blob para misturar dados. Uma vez que é criado, ele irá ficar armazenado na memória do browser até for solicitado a sua remoção ou mesmo o documento for "desconstruído".


> *Blob objects are immutable" We can't change data directly in a `Blob`, but we can slice parts of a `Blob`, create new `Blob` objects from them, mix them into a new `Blob` and so on.*

No meu dia a dia, costumo usar o Blob como se fosse um arquivo binário que possui um tipo específico, assim como um arquivo normal, no qual posso processar, enviar, renderizar no lado do cliente (browser) sem ter que precisar de um servidor mandando para mim um arquivo que talvez o usuário quase não irá visualizar ou interagir. Se eu fosse ter que ficar guardando tal arquivo no servidor provavelmente iria ter mais um certo custo para manter. 😉

## BLOB como URL

Assim como nós temos ```file//``` para representar a url para um arquivo real em um filesystem real, temos também o ```blob//``` que é usado para referenciar o conteúdo de uma blob, podendo ser acessado em quase todos os lugares que usa URLs.

Então sempre que criamos uma URL do objeto blob iremos ver a seguinte estrutura:
```
blob:http://127.0.0.1:5500/cb83b604-ad8c-4759-8d04-37d0f69b6fc7
```

Para criar uma blob URL, basta usar o ```URL.createObjectURL```, que irá retornar um DOMString que contém de fato a string da url que irá representar o objeto.

> Entenda a **DOMString** como uma string UTF-16 que irá ser mapeada diretamente para um ```String``` que é usado no Javascript.
```js
   const linkTag = document.createElement("a")
   // Nome da extensão e arquivo que irá ser baixado.
   linkTag.download = "blob.txt"
   
   const myCustomBlob = new Blob(["Olá, Dev!"],{
    type:"text/plain"
   })

    const url = URL.createObjectURL(myCustomBlob)

    linkTag.href = url
    // Baixa automaticamente o arquivo 
    linkTag.click()

    //Limpar referência do blob na memória do browser, pois como blob
    //é criado no documento ele irá ser imutável e só irá ser limpado da memória quando o documento for desfeito ou quando for solicitado a sua remoção da memória.
    URL.revokeObjectURL(url)
```
> ⚠️ Sempre que você chama o ```createObjectURL``` irá ser criado uma nova URL do objeto blob.

Vamos visualizar mais um exemplo para ficar mais claro 😁. Abaixo criei um script simples em que irá exibir em uma página html dois links que ao serem clicados pelo o usuário irá baixar duas versões do mesmo conteúdo em formatos diferentes, sendo respectivamente .txt e .html.

```js
    const blobHtml = new Blob(
        [
            '<html><head><title>Hello text</title></head><body><h1 style="color: red">Hello JavaScript!</h1></body></html>',
        ],
        { type: "text/html" }
    );

    const blobLink = URL.createObjectURL(blobHtml);

    // Saída : blob:http://127.0.0.1:5500/cb83b604-ad8c-4759-8d04-37d0f69b6fc7
    console.log(blobLink); //Irá sempre ser uma URL diferente.

    const linkToDownloadTxtFile = document.createElement("a");
    linkToDownloadTxtFile.href = blobLink;
    linkToDownloadTxtFile.download = "myBlob.txt";
    linkToDownloadTxtFile.innerText = "Baixar arquivo em formato txt";

    const linkToDownloadHTMLFile = document.createElement("a");
    linkToDownloadHTMLFile.href = blobLink;
    linkToDownloadHTMLFile.download = "myBlob.html";
    linkToDownloadHTMLFile.innerText = "Baixar arquivo em formato html";

    const app = document.querySelector("#app");

    app.appendChild(linkToDownloadTxtFile);

    app.insertAdjacentHTML("beforeend", "</br>");
    app.insertAdjacentElement("beforeend", linkToDownloadHTMLFile);

```
É importante notar que a URL gerada só irá ser válida enquanto não for criada outro blob e a aplicação ainda estiver rodando. Percebi pois quando pegava a mesma URL e jogava em outra aba do *browser* iria exibir normalmente a informação, mas se por acaso eu abrir outro navegador e tentar abrir a url ou até mesmo dar um reload na página dava problemas na exibição.

## Blob para Stream
Se você já trabalha com processamento de dados sobre demanda, ou seja, a medida em que vai recebendo ou criando os dados já vai realizando conjunto de operações contínuas com finalidade de gerar uma saída, deve já ter ouvido falar de Streams.

No NodeJs existe as Streams Nativas ( Readable, Transform, Writable) que oferecem uma alternativa de processamento de dados em Stream, que é uma alternativa ao processamento de dados em Batch. Assim como também no browser foi criado as Web Stream que são uma forma de trabalhar com Streams só que de maneira Universal, em cross-plataforma, ou seja, podendo trabalhar facilmente com Streams no navegador.

No processo de ETL (Extraction, Tranform and Load) é bastante usado esses tipos de processamentos usando estruturas de dados que permitem o processamento contínuo de pedaços (chunks) de uma fonte maior, sendo uma forma mais inteligente pois evita ter que armazenar na memória grande quantidade de dados que provavelmente sua máquina não irá conseguir rodar e com isso irá dar uma séries de bloqueios.

Como o foco dessa discussão não é Stream, inclusive posso até criar um artigo sobre elas. Iremos ir direto ao ponto 😅.

**Mas Davi, O que isso tem relação com Blob?** 

Bom, a interface do Blob possui o método ```stream()``` no qual retorna uma ReadableStream (Fonte de dados) que irá ser usada depois em uma pipeline para processamento de cada chunks. 

Para simplificar a nossa vida, abaixo criei um exemplo simples em que uso uma Readable Stream no NodeJs para exibir os dados bufferizado em um formato de texto, a medida em que pego cada chunk já peço logo para Decodificar os dados binários para texto.

```js
    const blobHtml = new Blob(
        [
            '<html><head><title>Hello text</title></head><body><h1 style="color: red">Hello JavaScript!</h1></body></html>',
        ],
        { type: "text/html" }
    );

    const readable = blobHtml.stream();
    //Transforma cada chunk que é por padrão um view object Uint8Arra (Representação binária em 8 bits do ArrayString) para uma String Javascript
    const textDecoded = readable.pipeThrough(new TextDecoderStream());

    for await (let myChunk of textDecoded) {
        console.log("chunk: ", myChunk);
    }

    // Saída: chunk:  <html><head><title>Hello text</title></head><body><h1 style="color: red">Hello JavaScript!</h1></body></html>
```
Note que acima Utilizei o TextDecoderStream, basicamente o que ele faz é só uma conversão dos dados binários de cada chunk em uma String do Javascript, essa técnica é muito usada quando se trabalha com Buffers.

Fiz mais um exemplo para ficar mais claro, só que dessa vez sem usar o TextDecoderStream.
```js
    const blobHtml = new Blob(
    [
        '<html><head><title>Hello text</title></head><body><h1 style="color: red">Hello JavaScript!</h1></body></html>',
    ],
    { type: "text/html" }
    );

    const readable = blobHtml.stream();
    const stream = readable.getReader();


    while (true) {
        const { done, value } = await stream.read();

        if (done) {
            console.log("Terminei tudo meu chapa! :)");
            break;
        }

        console.log("CHUNK >>> ", value);
    }

```
Note que basicamente as streams usam as mesma lógicas das **Generics** do Javascript, no qual retorna um objeto que possui o atributo ```done``` (se já finalizou a iteração) e ```value``` que é de fato o valor lido.

Para usos mais específicos, como Blob trabalha com "dados binários com tipos", podemos transformar um blob também para um **ArrayBuffer** que é nada mais nada menos do que usado para trabalhar com dados binários na memória de forma crua e genérica, e como ele não permite trabalhar com manipulação aí que surgems os *TypedArrays* que são formas de representações desses dados binários para poder realizar manipulações, como por exemplo temos o Uint8Array ( 8-bit unsigned integer array), Uint16Array (16-bit unsigned integer array), Uint32Array (32-bit unsigned integer array), Float64Array  (64-bit floating point numbers) e entre outros.
> Deixar link para essa discussão acima
```js
    const bufferPromise = await blob.arrayBuffer();

    // Ou
    blob.arrayBuffer().then(buffer => {
        // Faz alguma coisa aqui
    });
```
## Blob para base64
Uma alternativa muito interessante quando se trabalha com Blob é a possibilidade de converter um arquivo binário para um formato de texto em que irá ser usado na web em arquivos que só suportam dados em formato de texto como HTML ou CSS. Assim, se por acaso quiser pegar alguma imagem e exibir ela em uma previsualização na página, isso será possível se haver uma alternativa de converter dados binários de um conteúdo em um formato de texto que irá representar determinado arquivo e poderá ser embutido em tags do HTML.

No caso do HTML, podemos usar Strings em **DataURL** para exibir determinado arquivo enviado pelo o usuário na página, ou até mesmo compartilhar essa url na rede para ficar salva no servidor ou banco de dados e ser utilizado em outras aplicações.

> Por mais que DataURL sejam extremamentes usada para trabalhar com arquivos de imagem, se por acaso for usado arquivos muitos grandes irá acarretar em problemas relacionados a desempenho e memória devido ao processo de codificação. Logo, seu uso é mais indicado para arquivos menores.

Olha só que legal, no exemplo abaixo fiz uma demostração que irá gerar uma url que representa o arquivo binário que criei usando o blob. Com essa DataURL, posso facilmente jogar ela em qualquer navegador no meu computador e ele irá ler o arquivo em formato de texto.
```js
    let myBlob = new Blob(["Hello, world!"], { type: "text/plain" });

    const reader = new FileReader();

    // Quando terminar irá pegar os dados em formato DataURL acessando reader.result ou this.result.
    reader.onload = function () {
        console.log("DataURL: ", this.result);
    };

    //Inicia o processo de transformar dados binários em string DataURL
    reader.readAsDataURL(myBlob);

    // Saída: data:text/plain;base64,SGVsbG8sIHdvcmxkIQ==

```
O legal é que consigo também trabalhar com imagens para criar uma representação dela em formato de string e assim eu conseguir compartilhar na web ou renderizar no Frontend. 

Há também outras possibilidades de utilização do Blob, como o uso dele para trabalhar junto com o Canvas para transformar dados dele em um formato que poderá ser facilmente  enviado para algum lugar. Recomendo visitar o site [javascript.info](https://javascript.info/blob) pois há muitos tutoriais e explicações sobre o mundo do Javascript de forma bem detalhas e maravilhosa😉.

# **File Interface e FileReader** 
Quando começamos a trabalhar com arquivos no navegador, como por exemplo uploads de arquivos, temos alguns desafios que é pegar um dado que está no filesystem do sistema operacional e exibir ele no navegador para um propósito de pré-visualização.

Como já foi exibido no exemplo anterior, foi realizado a leitura de um Blob e transformado em um DataURL que poderá ser usado para para visualização ou envio de informações. Mas da mesma forma que consigo transformar dados do blob, também é possível codificar dados de um arquivo para base64. Isso é possível devido ao fato de objetos ```File``` herdarem do ```Blob```, possiblitando a utilização dos mesmos métodos e propriedades.

Assim, se o usuário passar um arquivo via input do tipo file no html, fica muito fácil pegar o arquivo e realizar alguma operação de leitura nele através do ```FileReader```, seja em formato binário, textual ou até para DataURL.

No exemplo abaixo, fiz uma pequena demonstração desse cenário de envio de arquivo e leitura para um formato base64.

```js
    const fileInput = document.getElementById("photo-1");

    fileInput.addEventListener("change", (element) => {
        const fileReader = new FileReader();
        //Como só quero pegar um único arquivo enviado, então acesso somente ao índice 0 do array.
        const image = fileInput.files[0];

        fileReader.addEventListener("load", (el) => {
            console.log(fileReader.result);
        });

        fileReader.readAsDataURL(image);
    });

    // Saída : data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAB4AAAAQ4CAYAAADoT...

```
> ```Blob``` e ```File``` ambos possuem o ```.stream()``` que retorna uma readable stream e assim fazendo com que você consiga usar pipelines para fazer o processamento sobre demanda de arquivos ou dados a medida em que recebe **chunks**.
```js
const fileInput = document.getElementById("photo-1");

const customWritableStream = new WritableStream({
  write(chunk) {
    console.log("Chunk >>> ", chunk);
  },
});

fileInput.addEventListener("change", (element) => {
  const readable = fileInput.files[0].stream();

  const textStream = readable.pipeThrough(new TextDecoderStream());

  textStream.pipeTo(customWritableStream);
});
```


E isso foi tudo pessoal, espero que tenha ficado claro, sinceramente acho esses conceitos muitos interessantes para estudar principalmente quando você começa a trabalhar com processamento de dados binários e tem que exibí-los em algum local na página ou até deixar disponível para baixar.

## Referências básicas:
- [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API)
- [javascript.](https://javascript.info)


Feito com ❤️ por Davi Silva ou conhecido como [Spinnafre](https://github.com/Spinnafre) 😉

