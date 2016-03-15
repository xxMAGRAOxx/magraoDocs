# Visão geral
Para que você possa vender nossos ebooks em sua loja serão necessários alguns passos e o auxílio de um programador.

Obs. Esta api requer o PHP como linguagem de programação e caso você trabalhe com outras linguagens disponibilizamos um webservice completo que você pode verificar <a href="http://www.google.com.br">aqui</a>.


# Requerimentos
<a href="http://php.net/manual/en/book.curl.php" target="blank">cURL</a>

<a href="http://php.net/" target="blank">PHP 5.5</a>

Conhecer o padrão <a href="http://www.editeur.org/83/Overview/" target="blank">Onix</a>

Chave de identificação e senha. Se ainda não possui solicite-as através do e-mail contato@bibliomundi.com.br. Reponderemos em instantes.

# Fluxo
1. Importar o nosso catálogo completo. Retornaremos para você um XML no padrão Onix com todos os ebooks cadastrados em nossa plataforma.

2. Inserir os ebooks em sua base de dados seguindo as normas do padrão Onix.

3. Realizar atualizações diárias para checar se existem novos ebooks, se existem ebooks a serem atualizados(Mudança de nome, por exemplo) ou se existem ebooks que requerem que você delete de sua base(Por não estarem mais sendo distribuídos por nós, por exemplo). Retornaremos para você um XML no padrão Onix com os ebooks que precisam ser atualizados, inseridos ou deletados.

4. Disponibilizar os ebooks para venda em sua loja.

5. Liberar o download do ebook para o cliente. 
 
# Instalação

Faça o download da nossa api e inclua em seu projeto.
Basta Fazer uma chamada ao arquivo “autoload.php”, que se encontra dentro da pasta "lib" e já terá acesso a todas as funcionalidades.
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
    $xml = $catalog->get();//Retorna um XML dos nossos ebooks no formato string e no padrão Onix
}
catch(\BBM\Server\Exception $e)
{
    throw $e;
}
</pre>

Cada tag &lt;Produto&gt; retornada pelo xml é um ebook. Você irá percorrer todas elas e, seguindo normas do padrão Onix, inserindo em sua base de dados.

# Passo 2 - Inserindo os ebooks em sua loja
Uma vez com o xml dos nossos ebooks, você pode trabalhar da maneira que achar melhor, mas recomendamos fortemente que utilize um parser, como o SimpleXML do php, por exemplo. Será de sua responsabilidade inserir os ebooks com as informações mínimas necessárias em sua loja. Recomendamos também que não insira ebooks que não estão disponíveis para venda, no momento da importação, e para isso você deverá realizar uma checagem através das tags PublishingStatus e ProductAvailability. Clicando <a target="blank" href="https://github.com/xxMAGRAOxx/magraoDocs/blob/master/onix_example.xml">aqui</a> você pode ver um exemplo de um xml Onix e de quais informações consideramos essenciais.

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
    throw $e;
}
</pre>

Para cada tag &lt;produto&gt; existirá uma tag chamada &lt;NotificationType&gt; indicando a operação a ser realizada.

Ex: 03 -> inserir. 04 -> Atualizar. 05 -> Deletar. Para a lista completa de códigos, vá ao item "P.1.2" da documentação Onix.

# Passo 4 - Realizando uma venda
Uma vez que você disponibilizar os ebooks em sua loja, seus clientes estarão aptos a relizar compras. Toda vez que um cliente tentar comprar um produto nosso será necessário que você valide a transação conosco e em caso de aprovação prosseguir para o checkout. Repare que a sua validação e o seu checkout e a nossa validação e o nosso checkout sãos duas coisas distintas. Você deverá sempre validar e fazer o checkout conosco para que tenhamos ciência de que a venda foi efetuada, para então podermos liberar o download para o seu cliente. Tenha isso em mente.

Instancie a classe Purchase passando suas credenciais como parâmetro.
<pre>$purchase = new BBM\Purchase('YOUR_API_KEY', 'YOUR_API_SECRET');</pre>

<pre>
$catalog->environment = 'production';
ou
$catalog->environment = 'sandbox';
</pre>

Envie os dados do Usuário, que efetuou a compra, respeitando as regras abaixo.
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
</pre>

e então adicione o Usuário
<pre>$purchase->setCustomer($customer);</pre>

Em seguida adicione o ebook passando o ID e preço do mesmo.
<pre>$purchase->addItem(3, 9.99);</pre>

Obs. Você pode adicionar quantos ebooks forem necessários, bastando apenas repetir o procedimento anterior para cada ebook.

Em seguida faça a validação do(s) ebook(s) e checkout.

# Tratando erros
