# Visão geral
Para que você possa vender nossos ebooks em sua loja serão necessários alguns passos e o auxílio de um programador.
<br />
Obs. Esta api requer o PHP como linguagem de programação e caso você trabalhe com outras linguagens disponibilizamos um webservice completo que você pode verificar <a href="http://www.google.com.br">aqui</a>.


# Requerimentos
Curl 
<br />
PHP 5.5  ou maior

# Fluxo
1. Antes de começarmos você irá precisar de uma chave de identificação e uma senha. Caso ainda não possua entre em contato conosco através do e-mail contato@bibliomundi.com.br
<br />
2. Importar o nosso catálogo completo. Retornaremos para você um XML no padrão Onix com todos os ebooks cadastrados em nossa plataforma.
<br />
3. Inserir os ebooks em sua base de dados e posteriormente disponibilizá-los para venda em sua loja.
<br />
4. Realizar atualizações diárias para checar se existem novos ebooks, se existem ebooks a serem atualizados(Mudança de nome, por exemplo) ou se existem ebooks que requerem que você delete de sua base(Por não estarem mais sendo distribuídos por nós, por exemplo). Retornaremos para você um XML no padrão Onix com os ebooks que precisam ser atualizados, inseridos ou deletados.

# Instalação

Faça o download da nossa api e inclua em seu projeto. <br />
Faça a chamada ao arquivo “autoload.php”, que se encontra dentro da pasta "lib" e já terá acesso a todas as classes.
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

# Passo 2 - Inserindo os ebooks em sua loja
Uma vez com o xml dos nossos ebooks, você pode trabalhar da maneira que achar melhor, mas recomendamos fortemente que utilize um parser, como o SimpleXML do php, por exemplo. Será de sua responsabilidade inserir os ebooks com as informações mínimas necessárias em sua loja. Recomendamos também que não insira ebooks que não estão disponíveis para venda, no momento da importação, e para isso você deverá realizar uma checagem através das tags PublishingStatus e ProductAvailability. Clicando <a target="blank" href="https://github.com/xxMAGRAOxx/magraoDocs/blob/master/onix_example.xml">aqui</a> você pode ver um exemplo de um xml Onix e de quais informações consideramos essenciais.

# Passo 3 - Realizando atualizações diárias
Realizamos atualizações diárias em nosso sistema e você precisará, também diariamente, criar uma rotina para checar se existem ebooks a serem inseridos, atualizados ou deletados.
Recomendamos que crie uma agendador de tarefas para rodar entre 01 e 06 da manhã(UTC-3) afim de evitar que ebooks sejam disponibilizados com dados defasados podendo causar erros na venda.<br />
Você irá fazer praticamente o mesmo do passo anterior precisando passar apenas um parâmetro no contrutor da classe Catalog.

Passe apenas um terceiro parâmetro chamado 'updates'.

<pre>$catalog = new BBM\Catalog('YOUR_API_KEY', 'YOUR_API_SECRET', 'updates');</pre>

<pre>
$catalog->environment = 'production';
ou
$catalog->environment = 'sandbox'; 

try
{
    $catalog->validate();//Valida suas credenciais
    $xml = $catalog->get();//Retorna um XML com os ebooks e as operações a serem relizadas(insert, update ou delete) no formato string e no padrão Onix
}
catch(\BBM\Server\Exception $e)
{
    throw $e;
}
</pre>

# Passo 4 - Realizando uma venda

# Tratando erros
