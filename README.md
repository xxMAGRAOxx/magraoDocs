# Sobre a Api
Nós somos a Bibliomundi, uma distribuidora de livros digitais e disponibilizamos esta Api com o objetivo de facilitar a integração dos ebooks cadastrados em nossa plataforma com a sua loja. Para que você possa vender nossos ebooks em sua loja serão necessários alguns passos e o auxílio de um programador.

Obs. Esta api requer o PHP como linguagem de programação e caso você trabalhe com outras linguagens, disponibilizamos um webservice completo que você pode verificar <a href="https://drive.google.com/file/d/0BzwFNhJ9FBNwS0JCSzA3cXFPYUk/view">aqui</a>.


# Requerimentos
- <a href="http://php.net/manual/en/book.curl.php" target="blank">cURL</a>
- <a href="http://php.net/" target="blank">PHP 5.5</a>
- Conhecer o padrão <a href="http://www.editeur.org/83/Overview/" target="blank">Onix</a>
- Chave de identificação e senha. Se ainda não possuir, solicite-as através do e-mail contato@bibliomundi.com.br. Responderemos em instantes.

# Fluxo
1. Importar o nosso catálogo completo. Retornaremos para você um XML no padrão Onix com todos os ebooks cadastrados em nossa plataforma.

2. Inserir os ebooks em sua base de dados seguindo as normas do padrão Onix.

3. Realizar atualizações diárias para checar se existem novos ebooks, se existem ebooks a serem atualizados(Mudança de nome, por exemplo) ou se existem ebooks que requerem que você delete de sua base(Por não estarem mais sendo distribuídos por nós, por exemplo). Retornaremos para você um XML no padrão Onix com os ebooks que precisam ser atualizados, inseridos ou deletados.

4. Disponibilizar os ebooks para venda em sua loja.

5. Liberar o download do ebook para o cliente. 
 
# Instalação

Basta apenas fazer o download(Ou clonar), incluir em seu projeto e em seguida fazer uma chamada ao arquivo “autoload.php”, que se encontra dentro da pasta "lib". Pronto, você já tem acesso a todas as funcionalidades da Api.

<pre>Ex: require 'lib/autoload.php';</pre>

# Passo 1 - Importando os ebooks para a sua loja
Intancie a classe Catalog passando suas credencias como parâmetro.
<pre>$catalog = new BBM\Catalog('YOUR_API_KEY', 'YOUR_API_SECRET');</pre>
Defina se o ambiente que irá importar, os nossos ebooks, é o de produção ou de testes.
<pre>
$catalog->environment = 'production';
ou
$catalog->environment = 'sandbox'; 
</pre>

O trecho de código a seguir valida suas credenciais e importa os ebooks.
<pre>
try
{
    $catalog->validate();//Valida suas credenciais
    $xml = $catalog->get();//Retorna um <a href="https://github.com/xxMAGRAOxx/magraoDocs/blob/master/onix_example.xml" target="blank">XML</a>, no formato string e no padrão Onix, contendo todos os ebooks cadastrados em nossa plataforma
}
catch(\BBM\Server\Exception $e)
{
    var_dump($e);
}
</pre>

Cada tag &lt;Produto&gt; retornada pelo <a href="https://github.com/xxMAGRAOxx/magraoDocs/blob/master/onix_example.xml" target="blank">XML</a> é um ebook. Você irá percorrer todas elas e, seguindo normas do padrão Onix, inserindo em sua base de dados.

# Passo 2 - Inserindo os ebooks em sua loja
Uma vez com o <a href="https://github.com/xxMAGRAOxx/magraoDocs/blob/master/onix_example.xml" target="blank">XML</a> dos nossos ebooks, você pode trabalhar da maneira que achar melhor, mas recomendamos fortemente que utilize um parser, como o SimpleXML do php, por exemplo. Será de sua responsabilidade inserir os ebooks com as informações mínimas necessárias em sua loja. Recomendamos também que não insira ebooks que não estão disponíveis para venda, no momento da importação, e para isso você deverá realizar uma checagem através das tags &lt;PublishingStatus&gt; e &lt;ProductAvailability&gt;. Clicando <a target="blank" href="https://github.com/xxMAGRAOxx/magraoDocs/blob/master/onix_example.xml">aqui</a> você pode ver um exemplo do XML, que retornaremos para você, no padrão Onix e com as informações que consideramos essenciais.

# Passo 3 - Realizando atualizações diárias
Realizamos atualizações diárias em nosso sistema e você precisará, também diariamente, criar uma rotina para checar se existem ebooks a serem inseridos, atualizados ou deletados.
Recomendamos que crie uma agendador de tarefas para rodar entre 01 e 06 da manhã(UTC-3) afim de evitar que ebooks sejam disponibilizados com dados defasados podendo assim causar erros na venda.

Passe apenas um terceiro parâmetro chamado 'updates'.

<pre>$catalog = new BBM\Catalog('YOUR_API_KEY', 'YOUR_API_SECRET', 'updates');</pre>

Não muda nada na forma de fazer a requisição.

<pre>
$catalog->environment = 'production';
ou
$catalog->environment = 'sandbox';

try
{
    $catalog->validate();//Valida suas credenciais
    $xml = $catalog->get();//Retorna um XML com os ebooks e suas respectivas operações a serem relizadas(insert, update ou delete) no formato string e no padrão Onix
}
catch(\BBM\Server\Exception $e)
{
    var_dump($e);
}
</pre>

Para cada tag &lt;produto&gt; existirá uma tag chamada &lt;NotificationType&gt; indicando a operação a ser realizada.

Ex: 03 -> inserir. 04 -> Atualizar. 05 -> Deletar.

# Passo 4 - Realizando uma venda
Uma vez que você disponibilizar os ebooks em sua loja, seus clientes estarão aptos a relizar compras. Toda vez que um cliente tentar comprar um produto nosso será necessário que você valide a transação conosco e em caso de aprovação prosseguir para o checkout. Repare que a sua validação e o seu checkout e a nossa validação e o nosso checkout sãos duas coisas distintas. Você deverá sempre validar e fazer o checkout conosco para que tenhamos ciência de que a venda foi efetuada, para então podermos liberar o download para o seu cliente. Tenha isso em mente.

Instancie a classe Purchase passando suas credenciais como parâmetro.
<pre>$purchase = new BBM\Purchase('YOUR_API_KEY', 'YOUR_API_SECRET');</pre>

Escolha o ambiente
<pre>
$catalog->environment = 'production';
ou
$catalog->environment = 'sandbox';
</pre>

Envie para nós, os dados do Usuário que efetuou a compra, respeitando as regras abaixo.
<pre>
$customer = [
    'customerIdentificationNumber' => 1, // INT, YOUR STORE CUSTOMER ID
    'customerFullname' => 'CUSTOMER NAME', // STRING, CUSTOMER FULL NAME
    'customerEmail' => 'customer@email.com', // STRING, CUSTOMER EMAIL
    'customerGender' => 'm', // ENUM, CUSTOMER GENDER, USE m OR f (LOWERCASE!! male or female)
    'customerBirthday' => '1991/11/03', // STRING, CUSTOMER BIRTH DATE, USE Y/m/d (XXXX/XX/XX)
    'customerCountry' => 'BR', // STRING, 2 CHAR STRING THAT INDICATE THE CUSTOMER COUNTRY (BR, US, ES, etc)
    'customerZipcode' => '31231223', // STRING, POSTAL CODE, ONLY NUMBERS
    'customerState' => 'RJ' // STRING, 2 CHAR STRING THAT INDICATE THE CUSTOMER STATE (RJ, SP, NY, etc)
];

$purchase->setCustomer($customer);
</pre>

Em seguida adicione o ebook passando o ID e preço do mesmo.
<pre>$purchase->addItem($ebookID, $ebookPrice);</pre>

Obs. Você pode adicionar quantos ebooks forem necessários, bastando apenas repetir o procedimento para cada ebook.

Em seguida faça a validação do(s) ebook(s) e posteriormente o checkout.

Fluxo:

- O Usuário compra um ou mais de nossos produtos.
- Você valida a compra através da função validate();
- Estando tudo ok você pode prosseguir para realizar tanto o seu quanto o nosso checkout.

Obs.

Não execute a venda antes de validar conosco, pois existem condições que podem inviabilizar a mesma, tais como sua loja não estar disponível para venda, problemas com o ebook etc.

Não se esqueça de realizar o checkout conosco e você só deve fazê-lo quando o pagamento for efetivado pelo seu cliente.

<pre>
try
{
    $purchase->validate();
    
    //A transaction key pode ser qualquer coisa que desejar, mas recomendamos que seja a mesma de sua transação. Ela será requisitada quando for efetuar o download do(s) ebook(s) atrelados a este checkout.
    $purchase->checkout('TRANSACTION_KEY', time());
}
catch(\BBM\Server\Exception $e)
{
    var_dump($e);
}
</pre>

Pronto. Se tudo ocorreu bem você acaba de registrar uma venda conosco.

# Passo 5 - Fazendo download do ebook
Uma vez que seu cliente comprou um de nossos ebooks, você validou a compra e realizou o checkout, estará apto a fazer o download do mesmo. Caberá a você decidir a maneira de disponibilizar um link(ou algo parecido) para que seu cliente possa efetuar o download.  Conosco, tudo o que precisará fazer para efetuar o download é informar o id da transação(O mesmo que utilizou no checkout) e o id do ebook.

Instancie a classe download passando suas credenciais como parâmetro
<pre>$download = new BBM\Download('YOUR_APY_KEY', 'YOUR_API_SECRET');</pre>

Escolha o ambiente
<pre>
$catalog->environment = 'production';
ou
$catalog->environment = 'sandbox';
</pre>

<pre>
$data = [
    'ebook_id' => $EBOOKID,
    'transaction_time' => time(),
    'transaction_key' => $TIMESTAMP // A chave que voce utilizou para realizar o checkout
];
</pre>

<pre>
try
{
    $download->validate($data);
    $download->download();// Faz o download do ebook
}
catch(\BBM\Server\Exception $e)
{
    var_dump($e);
}
</pre>

Se tudo ocorreu bem, ao chamar a função download(), automaticamente o arquivo do ebook será baixado para a máquina do cliente, pois trata-se de um EndPoint.

# Tratando erros
Erros podem acontecer em todas as etapas(Complete, Update, Validate, Checkout e Download) e será de sua responsabilidade tratá-los e informar ao Usuário, se for o caso. Independente da requisição que esteja sendo feita, sempre retornaremos uma Exception com o código e mensagem do erro. Você pode verificar <a href="https://github.com/xxMAGRAOxx/magraoDocs/blob/master/errors.md" target="blank">aqui</a> uma lista dos possíveis erros que podem acontecer e suas respectivas etapas. Disponibilizamos também uma <a href="https://github.com/bibliomundi/client-side-api/tree/master/docs/" target="blank">documentação</a> gerada pelo <a target="blank" href="http://www.phpdoc.org/" target="blank">PHPDoc.</a>
