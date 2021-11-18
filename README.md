# 1. Visão Geral
Apache Tika é um kit de ferramentas para extrair conteúdo e metadados de vários tipos de documentos, como Word, Excel e PDF ou até mesmo arquivos de multimídia como JPEG e MP4.

Todos os arquivos de texto e multimídia podem ser analisados usando uma interface comum, tornando Tika uma biblioteca poderosa e versátil para análise de conteúdo.

Neste artigo, daremos uma introdução ao Apache Tika, incluindo sua API de análise e como ele detecta automaticamente o tipo de conteúdo de um documento. Exemplos de trabalho também serão fornecidos para ilustrar as operações desta biblioteca.

# 2. Primeiros passos
Para analisar documentos usando o Apache Tika, precisamos apenas de uma dependência Maven:

```
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-parsers</artifactId>
    <version>1.17</version>
</dependency>
```

A versão mais recente deste artefato pode ser encontrada aqui.

# 3. A API do Parser
A API do Parser é o coração do Apache Tika, abstraindo a complexidade das operações de análise. Esta API depende de um único método:

```
void parse(
  InputStream stream, 
  ContentHandler handler, 
  Metadata metadata, 
  ParseContext context) 
  throws IOException, SAXException, TikaException
```

Os significados dos parâmetros deste método são:

- stream - uma instância de InputStream criada a partir do documento a ser analisado;
- handler - um objeto ContentHandler recebendo uma sequência de eventos XHTML SAX analisados ​​a partir do documento de entrada; este manipulador irá então processar eventos e exportar o resultado em um formulário específico;
- metadados - um objeto de metadados que transmite propriedades de metadados dentro e fora do analisador;
- context - uma instância do ParseContext que carrega informações específicas do contexto, usada para personalizar o processo de análise.

O método de análise lança uma IOException se falhar ao ler do fluxo de entrada, uma TikaException se o documento obtido do fluxo não puder ser analisado e uma SAXException se o manipulador não puder processar um evento.

Ao analisar um documento, Tika tenta reutilizar bibliotecas de analisador existentes, como Apache POI ou PDFBox, tanto quanto possível. Como resultado, a maioria das classes de implementação do Parser são apenas adaptadores para essas bibliotecas externas.

Na seção 5, veremos como os parâmetros de manipulador e metadados podem ser usados ​​para extrair conteúdo e metadados de um documento.

Por conveniência, podemos usar a classe de fachada Tika para acessar a funcionalidade da API do Parser.

4. Detecção automática
O Apache Tika pode detectar automaticamente o tipo de um documento e seu idioma com base no próprio documento, em vez de em informações adicionais.

### 4.1. Detecção de Tipo de Documento
A detecção dos tipos de documentos pode ser feita por meio de uma classe de implementação da interface Detector, que possui um único método:

```
MediaType detect(java.io.InputStream input, Metadata metadata) 
  throws IOException
```

Este método pega um documento e seus metadados associados - em seguida, retorna um objeto MediaType que descreve a melhor estimativa em relação ao tipo do documento.

Os metadados não são a única fonte de informação da qual um detector depende. O detector também pode fazer uso de bytes mágicos, que são um padrão especial próximo ao início de um arquivo, ou delegar o processo de detecção a um detector mais adequado.

Na verdade, o algoritmo usado pelo detector depende da implementação.

Por exemplo, o detector padrão trabalha primeiro com bytes mágicos, depois com propriedades de metadados. Se o tipo de conteúdo não tiver sido encontrado neste ponto, ele usará o carregador de serviço para descobrir todos os detectores disponíveis e testá-los sucessivamente.

### 4.2. Detecção de idioma
Além do tipo de documento, o Tika também pode identificar seu idioma, mesmo sem ajuda de informações de metadados.

Em versões anteriores do Tika, o idioma do documento é detectado usando uma instância LanguageIdentifier.

No entanto, LanguageIdentifier foi preterido em favor dos serviços da web, o que não está claro nos documentos de Introdução.

Os serviços de detecção de idioma agora são fornecidos por meio de subtipos da classe abstrata LanguageDetector. Usando os serviços da web, você também pode acessar serviços de tradução online completos, como o Google Translate ou o Microsoft Translator.

Por uma questão de brevidade, não examinaremos esses serviços em detalhes.

# 5. Tika em ação
Esta seção ilustra os recursos do Apache Tika usando exemplos de trabalho.

Os métodos de ilustração serão agrupados em uma classe:

```
public class TikaAnalysis {
    // illustration methods
}
```

### 5.1. Detectando Tipos de Documento
Este é o código que podemos usar para detectar o tipo de documento lido de um InputStream:

```
public static String detectDocTypeUsingDetector(InputStream stream) 
  throws IOException {
    Detector detector = new DefaultDetector();
    Metadata metadata = new Metadata();

    MediaType mediaType = detector.detect(stream, metadata);
    return mediaType.toString();
}
```

Suponha que temos um arquivo PDF denominado tika.txt no classpath. A extensão deste arquivo foi alterada para tentar enganar nossa ferramenta de análise. O tipo real do documento ainda pode ser encontrado e confirmado por um teste: